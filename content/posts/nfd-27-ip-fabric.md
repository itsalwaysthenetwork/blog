---
title: "NFD27 - IP Fabric"
tags:
  - nfd27
---

Day two of NFD27 kicked off with [IP Fabric][ipf].  I hadn't heard of
them before, but I'm in North America and they're in Europe, so I guess 
that's a bit like most North American operators not being familiar with
Nokia (f.k.a Alcatel-Lucent).  We're going to see a _lot_ of overlap
with [Forward Networks][fwd], the company that wrapped up the first day
of NFD27 (and was previously covered [here][fwdnfd27]).

# First Impressions

I mentioned there's some overlap with Forward Networks, and is there
ever!  Based on the presentations, I think they're competitors.  Forward
is based in North America, whereas IP Farbic is in Europe.  I think this
is an important distinction to make that is sometimes ignored: it's
hard to sell when your timezones aren't aligned and when you have
language and cultural differences.  Global sales are hard.

But let's stop talking about trying to manage a global sales pipeline
and start talking about IP Fabric!

Let's get the buzzword out of the way: Network Assurance.  That's their
pitch, and even if Forward doesn't use that phrase, that's what Forward
is doing, too.  A lot of companies and/or products are throwing that
phrase around, too: Juniper uses the phrase when referring to its Mist
platform.  But what's special about IP Fabric?

Perhaps most importantly, they support a wide range of vendors and
technologies.  More than just traditional on-premises network devices,
IP Fabric also supports public cloud.  It leverages the APIs of those
providers and can understand network-related items such as VPCs,
firewall rules, Transit Gateway (for AWS), and more.  It can then
combine that understanding with its understanding of on-premises
networks to paint a picture of end-to-end hybrid network flows.

Like I mentioned in my thoughts on Forward, this is a powerful
capability.  As an operator of a hybrid network, I would _love_ to have
the ability to look into the path a packet takes from my on-premises
network all the way into the public cloud.  Perhaps more important than
that, though: I want to share that insight with other people at my
company.  I want to enable and empower developers, support teams,
and server/systems teams to understand what's happening when things go
wrong or how something works.

# Final Thoughts

The interface was intuitive, but the overall maturity was somewhat
lacking for my use cases.  Specifically, while they do support public
cloud providers, their support for Google Cloud Platform is not as
mature as their support for AWS.  There's also a lack of Kubernetes
support.  Maybe that's not important to everyone, but if you're in an
organization leveraging Kubernetes, this is an pretty major feature that
could be game-changing, especially for anyone new to or unfamiliar with
the internal networking.

This isn't really a dig at them,  though.  The nature of product
development is that you'll deliver some features before others, and I'd
rather have that than wait 5 years to get everything.

[fwd]: https://www.forwardnetworks.com/
[fwdnfd27]: /posts/nfd-27-forward-networks
[ipf]: https://ipfabric.io/
