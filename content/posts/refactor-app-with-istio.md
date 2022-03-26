---
title: "Refactoring Apps Safely with Istio"
tags:
  - kubernetes
  - go
  - istio
publishdate: "2022-03-25"
---

This blog post will walk you through rewriting a portion of a service safely,
100% in production, by leveraging Istio's routing features.  It features a
basic service with the following endpoints:

* `/healhtz`: a healthcheck endpoint that always responds with a `200` status
  code
* `/`: the base endpoint, which generates a random number a random number of
  times

Later, we'll discover that the random number generation function in our
original service is suffering from horrible performance.  To solve this, we'll
start by refactoring that function into its own endpoint:

* `/random/number`: an endpoint that returns a random number between `0` and
  `9998`

Once the function has been refactored into its own endpoint and our internal
`/` endpoint has been updated to call the new endpoint (instead of the function
directly), we'll rewrite the slow, problematic code in another language, deploy
it, and shift traffic slowly, observing the impact on performance as we go.

> The codebase is trivially slow.  An intentional delay exists in the original
> Python codebase and does not exist in the rewritten Golang implementation.
> This post is not a comparison of languages, and there is an assumption that
> the "faulty" performance is not a fault of Python (and is therefore not
> necessarily solved by rewriting the logic in Go), but instead an issue so
> embedded in an existing codebase (such as the libraries it uses) that
> rewriting portions of the codebase from scratch would be easier, safer, and
> just plain "better" than trying to fix the original code.

## Prerequisites

Before we can really get going, we need to know what technologies we're using.
We already spoiled the biggest part of the stack: [Istio][istio].  But we also
need somewhere to run our application.  Since we're using Istio, we'll also be
using Kubernetes.  To run Kubernetes, we'll use [minikube][minikube].  To
manage our service deployment and configuration, we'll use [helm][helm].
Finally, we'll use [skaffold][skaffold] for a slightly faster feedback loop.
After all, we want continuous development!

### Podman/Docker

Before anything else, we want to be able to build containers.  Docker is
falling out of favor in some circles (such as Kubernetes,
[where it has been deprecated as a container runtime][docker-deprecation]), so
I've opted to use [podman][podman] for my local container needs.  If you're
running Fedora (I am), you can install `podman`:

```bash
$ sudo dnf update -y
$ sudo dnf install -y podman
```

For other operating systems, see the [official instructions][podman-install].
For Docker, see [their installation instructions][docker-install].

> If you're new to `podman`, 90% of the commands appear to be the same as
> `docker`.  For example, `docker build -t example .` and
> `podman build -t example .`.

### Minikube

Before we can do anything with Kubernetes, we need to
[install kubectl][kubectl-install] and [install minikube][minikube-install].
`minikube` lets us run Kubernetes on our laptops, while `kubectl` gives us a
way to interact with Kubernetes.

If you're on Linux, the following should work:

```bash
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Or on macOS:

```bash
$ brew install kubectl
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
$ sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

> If you run into problems, the [official minikube instructions][minikube-install]
> or [official kubectl docs][kubectl-install] would be a good place to start.

Once installed, start a cluster up:

```bash
$ minikube start --cpus 4 --memory 8192
😄  minikube v1.25.2 on Fedora 35
✨  Using the podman driver based on user configuration
❗  Your cgroup does not allow setting memory.
    ▪ More information: https://docs.docker.com/engine/install/linux-postinstall/#your-kernel-does-not-support-cgroup-swap-limit-capabilities
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
E0323 19:31:25.359746 2176044 cache.go:203] Error downloading kic artifacts:  not yet implemented, see issue #8426
🔥  Creating podman container (CPUs=4, Memory=8192MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

> You may not need 4 CPUs and 8G RAM to follow along.  My machine had the spare
> capacity, so I tried to get as close to the Istio-recommended specs of 4 CPUs
> and 16G RAM as I could.

Next, enable the `metallb` addon to make it easier to access cluster services;
and enable the `registry` addon to easily publish images:

```bash
$ minikube addons enable metallb
    ▪ Using image metallb/speaker:v0.9.6
    ▪ Using image metallb/controller:v0.9.6
🌟  The 'metallb' addon is enabled
$ minikube addons enable registry
    ▪ Using image registry:2.7.1
    ▪ Using image gcr.io/google_containers/kube-registry-proxy:0.4
