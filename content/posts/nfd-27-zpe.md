---
title: "NFD27 - ZPE"
tags:
  - nfd27
---

The first company presenting at NFD27 was [ZPE][zpe].  They talked about
their Gen 3 Out-of-Band Management systems and the Zero Pain Ecosystem.

> In an interesting revelation, ZPE stands for Zero Point Energy, not
> Zero Pain Ecosystem.  Clever bit of marketing.

OOBM is old news.  It's "table stakes".  So what makes ZPE's OOBM
something to look at?

# First Impressions

Honestly, the presentation started out a bit of a mess.  We circled the
journey three or four times, regularly rehashing the same information.
However, they have a few unique features that set them apart -- for a
time.  While many network engineers think of OOBM in terms of the
rollover serial cable, there are other things that don't use serial but
still need OOBM.  Cisco comes to mind with things like CIMC for UCS
or Meraki's Local Status Page.  A more traditional console server, like
OpenGear's CM line, doesn't support out-of-band access to these sorts of
interfaces.  While an operator could deploy a VPN to the site, ZPE
actually supports this out of the box in the same device as the
traditional serial console.  How long will this remain a differentiator?
Probably not long -- even with OpenGear, the OM line has an x86 CPU and
can run Docker, which means you could run OpenVPN on it and connect to
those web interfaces.  The ZPE _also_ has an x86 processor, which means
that it can do anything that the OpenGear OM can.  The value with ZPE,
though, is how integrated features are when compared to the
do-it-yourself approach you'd have to take with OpenGear (or some other
competing OOBM vendor).

Perhaps a more interesting feature is that ZPE will run Java plugins for
you.  This lets you get away from those pesky Java-based OOBM
interfaces, but it raises the question of support and security.  Some of
those really annoying interfaces are old and require ancient versions of
Java with tons of vulnerabilities..

ZPE has a host of other features, like running Ansible playbooks to help
automate initial provisioning.  Unfortunately, though, so much of the
presentation was spent on the OOBM story that we didn't really get to
dive into these sorts of use cases.

# Final Thoughts

Overall, ZPE's offering is solid, but their ability to frame it was
lacking.  Competitors will eventually catch up in the areas that become
table stakes, and the differentiators will end up being niche uses or
integrations (such as environmental smoke detectors or actuators).

[zpe]: https://zpesystems.com
