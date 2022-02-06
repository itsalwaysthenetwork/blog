---
title: "Managing Virtual Network Labs with KNE"
tags:
  - kubernetes
  - kne
  - srlinux
publishdate: "2022-02-05"
---

This post will be a (very short) introduction to [`kne`][kne], a project
from Google that lets you run virtual network topologies in Kubernetes.
I'm writing this as the current docs are a bit lacking for anyone who
really just wants a lab topology without necessarily understanding how
to do things with Kubernetes.

# Prerequisites

To get started, you'll need a few things:

* [Docker][docker]
* [Go][golang]
* [`kind`][kind]
* [`kubectl`][kubectl]

## Install Docker

This is the most complicated step.  The official intsallation guide from
Docker is [here][install-docker], but if you're on Linux, you can use
the install script:

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh ./get-docker.sh
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
$ newgrp docker
$ docker run hello-world
```

> The last command just runs a test image to verify that Docker was
> successfully installed.

## Install Go

Right now, the tooling expects you to have [Go][golang] installed.  If
you don't have it already installed, you can follow the official
guide [here][install-golang], or if you're on a Linux system, you can
run the commands below:

```bash
$ curl -LOs https://go.dev/dl/go1.17.6.linux-amd64.tar.gz
$ sudo tar -C /usr/local -xzf go1.17.6.linux-amd64.tar.gz
$ echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
$ export PATH=$PATH:/usr/local/go/bin
```

> You might need to adjust where you place the `PATH` update based on
> your shell, but that's outside the scope of this article.

## Install `kind`

[`kind`][kind] is a helper binary for running Kubernetes in Docker.  It
is installed with Go, and doing so is straightforward!  Simply run the
following command:

```bash
$ go install sigs.k8s.io/kind@v0.11.1
```

That's it!

## Install `kubectl`

`kubectl` is how we'll interact with our Kubernetes cluster and, more
importantly, how we'll log into our routers.  On Linux, you can install
it with the following command:

```bash
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
$ chmod +x kubectl
$ mv kubectl ~/.local/bin
$ kubectl version --client
```

> The above assumes you have `~/.local/bin` somewhere in your path.  If
> you don't, simply move `kubectl` to somewhere that is in your path,
> such as `/usr/local/bin`.

If you're not on Linux, or if you run into issues or want more a more
detailed guide, check the
[official `kubectl` installation instructions][install-kubectl].

## Verify Prerequisites

Once you've installed the prerequisites, you can run through the
following commands to ensure they're functioning:

* Docker: `docker run hello-world`
* `go`: `go version`
* `kind`: `kind version`
* `kubectl`: `kubectl version --client`

All of the above commands should be successful.  I've run them with my
output below as an example:

```bash
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:507ecde44b8eb741278274653120c2bf793b174c06ff4eaa672b713b3263477b
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

$ go version
go version go1.17.6 linux/amd64
$ kind version
kind v0.11.1 go1.17.6 linux/amd64
$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.1", GitCommit:"86ec240af8cbd1b60bcc4c03c20da9b98005b92e", GitTreeState:"clean", BuildDate:"2021-12-16T11:41:01Z", GoVersion:"go1.17.5", Compiler:"gc", Platform:"linux/amd64"}
$
```

# Install `kne`

Now that we have prerequisites, it's time to install `kne`!  Again, this
is a very straightforward process:

```bash
$ go install github.com/google/kne/kne_cli@latest
```

This will install `kne_cli` to `$GOPATH/bin`.  On my system, that's
`~/go/bin`, and I have that in my `$PATH`.  If you don't, then you can
either add it to your path _or_ move it somewhere that is in your path.

```bash
# Add `~/go/bin/` to your path:
$ echo "export PATH=$PATH:$HOME/go/bin" >> ~/.profile
$ export PATH=$PATH:$HOME/go/bin" >> ~/.profile

# OR