🔎  Verifying registry addon...
🌟  The 'registry' addon is enabled
```

Before proceeding, update the `metallb` configuration with a range of IPs to
assign.  For this, find out the IP of your `minikube` node:

```bash
$ kubectl get nodes -o wide
NAME       STATUS   ROLES                  AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                  CONTAINER-RUNTIME
minikube   Ready    control-plane,master   8m28s   v1.23.3   192.168.49.2   <none>        Ubuntu 20.04.2 LTS   5.16.13-1.surface.fc35.x86_64   docker://20.10.12
```

In my example, the `INTERNAL-IP` is `192.168.49.2`, so I'll pick a range of `192.168.49.50-192.168.49.100`,
a subset of IPs within the `192.168.49.0/24` CIDR.  Update the `configmap` for `metallb`:

```bash
$ kubectl edit configmap config -n metallb-system
```

Find the list of `addresses` and change it to have a single element with your
range.  For my example, I modified the following:

```yaml
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - -
```

To look like this:

```yaml
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.49.50-192.168.49.100
```

Or, as just a simple diff:

```bash
$ $ diff old.yml new.yml 
8c8
<       - -
---
>       - 192.168.49.50-192.168.49.100
```

> While the above addons aren't necessary, they can make troubleshooting easier
> if you run into issues.

And finally, make sure you can list pods:

```bash
$ kubectl get pods -n kube-system
NAME                               READY   STATUS    RESTARTS        AGE
coredns-64897985d-rbsxk            1/1     Running   0               5m21s
etcd-minikube                      1/1     Running   0               5m33s
kube-apiserver-minikube            1/1     Running   0               5m35s
kube-controller-manager-minikube   1/1     Running   0               5m32s
kube-proxy-kpxl2                   1/1     Running   0               5m21s
kube-scheduler-minikube            1/1     Running   0               5m32s
registry-nk4wd                     1/1     Running   0               51s
registry-proxy-hc274               1/1     Running   0               51s
storage-provisioner                1/1     Running   1 (4m49s ago)   5m31s
```

### Helm

Helm is a package manager for Kubernetes.  In much the same way as you might
`brew install <pkg>` or `apt-get install <pkg>`, you can
`helm install <chart>`.

Installing helm is as easy as running the Helm install script:

```bash
$ curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

> `curl [...] | bash` isn't exactly great form.  Visit
> [the official instructions][helm-install] for a better approach.

### Istio

Istio is the key to this entire blog post and enables features such as:

* mTLS
* Tracing
* Performance metrics
* Load balancing
* A/B traffic routing
* Circuit breaking

Installing Istio as as simple as running a script and an `istioctl` command:

```bash
$ curl -L https://istio.io/downloadIstio | sh -
$ cd istio-*
$ sudo install bin/istioctl /usr/local/bin/istioctl
$ istioctl install
This will install the Istio 1.13.2 default profile with ["Istio core" "Istiod" "Ingress gateways"] components into the cluster. Proceed? (y/N) y
✔ Istio core installed                                                                                                                                                                                  
✔ Istiod installed                                                                                                                                                                                      
✔ Ingress gateways installed                                                                                                                                                                            
✔ Installation complete                                                                                                                                                                                 Making this installation the default for injection and validation.

Thank you for installing Istio 1.13.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/pzWZpAvMVBecaQ9h9
```

Before going further, let's find the `LoadBalancer` IP for the Istio ingress
gateway pod and create an entry in `/etc/hosts` to map the IP to the name of
the service we're refactoring.

```bash
$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   10.109.42.27   192.168.49.50   15021:31486/TCP,80:32626/TCP,443:30232/TCP   19m
$ echo "192.168.49.50 rewrite-app-with-istio.test" | sudo tee -a /etc/hosts
192.168.49.50 rewrite-app-with-istio.test
```

> An Istio gateway pod is similar to an Ingress controller pod.  It makes it
> possible to get traffic into the service mesh from the outside world.

For more details, see the [official guide][istio-install].

### Skaffold

Skaffold lets us define how our application is deployed and watches for changes
on disk, automatically rebuilding and/or redeploying when changes are detected.

Installing Skaffold on Linux:

