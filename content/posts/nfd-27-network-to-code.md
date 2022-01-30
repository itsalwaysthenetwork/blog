---
title: "NFD27 - Network to Code"
tags:
  - nfd27
  - ntc
  - nautobot
---

> Preamble time!  [Nautobot][nautobot] vs. [NetBox][netbox] has been a
> sensitive subject in the community.  I'm not here to sway you or
> endorse one over the other.  Use what you think fits you and your
> organization best.

After [ZPE][zpe-post], [Network to Code][ntc] was up to talk to us about
all of the latest with Nautobot.  The presentations were cohesive, and
they told a compelling story: you've got stuff to do, and we help you do
those things.  Maybe the most interesting thing about their presentation
was that they really only covered plugins -- there was no new core
whizbang product or platform announced here, just "here's more stuff
you can have for free if you use Nautobot."

# Circuit Maintenance Plugin

First up was the [Circuit Maintenance Plugin][ckt-plugin].  This plugin
is very similar to [Janitor][janitor], a standalone Flask application
from [@NetworkAndrew][networkandrew], except that it's a plugin for
[Nautobot][nautobot].  So, if you're not a Nautobot user and/or don't
want to be one, consider looking at [Janitor][janitor] for similar
functionality.

But if you are a Nautobot user -- or if you are just curious about its
features -- then the [Circuit Maintenance Plugin][ckt-plugin] looks like
it solves a serious pain point.

Keeping track of circuit maintenances is tedious work.  _Someone_ has to
do it, though -- or maybe some _thing_.  This plugin reads e-mail
inboxes to find and parse carrier circuit maintenance notifications.  It
generates and tracks initial notifications and updates, then records
those items as events in Nautobot.  It doesn't stop there, though -- if
you have your circuits in Nautobot, you can also see what impact each
maintenance will have!

> "Impact" is used lightly -- obviously it's up to the operator to know
> just how critical any given circuit is to the continuity of business.

There's even more, though.  Prometheus metrics can be configured to show
a given circuit's operational status based on the maintenance plugin's
records.  This is an _expected_ state, though, so it's important to look
at this more like a "is this circuit currently in the maintenance
window" rather than  the _actual_ operational status of that circuit.

One thing that's missing here is event-based actions.  I'd love to see
plugins like this emit an event to an event/message system so that other
integrations can simply listen for the event and then schedule or take
actions -- such as automatically removing a circuit from production use
10 minutes prior to the maintenance window and automatically returning
it to service after the window.  This is feedback I definitely provided
to them, but it's a bigger thing than just the circuit plugin since that
would be core Nautobot functionality -- which would be great to see for
other reasons.

# Single Source of Truth

I hit buzzword bingo!

All jokes aside, though, this was an interesting presentation that
perhaps is a bit of a misnomer.  The [Single Source of Truth][ssot]
plugin isn't so much a single source of truth (is anything?) as much
as it is a system for defining unidirectional data syncs.  You can
tell Nautobot to ingest data from external systems -- think of migrating
from one DCIM (such as [Device42][d42]) to Nautobot.  You can also tell
Nautobot to publish data to another system -- think of syncing IPAM to
a DNS system.  These can also be bidirectional by creating two
unidirectional syncs.

A lot of this, though, requires you to write some Python code to map
data from one system to another.  This isn't unexpected, but if I'm
making a wishlist for my ideal fantasy world, this would be something
that I could do entirely via a GUI.  However, as more people develop
plugins to this plugin, it will be much easier to use and leverage the
community efforts.

# ChatOps

I can't get excited about this one.  90% of what I saw was either not
useful to me or was something I'd rather see in a CI/CD pipeline.  Maybe
using ChatOps to kick off a pipeline would be cool, but I'm not sure.
That said, quickly getting data from your inventory is convenient.  But
getting a static Grafana graph...not so much.  If I ever share a static
graph, it's for a very specific time range to show something specific,
and at that point I need to be in my graph system to find what I'm
looking for and set the time range anyway.  Obtaining metrics data just
isn't a great use case for ChatOps.

However, an integration with Forward Networks was revealed later in NFD27
that was _extremely_ attractive: using ChatOps to do path lookups from
Forward Networks and getting that topology (nearly) instantly in Slack.
There is so much value in that that I'd use ChatOps for that one bit of
functionality alone.

# Final Thoughts

Nautobot is worth a look.  On its own, as a core thing, I don't see it
differentiating much from Netbox.  The ready-to-use plugins, though...
that's compelling as an end user who doesn't have the time or teams or
expertise to develop custom plugins like the
[Circuit Maintenance Plugin][ckt-plugin].  Coupled with the
(unfortunately named) [Single Source of Truth plugin][ssot], I can see
someone evaluating either a complete migration to Nautobot or, at worst,
an integration with other DCIMs like [Device42][d42] or even
[Netbox][netbox] itself.

Finally, if you're interested in giving Nautobot a spin, I spent a
little bit of time creating a repository to deploy some infrastructure
to DigitalOcean and deploy Nautobot to their managed Kubernetes Service
[here][nautobot-doks].

[nautobot-doks]: https://github.com/supertylerc/nautobot-doks]
[nautobot]: https://github.com/nautobot/nautobot
[netbox]: https://github.com/netbox-community/netbox
[zpe-post]: /posts/nfd-27-zpe
[ntc]: https://www.networktocode.com/
[ckt-plugin]: https://github.com/nautobot/nautobot-plugin-circuit-maintenance
[janitor]: https://github.com/wasabi222/janitor
[networkandrew]: https://twitter.com/NetworkAndrew
[ssot]: https://github.com/nautobot/nautobot-plugin-ssot
[d42]: https://www.device42.com/