# Move `kne_cli` to somewhere in your `PATH`, such as `~/.local/bin`:
$ mv ~/go/bin/kne_cli ~/.local/bin
```

> Personally, I don't love the binary name being `kne_cli`, so I rename
> it with `mv ~/go/bin/kne_cli ~/go/bin/kne`.  However, to avoid
> confusion with the rest of the documentation and help outputs, this
> post will continue to refer to the command as `kne_cli`.


Now, to verify that it's working:

```bash
$ kne_cli help
Kubernetes Network Emulation CLI.  Works with meshnet to create
layer 2 topology used by containers to layout networks in a k8s
environment.

Usage:
  kne_cli [command]

Available Commands:
  create      Create Topology
  delete      Delete Topology
  deploy      Deploy cluster.
  help        Help about any command
  show        Show Topology
  topology    Topology commands.

Flags:
  -h, --help               help for kne_cli
      --kubecfg string     kubeconfig file (default "/home/tyler/.kube/config")
  -v, --verbosity string   log level (default "info")

Use "kne_cli [command] --help" for more information about a command.
```

# Deploy KNE

KNE has two mechanisms for deployment: the built-in command
(`kne_cli deploy`) or a list of full commands for installing to a normal
Kubernetes cluster.  We're going to use the `kne_cli deploy` method
because it does everything for you, but be aware that the scalability
may not be what you need for larger topologies or systems (like IOS-XR).

Deploy KNE is straightforward: simply download the example deployment
definition and deploy it:

```bash
$ git clone https://github.com/google/kne ~/kne
$ cd ~/kne
$ kne_cli deploy deploy/kne/kind.yaml
```

Next, verify that your deployment was successful:

```bash
$ kind get clusters
kne
$ kubectl get nodes
NAME                STATUS   ROLES                  AGE   VERSION
kne-control-plane   Ready    control-plane,master   15h   v1.22.1
```

# Install the SR Linux Controller

This post uses SR Linux as the example because it's the lowest barrier
to entry.  Although KNE supports Arista cEOS and Cisco IOS-XR, neither
of those options are freely available without going through a pay or
registration wall.  [SR Linux][srlinux], on the other hand, is publicly
available as a container image, for free, without a pay wall.  So that's
what we're going to use!

In order to use SR Linux with KNE, though, we need the SR Linux
Controller.  Fortunately, installing this is also very easy!

```bash
$ kubectl apply -k https://github.com/srl-labs/srl-controller/config/default
```

Now, make sure it's successfully installed:

```bash
$ kubectl get pods -n srlinux-controller
NAME                                                    READY   STATUS    RESTARTS      AGE
srlinux-controller-controller-manager-fb7848755-nj9tl   2/2     Running   2 (71m ago)   15h
```

> It might take a few minutes.  You can either re-run the command until
> you see `2/2` under `READY` and `Running` under `STATUS`, or you can
> add `-w` to the command to refresh whenever the status of the pod
> changes (i.e., `kubectl get pods -n srlinux-controller -w`).

# Deploy a Sample Topology

KNE comes with some sample topologies.  We can now deploy one of these
with SR Linux.  The topology is a simple 3-node topology, and we'll
later configure basic IPv6 addresses and LLDP to verify operation.

Deploying the topology is (again) very simple:

```bash
$ cd ~/kne
$ kne_cli create examples/srlinux/3node-srl.pbtxt
```

This will create the SR Linux virtual routers in the `3node-srlinux`
namespace.  In another terminal window, we can watch the status of those
pods in a similar manner as we did for watching the controller:

```bash
$ kubectl get pods -n 3node-srlinux -w
```

Once everything is running, we can log into our routers!

# Log In and Configure Interfaces

Okay, this is the part where things _might_ stop being so easy.  SR
Linux isn't like IOS, NX-OS, EOS, IOS-XR, or Junos.  If you're
unfamiliar with SR Linux, it's very similar to the old Alcatel-Lucent
TiMOS.  And if you're not familiar with that, this might be a bit of
an interesting configuration experience.  It is a _fantastic_ model,
though, and has always been one of my favorite systems to manage and
configure because of its hierarchical nature.  Everything
_just makes sense_ to me.  Anyway, enough about that, let's log in!

To log in, we use `kubectl`!

```bash
$ kubectl exec -it r1 -n 3node-srlinux -- sr_cli
Defaulted container "r1" out of: r1, init-r1 (init)
Using configuration file(s): ['/etc/opt/srlinux/srlinux.rc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.
--{ [FACTORY] running }--[  ]--
A:r1#
```

To log out, the command is simply `quit`.

## SR Linux Basics

The most important thing to know is that SR Linux has a concept of
contexts.  When you first login, you're in the `running` context, which
is the active `running` configuration.  If you want to make a
configuration change, you need to enter the `candidate` context.  This
two-stage configuration style will be somewhat familiar to IOS-XR and
Junos operators, except instead of typing `configure`, you type
`enter candidate`.  To view the configuration, you can simply type
`info`.  For a specific level of hierarchy, you can add that to the end
of the `info` command, e.g., `info interface mgmt0`:

```bash
A:r1# info interface mgmt0
    interface mgmt0 {
        admin-state enable
        subinterface 0 {
            admin-state enable
            ipv4 {
                dhcp-client {
                }
            }
            ipv6 {
                dhcp-client {
                }
            }
        }
    }
--{ [FACTORY] running }--[  ]--
A:r1#
```

The style looks similar to Junos, including hierarchy, curly braces, and
mandatory subinterfaces (`unit` from Junos) for logical configuration.

There are also `show` commands and even a `show` context.

```bash
--{ [FACTORY] running }--[  ]--
A:r1# show interface brief
+---------------------+-----------------------------------+-----------------------------------+-----------------------------------+-----------------------------------+
|        Port         |            Admin State            |            Oper State             |               Speed               |               Type                |
+=====================+===================================+===================================+===================================+===================================+
| ethernet-1/1        | disable                           | down                              | 100G                              |                                   |
| ethernet-1/2        | disable                           | down                              | 100G                              |                                   |
| ethernet-1/3        | disable                           | down                              | 100G                              |                                   |

<SNIP>
```

## Configure an IPv6 Address

As mentioned above, we need to enter the `candidate` context to make
configuration changes, so let's do that.  Use the following commands to
configure an IPv6 address on `ethernet-1/1` on `r1`:

```bash
enter candidate
interface ethernet-1/1
subinterface 0
ipv6 address 2001:db8:1::1/64
/
diff
commit now
```

Example output:

```bash
A:r1# enter candidate
--{ [FACTORY] candidate shared default }--[  ]--
A:r1# interface ethernet-1/1
--{ [FACTORY] * candidate shared default }--[ interface ethernet-1/1 ]--
A:r1# subinterface 0
--{ [FACTORY] * candidate shared default }--[ interface ethernet-1/1 subinterface 0 ]--
A:r1# ipv6 address 2001:db8:1::1/64
--{ [FACTORY] * candidate shared default }--[ interface ethernet-1/1 subinterface 0 ipv6 address 2001:db8:1::1/64 ]--
A:r1# /
--{ [FACTORY] * candidate shared default }--[  ]--
A:r1# diff
+     interface ethernet-1/1 {
+         subinterface 0 {
+             ipv6 {
+                 address 2001:db8:1::1/64 {
+                 }
+             }
+         }
+     }
--{ [FACTORY] * candidate shared default }--[  ]--
A:r1# commit now
All changes have been committed. Leaving candidate mode.
--{ [FACTORY] + running }--[  ]--
A:r1# info interface ethernet-1/1
    interface ethernet-1/1 {
        admin-state enable
        subinterface 0 {
            ipv6 {
                address 2001:db8:1::1/64 {
                }
            }
        }
    }
--{ [FACTORY] + running }--[  ]--
A:r1#
```

We can use `show interface ethernet-1/1` to get the operational status:

```bash
--{ [FACTORY] + running }--[  ]--
A:r1# show interface ethernet-1/1
=======================================================================================================================================================================
ethernet-1/1 is up, speed 100G, type None
  ethernet-1/1.0 is up
    Network-instance:
    Encapsulation   : null
    Type            : routed
    IPv6 addr    : 2001:db8:1::1/64 (static, unknown)
    IPv6 addr    : fe80::74:2eff:feff:0/64 (link-layer, unknown)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
=======================================================================================================================================================================
--{ [FACTORY] + running }--[  ]--
A:r1#
```

Now, let's do the same on `r2`.  First, use `quit` to logout of `r1`,
then use `kubectl` to log into `r2` and configure its `ethernet-1/1`
interface:

```bash
kubectl exec -it r2 -n 3node-srlinux -- sr_cli
enter candidate
interface ethernet-1/1 subinterface 0 ipv6 address 2001:db8:1::2/64
/
diff
commit now
```

Example output:

```bash
$ kubectl exec -it r2 -n 3node-srlinux -- sr_cli
Defaulted container "r2" out of: r2, init-r2 (init)
Using configuration file(s): ['/etc/opt/srlinux/srlinux.rc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.
--{ running }--[  ]--
A:r2# enter candidate
--{ candidate shared default }--[  ]--
A:r2# interface ethernet-1/1 subinterface 0 ipv6 address 2001:db8:1::2/64
--{ * candidate shared default }--[ interface ethernet-1/1 subinterface 0 ipv6 address 2001:db8:1::2/64 ]--
A:r2# /
--{ * candidate shared default }--[  ]--
A:r2# diff
+     interface ethernet-1/1 {
+         subinterface 0 {
+             ipv6 {
+                 address 2001:db8:1::2/64 {
+                 }
+             }
+         }
+     }
--{ * candidate shared default }--[  ]--
A:r2# commit now
All changes have been committed. Leaving candidate mode.
--{ + running }--[  ]--
A:r2#
```

Now that we've configured our IPv6 addresses, we should be able to use
our interfaces, right?  Wrong!  Unlike other vendors operating systems,
SR Linux does not place interfaces or configurations into a default
routing instance.  We have to do that ourselves.  So, on each of our
routers, we need to create an instance and add our interface to it.  If
you're familiar with `routing-instance` on Junos, this is similar --
just one extra step to get something into an instance.  I'll name a new
instance called `main`, but you can also use the `default` instance:

```bash
enter candidate
network-instance main
interface ethernet-1/1.0
/
diff
commit now
```

Example output for both routers, including using `kubectl` to log in:

```bash
$ kubectl exec -it r1 -n 3node-srlinux -- sr_cli
Defaulted container "r1" out of: r1, init-r1 (init)
Using configuration file(s): ['/etc/opt/srlinux/srlinux.rc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.
--{ [FACTORY] + running }--[  ]--
A:r1# enter candidate
--{ [FACTORY] + candidate shared default }--[  ]--
A:r1# network-instance main
--{ [FACTORY] +* candidate shared default }--[ network-instance main ]--
A:r1# interface ethernet-1/1.0
--{ [FACTORY] +* candidate shared default }--[ network-instance main interface ethernet-1/1.0 ]--
A:r1# /
--{ [FACTORY] +* candidate shared default }--[  ]--
A:r1# diff
+     network-instance main {
+         interface ethernet-1/1.0 {
+         }
+     }
--{ [FACTORY] +* candidate shared default }--[  ]--
A:r1# commit now
All changes have been committed. Leaving candidate mode.
--{ [FACTORY] + running }--[  ]--
A:r1# quit

$ kubectl exec -it r2 -n 3node-srlinux -- sr_cli
Defaulted container "r2" out of: r2, init-r2 (init)
Using configuration file(s): ['/etc/opt/srlinux/srlinux.rc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.
--{ [FACTORY] + running }--[  ]--
A:r2# enter candidate
--{ [FACTORY] + candidate shared default }--[  ]--
A:r2# network-instance main interface ethernet-1/1.0
--{ [FACTORY] +* candidate shared default }--[ network-instance main interface ethernet-1/1.0 ]--
A:r2# /
--{ [FACTORY] +* candidate shared default }--[  ]--
A:r2# diff
+     network-instance main {
+         interface ethernet-1/1.0 {
+         }
+     }
--{ [FACTORY] +* candidate shared default }--[  ]--
A:r2# commit now
All changes have been committed. Leaving candidate mode.
--{ [FACTORY] + running }--[  ]--
```

Now, let's verify that we can ping from one host to another!

```bash
$ kubectl exec -it r1 -n 3node-srlinux -- sr_cli
A:r1# ping6 2001:db8:1::2 network-instance main -c 3
Using network instance main
PING 2001:db8:1::2(2001:db8:1::2) 56 data bytes
64 bytes from 2001:db8:1::2: icmp_seq=1 ttl=64 time=37.4 ms
64 bytes from 2001:db8:1::2: icmp_seq=2 ttl=64 time=34.9 ms
64 bytes from 2001:db8:1::2: icmp_seq=3 ttl=64 time=33.8 ms

--- 2001:db8:1::2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 33.801/35.384/37.444/1.525 ms
--{ [FACTORY] + running }--[  ]--
A:r1#
```

## Configure LLDP

With SR Linux, we don't get LLDP enabled everywhere.  Now that we have
a little more comfort with SR Linux, let's enable LLDP on our interfaces
and verify its operation.  On both routers:

```bash
enter candidate
system lldp interface ethernet-1/1
/
diff
commit now
```

Example outputs, including the usage of `kubectl`, for both routers:

```bash
$ kubectl exec -it r1 -n 3node-srlinux -- sr_cli
Defaulted container "r1" out of: r1, init-r1 (init)
Using configuration file(s): ['/etc/opt/srlinux/srlinux.rc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.
--{ [FACTORY] + running }--[  ]--
A:r1# enter candidate
--{ [FACTORY] + candidate shared default }--[  ]--
A:r1# system lldp interface ethernet-1/1
--{ [FACTORY] +* candidate shared default }--[ system lldp interface ethernet-1/1 ]--
A:r1# /
--{ [FACTORY] +* candidate shared default }--[  ]--
A:r1# diff
      system {
+         lldp {
+             interface ethernet-1/1 {
+             }
+         }
      }
--{ [FACTORY] +* candidate shared default }--[  ]--
A:r1# commit now
All changes have been committed. Leaving candidate mode.
--{ [FACTORY] + running }--[  ]--
A:r1# quit

$ kubectl exec -it r2 -n 3node-srlinux -- sr_cli
Defaulted container "r2" out of: r2, init-r2 (init)
Using configuration file(s): ['/etc/opt/srlinux/srlinux.rc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.
--{ + running }--[  ]--
A:r2# enter candidate
--{ + candidate shared default }--[  ]--
A:r2# system lldp interface ethernet-1/1
--{ +* candidate shared default }--[ system lldp interface ethernet-1/1 ]--
A:r2# /
--{ +* candidate shared default }--[  ]--
A:r2# diff
      system {
+         lldp {
+             interface ethernet-1/1 {
+             }
+         }
      }
--{ +* candidate shared default }--[  ]--
A:r2# commit now
All changes have been committed. Leaving candidate mode.
--{ + running }--[  ]--
A:r2# quit
```

Finally, let's verify with the `show system lldp neighbor` command:

```bash
$ k exec -it r2 -n 3node-srlinux -- sr_cli
Defaulted container "r2" out of: r2, init-r2 (init)
Using configuration file(s): ['/etc/opt/srlinux/srlinux.rc']
Welcome to the srlinux CLI.
Type 'help' (and press <ENTER>) if you need any help using this.
--{ + running }--[  ]--
A:r2# show system lldp neighbor
  +--------------+-------------------+---------------------+---------------------+---------------------+--------------------+---------------+-------------+-----------+
  |     Name     |     Neighbor      |   Neighbor System   | Neighbor Chassis ID |   Neighbor First    |   Neighbor Last    | Neighbor Port | BGP GroupId | BGP Peers |
  |              |                   |        Name         |                     |       Message       |       Update       |               |             |           |
  +==============+===================+=====================+=====================+=====================+====================+===============+=============+===========+
  | ethernet-1/1 | 02:74:2E:FF:00:00 | r1                  | 02:74:2E:FF:00:00   | a minute ago        | 21 seconds ago     | ethernet-1/1  |             |           |
  +--------------+-------------------+---------------------+---------------------+---------------------+--------------------+---------------+-------------+-----------+
--{ + running }--[  ]--
A:r2#
```

## Final Thoughts on SR Linux

Look, I've been a fan of Nokia's routers since before they were Nokia.
I previously worked on them when they were Alcatel-Lucent running TiMOS.
They were the most stable devices I had ever used, and I gotta say, the
improvements that are in SR Linux make the OS even better than it was.
There's a huge range of help available in the CLI.  You can even get
some really useful information on arguments, like when configuring IPv6,
you might notice the following if you use `?`:

```bash
Local commands:
  primary*          One of the IPv6 prefixes assigned to the subinterface can be explicitly configured as primary by setting this leaf to true. This designates the
                    associated IPv6 address as a primary IPv6 address of the subinterface. By default, the numerically lowest value IPv6 address is selected as the
                    primary address.
```

That's not something I looked up in their docs.  It's available directly
from the CLI.  This is how the network operator experience should be.
Oh, and if you don't know where to find or configure something, have a
look at the `tree` command:

```bash
A:r2# tree /system lldp
lldp
+-- admin-state
+-- hello-timer
+-- hold-multiplier
+-- trace-options
+-- bgp-auto-discovery
|   +-- admin-state
|   +-- group-id
|   +-- network-instance
+-- management-address
|   +-- type
+-- interface
    +-- admin-state
    +-- bgp-auto-discovery
        +-- admin-state
        +-- group-id
        +-- peering-address
--{ + running }--[  ]--
A:r2#
```

So now you know everything that can be configured under `/system/lldp`.
Want to know specifics about one of those items?  Just use `?` after the
element:

```bash
--{ + running }--[  ]--
A:r2# system lldp hello-timer ?

usage: hello-timer

System level hello timer for the LLDP protocol

[number64, range 0..18446744073709551615, units seconds, default 30]

Output modifier commands:
  #                 Comment out the rest of the line
  >                 Redirect output to a file
  >>                Append output to a file
  |                 Forward output to the next command

--{ + running }--[  ]--
A:r2#
```

How is this not just the gold standard of all network operating systems?

# Final Thoughts

KNE is a quick, easy, low-barrier way to create a virtual network lab.
SR Linux does everything right and makes their images available
publicly.  It's a match made in heaven, honestly, and every vendor
hiding their containers behind a pay or registration wall should feel
shame.  Lots and lots of shame.

The possibilities here are damn near endless: integration tests, logic
validation, quick lab mockups, massive scalability in network labs
without the annoying complexity of OpenStack or VIRL or trying to figure
out how to do multi-node networking with Vagrant and KVM.

I'm a fan.  I want every vendor NOS to have a public container and
integration with KNE.  I want my entire network and my development
pipeline to be tightly integrated, easy, and flexible.  It took me 30
minutes to setup a single KNE node with SR Linux, using IPv4 for all of
the Kubernetes networking, and configure working IPv6 interfaces and
LLDP between the routers.

Was it something as complex as EVPN?  No, but if you're looking for that,
there's an [SR Linux guide][srlinux-evpn] that uses 3 nodes to create an
EVPN fabric.  You've already got three nodes if you've made it through
this -- just go through the EVPN guide, changing interfaces according to
the map below:

```bash
links: {
    a_node: "r1"
    a_int: "e1-1"
    z_node: "r2"
    z_int: "e1-1"
}
links: {
    a_node: "r1"
    a_int: "e1-2"
    z_node: "r3"
    z_int: "e1-1"
}
links: {
    a_node: "r2"
    a_int: "e1-2"
    z_node: "r3"
    z_int: "e1-2"
}
```

If you want to try this with cEOS or IOS-XR, you've gotta jump through
more hoops.  I won't cover that here.  But maybe we should just pressure
our vendors to follow Nokia's model.

[docker]: https://www.docker.com/
[install-docker]: https://docs.docker.com/engine/install/
[kubectl]: https://kubernetes.io/docs/reference/kubectl/overview/
[install-kubectl]: https://kubernetes.io/docs/tasks/tools/#kubectl
[kind]: https://kind.sigs.k8s.io/
[kne]: https://github.com/google/kne
[golang]: https://go.dev/
[install-golang]: https://go.dev/doc/install
[srlinux]: https://learn.srlinux.dev/
[srlinux-evpn]: https://learn.srlinux.dev/tutorials/l2evpn/fabric/#leaf-spine-interfaces