```bash
$ curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64
$ sudo install skaffold /usr/local/bin/
```

And on macOS:

```bash
$ curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-darwin-amd64
$ sudo install skaffold /usr/local/bin/
```

As usual, consult the [official docs][skaffold-install] for more details.

## Application Baseline

Our entire goal is to refactor an application with poor performance, so let's
start with deploying our application as it exists today!

You should be able to follow the commands below to clone the repository, switch
to the appropriate version (reflected as branches named `phase-$number`),
deploy to your `minikube` cluster using `skaffold`, and verify the service
works.

```bash
$ git clone https://github.com/supertylerc/istio-rewrite-app
$ git checkout phase-1
$ skaffold dev
<SNIP>

# In another terminal window
$ curl rewrite-app-with-istio.test/
{"location":"/","num_requests":44,"responses":[{"number":6245},{"number":3801},{"number":2548},{"number":6676},{"number":6959},{"number":8770},{"number":8827},{"number":6233},{"number":3857},{"number":6438},{"number":7188},{"number":2737},{"number":3254},{"number":1474},{"number":959},{"number":4878},{"number":1151},{"number":8851},{"number":5925},{"number":4714},{"number":5203},{"number":2429},{"number":5149},{"number":9408},{"number":7805},{"number":6783},{"number":4636},{"number":3549},{"number":3188},{"number":9600},{"number":4609},{"number":342},{"number":9104},{"number":8082},{"number":3424},{"number":8156},{"number":3603},{"number":2884},{"number":8624},{"number":1833},{"number":5310},{"number":1898},{"number":7776},{"number":3541}],"time":20.019914}
```

You can rerun this `curl` multiple times, taking note of the `num_requests` and
`time` fields.  Let's take a (brief) look at our code at this stage:

```python
@app.route('/')
def root():
    start = datetime.now()

    num_requests = random.randint(0, 50)
    responses = []
    for _ in range(0, num_requests):
        responses.append(random_number())

    finish = datetime.now()
    duration = finish - start 

    return {
        "location": "/",
        "num_requests": num_requests,
        "responses": responses,
        "time": float(duration.total_seconds()),
    }


def random_number():
    # Simulate some problem in the app logic by sleeping for random time
    # This will later become a new API endpoint that we break out
    time.sleep(random.randint(0.0, 1.0))
    return {
        "number": random.randint(0, 9999)
    }
```

As you can see, we're introducing a `time.sleep()` function to sleep between
`0` and `1.0` second, or between `0` and `1000` miliseconds.  This simulates
poor performance in our code, and it's the entire reason we've decided to
rewrite our application.  The first thing we'll do is to make this
`random_number()` function into an endpoint within our existing Python
application, and we'll follow that up by writing an implementation of _that_
endpoint in Go.  Along the way, we'll leverage some basic Istio features to
make this refactoring safer for our end users.

### Install Observability Stack

So before we start refactoring, we should generate some requests so that we
can have a baseline performance.  For this, we'll deploy Prometheus, Grafana,
Kiali, and Jaeger.  This will make up our "observability stack," giving us
some insight into the performance and reliability of our application.  One of
the things to note through this process is that we're not making any changes to
our application code to get this information, which means no matter how
distributed or diverse your application stack (and its many languages), you
can get this with no code changes.

* Install Prometheus, a time-series database and collection tool

```bash
$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/prometheus.yaml
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
```

* Install Grafana, a data visualization and dashboarding tool

```bash
$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/grafana.yaml
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
```

* Install Jaeger, a distributed tracing tool

```bash
$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/jaeger.yaml
deployment.apps/jaeger created
service/tracing created
service/zipkin created
service/jaeger-collector created
```

* Install Kiali, a service mesh observability dashboard

```bash
$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/kiali.yaml
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created
```

### Generate Requests

For this, we'll use `siege` to run multiple `GET /` requests concurrently.
We'll use our observability stack to measure the performance of our app based
on 10 concurrent requests.

On Fedora, you can install `siege` with a simple `sudo dnf install -y siege`.
On Ubuntu, this is `sudo apt install -y siege`, and on macOS you can use `brew`
to install it with `brew install siege`.

Now, start your siege!

