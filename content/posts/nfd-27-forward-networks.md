---
title: "NFD27 - Forward Networks"
tags:
  - nfd27
---

Day one wrapped up with [Forward Networks][fwd].  Although I've heard
their name in passing previously, I've never actually checked them out.
After seeing their presentation, I can say I wish I had looked at them
before, because there's tremendous value here.

# First Impressions

[Forward Networks][fwd] does a few things, but at a (_very_) basic
level, it combines some of the functionality of [Batfish][batfish] and
[SuzieQ][suzieq], then wraps a pretty enterprise UI on top.  It crawls
your network on a regular interval, learning and modeling your routing,
switching, underlay, overlay, and public cloud infrastructure.  These
are then made into snapshots.  Snapshots can be created on an interval
or on demand.  Once you have a snapshot, you can ask questions like
"What is the path from A to Z?".  [Forward Networks][fwd] will then
evaluate every component from A to Z to understand what the path would
be -- including ACLs, PBR, firewalls, MPLS, VxLAN, and more -- and
provide drill downs for each logical step.

This is huge.  I can't tell you how many times I've done a route lookup
at 3AM and been confused for 10 minutes -- at which point I remembered
that there's Policy-Based Routing configured on that device and the path
a packet is taking is different than what I was looking at in the RIB
and/or FIB as a result.  Forward understands this, and it even tells you
"HEY!  This is going this way because of PBR."

This data is available via a nice web interface, but they've also
created their own intuitive query language to help you ask questions
that aren't really path-related, such as "is my BGP Router ID configured
to be the same as my Loopback0 IP on all devices?"

They offered quite a few different demos, but ultimately they were just
a different lens and context for this exploratory data.

# Final Thoughts

Forward Networks is doing something exciting.  I can't wait to see them
go higher up the stack.  For now, it's all L1/L2/L3, but I'm looking
forward to the day when L4 and L7 data can be ingested and modeled.

While there's very mature public cloud support, I was disappointed to
hear that there's no Kubernetes support.  As more and more applications
move to Kubernetes, insight into load balancing, NetworkPolicy, and
Ingress can be incredibly valuable.

[fwd]: https://www.forwardnetworks.com/
[batfish]: https://www.batfish.org/
[suzieq]: https://github.com/netenglabs/suzieq