```bash
$ siege --concurrent=10 --benchmark --time=5M http://rewrite-app-with-istio.test/
HTTP/1.1 200    14.03 secs:     623 bytes ==> GET  /
HTTP/1.1 200    13.02 secs:     338 bytes ==> GET  /

<SNIP>

HTTP/1.1 200     2.02 secs:     160 bytes ==> GET  /
HTTP/1.1 200    13.02 secs:     638 bytes ==> GET  /
HTTP/1.1 200    16.03 secs:     544 bytes ==> GET  /

Lifting the server siege...
Transactions:		         245 hits
Availability:		      100.00 %
Elapsed time:		      299.78 secs
Data transferred:	        0.10 MB
Response time:		       11.84 secs
Transaction rate:	        0.82 trans/sec
Throughput:		        0.00 MB/sec
Concurrency:		        9.68
Successful transactions:         245
Failed transactions:	           0
Longest transaction:	       32.04
Shortest transaction:	        0.00
```

While this is running, use the `istioctl dashboard grafana` and
`istioctl dashboard kiali` commands to explore the metrics that Istio is
providing "for free."

![Grafana Dashboard Baseline](/img/phase1-baseline-grafana.png)
![Kiali Dashboard Baseline](/img/phase1-baseline-kiali.png)

## Refactor Logic to its own Endpoint

As you can see, we get metrics that confirm our app does not have great
performance.  Now, after some debugging, we've determined that the problem is
our random number generation logic.  We've decided that the best way to handle
this is to rewrite that particular logic in Go.  Before that, though, we decide
to refactor the logic _just slightly_ so that the random number generation is
contained within its own endpoint.  To simulate that, you can checkout the
`phase-2` branch:

```bash
$ git checkout phase-2
branch 'phase-2' set up to track 'origin/phase-2'.
Switched to a new branch 'phase-2'
```

Our changes are pretty simple: we add the `requests` library to make calls to
the `/random/number` endpoint, and we add the `/random/number` route using
Flask.

```bash
$ diff --git a/app.py b/app.py
index 01eb241..717dda7 100644
--- a/app.py
+++ b/app.py
@@ -18,7 +18,7 @@ def root():
     num_requests = random.randint(0, 50)
     responses = []
     for _ in range(0, num_requests):
-        responses.append(random_number())
+        responses.append(requests.get(f"{EXAMPLE_SERVICE}/random/number").json())
 
     finish = datetime.now()
     duration = finish - start 
@@ -31,11 +31,13 @@ def root():
     }
 
 
+@app.route("/random/number")
 def random_number():
     # Simulate some problem in the app logic by sleeping for random time
-    # This will later become a new API endpoint that we break out
+    # This is a new API endpoint that we will rewrite in Golang later
     time.sleep(random.randint(0.0, 1.0))
     return {
+        "location": "/random/number",
         "number": random.randint(0, 9999)
     }
```

Great!  If your `skaffold` is still running, this should update automatically.
If it isn't, though, you can just `skaffold dev` to re-deploy.  We shouldn't
see any difference between the previous version and the new one -- we just
added an endpoint with existing business logic and started making additional
HTTP requests to get the data.  Re-run your `siege` tests and review your
baseline metrics for this phase.

When you look at your metrics, you might notice that the 50th and 90th
percentile response times appear to be drastically improved simply by
refactoring our endpoint.  Unfortunately, this isn't really reflecting
our end user's experience.  Those metrics are now also reflecting the requests
we're making to our new endpoint!  Since we call these _every_ time we want a
number (instead of the endpoint accepting a number of numbers to generate), we
get deceptively "better" performance in the 90th percentile!  This can be
confusing, but the real key here is our 99th percentile.  Something is still
very wrong.

![Grafana Dashboard Baseline](/img/phase2-baseline-grafana.png)
![Kiali Dashboard Baseline](/img/phase2-baseline-kiali.png)

## Rewrite the Random Number Generator in Go

Now, we'll rewrite our Random Number Generator in Go.  The main `/` URI will
still be handled by our original Python code, but we'll try to improve our
performance by switching to Go for _just_ the problematic endpoint.  To do
that safely, we'll leave the `/random/number` endpoint in the Python codebase
and slowly migrate traffic using Istio's traffic routing features.  This will
let us see if there are any major, unseen errors before we really migrate the
endpoint.

To start, `git checkout phase-3`.  The diff between `phase-2` and `phase-3`
is significant.  You should definitely check it out, but the short version
is:

* We wrote a new app in Go to handle `/random/number`
* We refactored the Helm chart _heavily_ to support multiple deployments
* We added new functionality to the Helm chart to support Istio's traffic
  routing functionality

We'll talk mostly about the Istio-specific changes from here on.  To start,
though, let's run our application as-is.  It's currently configured to continue
routing all traffic to the original Python application.  Let's use `siege` to
generate some traffic and ensure that's still the case using our observability
stack.  First, though, let's verify that we now have _two_ deployments of our
app:

```bash
$ kubectl get deploy
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
rewrite-app-with-istio-blue    1/1     1            1           11s
rewrite-app-with-istio-green   1/1     1            1           11s
$ kubectl get deploy -o jsonpath='{range .items[*]}{.spec.template.spec.containers[*].image}{"\n"}'
rewrite-app-with-istio:0d3d71f094fb4e2c556bdaef5f6ebf10ffc8666ef5f69a1e8ca92c9dd822dfbf
rewrite-app-with-istio-v2:59bda0b318308776108b01d841a62c99ea4d95cc65cb1e791d821c48e99cd1e3
```

We can see above that one of our images has `-v2` in the name -- that's our new
Go implementation.  Next, use `siege` and your observability tools to ensure no
traffic is going to the Go implementation currently.

![Kiali Dashboard Baseline](/img/phase3-baseline-kiali.png)

This looks pretty much the same.  Ok, now let's starting shifting some traffic!

### Shift 1% of Traffic to Go Service

We'll start with just 1% of traffic to our new Go service.  To do this, edit the
`helm/values.yaml` file and find the `weight` keys.  Modify them so that the
`weight` of the `green` subset is `1` and the `weight` of the `blue` subset is
`99`:

```bash
$ git diff
diff --git a/helm/values.yaml b/helm/values.yaml
index f926f96..ee5dd64 100644
--- a/helm/values.yaml
+++ b/helm/values.yaml
@@ -110,11 +110,11 @@ istioVirtualServices:
           - destination:
               host: rewrite-app-with-istio
               subset: green
-            weight: 0
+            weight: 1
           - destination:
               host: rewrite-app-with-istio
               subset: blue
-            weight: 100
+            weight: 99
       - name: "rewrite-app-with-istio-route-v1"
         route:
           - destination:
```

If you're using `skaffold dev`, it should pick this change up and automatically
deploy it.  Now, use `siege` again to generate traffic.  This time, keep an eye
on your observability data!

![Kiali Dashboard Baseline](/img/phase3-errors-kiali.png)
![Grafana Dashboard Baseline](/img/phase3-errors-grafana.png)

Oh no!  We see an increased error rate!  But we only shifted 1% of traffic, so
why is our error rate so high?

First, shifting traffic isn't an exact science.  But more importantly, we
shifted traffic for an endpoint called from `/`, and `/` is calling that one
endpoint multiple times.  So, if `/` calls `/random/number` 30 times, and _any_
of those gets routed to our new service, it will result in `/` having an error.
So our 1% shift for `/random/number` has a >1% effect on `/` as a result.

#### Detour: The Magic Behind Shifting Traffic

The "magic" of this process is Istio's combination of `VirtualService` and
`DestinationRule`.  An Istio `DestinationRule` lets us classify our service
into multiple parts, or `subset`s.  A `VirtualService` lets us describe how to
route traffic for our service.  Note that this leverages the normal Kubernetes
`Service` for service discovery, but it doesn't actually _use_ the Kubernetes
`Service` for making decisions or delivering traffic.  To get an idea for how
these work, let's start with our `DestinationRule` manifest:

```yaml
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: rewrite-app-with-istio
spec:
  host: rewrite-app-with-istio
  subsets:
  - name: blue
    labels:
      color: blue
  - name: green
    labels:
      color: green
```

This manifest tells us that, for the "host" `rewrite-app-with-istio`, we have
some "subsets".  So, what's a `host`, and what are `subsets`?

The `host` is, effectively, the Kubernetes service name to which these
`subsets` will apply.  The `subsets` here are effectively label selectors for
endpoints, or pods, within that service.  One way we can see what these
`subsets` are is by using `kubectl` to get the label selectors that the service
uses, then find pods that match them and get _their_ labels:

```bash
$ kubectl get svc/rewrite-app-with-istio -o=jsonpath='{.spec.selector}{"\n\n"}'
{"app.kubernetes.io/instance":"rewrite-app-with-istio","app.kubernetes.io/name":"rewrite-app-with-istio"}

$ kubectl get pod \
    -l app.kubernetes.io/instance=rewrite-app-with-istio,app.kubernetes.io/name=rewrite-app-with-istio \
    -o jsonpath='{"\n"}{range .items[*]}{.metadata.name}: {" labels: "}{.metadata.labels}{"\n\n"}{end}'

rewrite-app-with-istio-blue-8b5cdc96f-64wfm:  labels: {"app.kubernetes.io/instance":"rewrite-app-with-istio","app.kubernetes.io/name":"rewrite-app-with-istio","color":"blue","pod-template-hash":"8b5cdc96f","security.istio.io/tlsMode":"istio","service.istio.io/canonical-name":"rewrite-app-with-istio","service.istio.io/canonical-revision":"latest","sidecar.istio.io/inject":"true"}

rewrite-app-with-istio-green-7456c447c4-h4ct6:  labels: {"app.kubernetes.io/instance":"rewrite-app-with-istio","app.kubernetes.io/name":"rewrite-app-with-istio","color":"green","pod-template-hash":"7456c447c4","security.istio.io/tlsMode":"istio","service.istio.io/canonical-name":"rewrite-app-with-istio","service.istio.io/canonical-revision":"latest","sidecar.istio.io/inject":"true"}
```

If we look carefully, we can see that our pods have a `color` label, and that
for one pod it's `blue`, and for the other, it's `green`.  This is how our
`subsets` are found: first, lookup the `service` that matches the `host`; then,
find pods that the service matches that _also_ have `labels` that match our
`DesinationRule`'s `subsets` labels.

> It isn't _quite_ based on the `service`, but it's close enough for
> illustration and example purposes.

So, if we want to know "which `subset` is the one with a `name` of `green`, we
can use the service's `.spec.selector` plus the `subset`'s `labels`:

```bash
$ kubectl get pod \
    -l app.kubernetes.io/instance=rewrite-app-with-istio,app.kubernetes.io/name=rewrite-app-with-istio,color=green
NAME                                            READY   STATUS    RESTARTS   AGE
rewrite-app-with-istio-green-7456c447c4-h4ct6   2/2     Running   0          25m
```

Okay!  Now that we know how subsets are identified, how do we use them?  For
that, we turn to the `VirtualService.`  For our example, we have two: one that
applies to our ingress `Gateway`; and one that applies to in-cluster traffic
(or traffic natively within the mesh).  We'll just look at the one for our
in-cluster traffic, but the same principles apply to the one for our `Gateway`
(albeit with the addition of a `.spec.gateways[]` to identify the `Gateway`s to
which the VirtualService should be applied).

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  labels:
    app.kubernetes.io/instance: rewrite-app-with-istio
    app.kubernetes.io/name: rewrite-app-with-istio
    app.kubernetes.io/version: 0.0.1
  name: cluster
spec:
  hosts:
  - rewrite-app-with-istio
  http:
  - match:
    - uri:
        prefix: /random/number
    name: rewrite-app-with-istio-route-v2
    route:
    - destination:
        host: rewrite-app-with-istio
        subset: green
      weight: 1
    - destination:
        host: rewrite-app-with-istio
        subset: blue
      weight: 99
  - name: rewrite-app-with-istio-route-v1
    route:
    - destination:
        host: rewrite-app-with-istio
        subset: blue
      weight: 100
```

This shows us that we're applying our rules to anything that's talking to the
`rewrite-app-with-istio` host.  We're specifically creating `http` rules since
our application is an HTTP-based application.  These rules are evaluated in
order.  Our first rule matches a URI `prefix` of `/random/number`, which is the
endpoint we're refactoring.  It gives it a `name`, and then it has multiple
possible ways to `route` the traffic: a `weight` of `1` (or 1%) will go to the
`green` subset; a `weight` of `99` (or 99%) will go to the `blue` subset.
Recall that our original Python implementation is in the `blue` `subset`
(because the `subset` named `blue` in a `DestinationRule` for the same `host`
looks for pods within the `host` service [`rewrite-app-with-istio`] with the
label of `color: blue`, and the only pods matching that label are our Python
application), while the Go implementation is in the `green` `subset`.

Our second `http` rule doesn't have a `match`, so it applies to everything.
It's effectively a "catch all."  But because these rules are evaluated
sequentially, it's only matching on requests that have a URI that _isn't_
`/random/number`.  Being aware of this sequential nature is crucial: if the
"catch all" rule existed _before_ the `/random/number` rule, then the rule for
`/random/number` would never be evaluated.

There's a _lot_ more power to be had in the combinations of `Gateway`,
`DestinationRule`, and `VirtualService` and the many parameters that they each
offer.  Distributing traffic in this way is only the simplest of choices.

### Fix the Go Bug

To fix our Go bug, we'll simply remove the artificial HTTP 500 response we were
setting and fix the usage of `rand.Intn()`:

```bash
diff --git a/v2/main.go b/v2/main.go
index d386dd3..4de51a2 100644
--- a/v2/main.go
+++ b/v2/main.go
@@ -30,10 +30,10 @@ func health(w http.ResponseWriter, req *http.Request) {
 }
 
 func randomNumber(w http.ResponseWriter, req *http.Request) {
-  w.WriteHeader(http.StatusInternalServerError)
+  randomNumber := rand.Intn(9999)
   json.NewEncoder(w).Encode(&randomNumberResponse{
     Location: "/random/number",
-    Number: nil,
+    Number: randomNumber,
     EndpointVersion: "v2",
   })
 }
```

At this point, `skaffold` should pick your changes up and automatically deploy
the new version of your Go app.  So, let's give `siege` another try and observe
the results!

![Kiali Dashboard Baseline](/img/phase3-1percent-kiali.png)
![Grafana Dashboard Baseline](/img/phase3-1percent-grafana.png)

Okay!  Now we can clearly see _around_ 1% of traffic is going to our new app
we have no errors!  Let's go ahead and push those ratios to 50/50.

### Shift 50% of Traffic to Go Service

Again, edit `helm/values.yaml` and change those weights to `50` and `50`:

```bash
diff --git a/helm/values.yaml b/helm/values.yaml
index f926f96..9705033 100644
--- a/helm/values.yaml
+++ b/helm/values.yaml
@@ -110,11 +110,11 @@ istioVirtualServices:
           - destination:
               host: rewrite-app-with-istio
               subset: green
-            weight: 1
+            weight: 50
           - destination:
               host: rewrite-app-with-istio
               subset: blue
-            weight: 99
+            weight: 50
       - name: "rewrite-app-with-istio-route-v1"
         route:
           - destination:
```

Skaffold should pick this up and deploy it, so let's once again generate some
traffic with `siege` and observe.

![Kiali Dashboard Baseline](/img/phase3-5050-kiali.png)
![Grafana Dashboard Baseline](/img/phase3-5050-grafana.png)

From this, we can see that all of our requests are still returning successfully
and that around half of our requests are going to our new Go service!  We can
even see that performance is, in fact, better.  We have roughly _half_ the 99th
percentile response time we had before, and the number of requests we've been
able to handle has increased around 50%.  So, let's take our refactor the its
logical conclusion and shift 100% of traffic to the Go service.

### Shift 100% of Traffic to Go Service

As before, we're just changing weights in `helm/values.yaml` to shift traffic
around:

```bash
diff --git a/helm/values.yaml b/helm/values.yaml
index f926f96..b4a72f7 100644
--- a/helm/values.yaml
+++ b/helm/values.yaml
@@ -110,11 +110,11 @@ istioVirtualServices:
           - destination:
               host: rewrite-app-with-istio
               subset: green
-            weight: 0
+            weight: 100
           - destination:
               host: rewrite-app-with-istio
               subset: blue
-            weight: 100
+            weight: 0
       - name: "rewrite-app-with-istio-route-v1"
         route:
           - destination:
```

> Remember that right now, our `green` deployment is our Go service, while
> `blue` is the old Python service.

Let's use `siege` again to generate some artificial traffic and see what our
refactor has given us!  This time, though, I'm going to reduce the duration of
our siege to 1 minute -- this is because the performance increased so much that
I exhausted the ephemeral ports available on my machine (due to `TIME_WAIT`
state)!

```bash
$ siege --concurrent=10 --benchmark --time=1M http://rewrite-app-with-istio.test/
HTTP/1.1 200     0.87 secs:    2848 bytes ==> GET  /
HTTP/1.1 200     0.74 secs:    2436 bytes ==> GET  /

<SNIP>

HTTP/1.1 200     0.23 secs:     745 bytes ==> GET  /
HTTP/1.1 200     0.58 secs:    1896 bytes ==> GET  /
HTTP/1.1 200     0.94 secs:    2985 bytes ==> GET  /

Lifting the server siege...
Transactions:		        1126 hits
Availability:		      100.00 %
Elapsed time:		       59.30 secs
Data transferred:	        1.83 MB
Response time:		        0.52 secs
Transaction rate:	       18.99 trans/sec
Throughput:		        0.03 MB/sec
Concurrency:		        9.93
Successful transactions:        1126
Failed transactions:	           0
Longest transaction:	        1.67
Shortest transaction:	        0.01
```

![Kiali Dashboard Baseline](/img/phase3-100percent-kiali.png)
![Grafana Dashboard Baseline](/img/phase3-100percent-grafana.png)

From this, we can see that our refactored endpoint has _massively_ improved
our application's performance, even though we're sending everything through
our initial Python code's `/` endpoint and only sending the `/random/number`
endpoint to our new Go implementation.

## Conclusion and Final Thoughts

Hopefully this shows how you can leverage Istio to incrementally shift traffic
into a refactored application.  While this example rewrote a single endpoint in
an entirely new language, you could use the same techniques to refactor code in
the same language, to rewrite the _entire_ application, to introduce new
features, and more.  Istio is a powerful ally in the realm of operations: you
get observability and operational control over your application stack; you get
mutual TLS; you can inject failures (not covered); you can implement fault
tolerance via circuit breaking (not covered); and much, much more.  With the
ability to shift traffic in increments as small as 1%, you can deploy new code,
whether it be bug fixes, new features, refactoring, or even debugging builds
directly to production, knowing that you're in control.  Because let's face it,
even the best staging environments rarely _truly_ mirror production usage.

We didn't really talk much about tracing, but we _did_ install
[Jaeger][jaeger].  You can check it out by using the
`istioctl dashboard jaeger` command.  The default configuration "traces" around
1% of requests, but that can be changed.

This was a long, dense post.  It was a challenge to write.  Writing the code
was incredibly easy -- after all, they're just random number generators with
some artificial delay built-in.  But trying to decide if I should show errors,
how much detail I wanted to go into with exploring metrics (Grafana, Kiali, and
even Jaeger is installed if you want to look at some very limited tracing) --
that was difficult.  I also had to decide if I would simply refactor the
existing code or rewrite it in an entirely new language.  Should I include
details on the prerequisites, or should I just skip to the meat of the thing?
Ultimately, I wanted to create something that (hopefully) others could follow
if they started out from a brand new OS install.  I hope I found the right
balance in the end.

If you found this helpful, interesting, or just the literal worst of all things,
I'd love to hear your feedback [on Twitter][twitter].

[minikube-install]: https://minikube.sigs.k8s.io/docs/start/
[istio-install]: https://istio.io/latest/docs/setup/getting-started/
[helm-install]: https://helm.sh/docs/intro/install/
[skaffold-install]: https://skaffold.dev/docs/install/#standalone-binary
[podman-install]: https://podman.io/getting-started/installation
[docker-install]: https://docs.docker.com/get-docker/
[kubectl-install]: https://kubernetes.io/docs/tasks/tools/
[twitter]: https://twitter.com/supertylerc
[istio]: https://istio.io/
[minikube]: https://minikube.sigs.k8s.io/
[skaffold]: https://skaffold.dev/
[helm]: https://helm.sh/
[jaeger]: https://www.jaegertracing.io/
[podman]: https://podman.io
[docker-deprecation]: https://kubernetes.io/blog/2022/01/07/kubernetes-is-moving-on-from-dockershim/
