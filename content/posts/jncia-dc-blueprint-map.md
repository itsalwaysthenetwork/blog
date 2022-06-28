---
title: "Mapping the JNCIA-DC Blueprint"
tags:
  - certification
  - juniper
  - jncia
  - jncia-dc
  - datacenter
publishdate: "2022-05-25"
toc: true
---

## Introduction

In this (very) short blog post, I'll provide a mapping of the
[official JNCIA-DC objectives][jncia-dc-objectives] to some resources
for learning about the topic.

> You may notice that several of these sections suggest studying the
> JNCIA-Junos or JNCIS-ENT.  That's because there's a _lot_ of overlap.
> Honestly, the right approach is probably to get your JNCIA-Junos and
> JNCIS-ENT before attempting the JNCIA-DC, but if you're impatient,
> then hopefully this mapping helps.

## Data Center Architectures

### Identify concepts and general features of Data Center architectures

#### Traditional Architectures (multi-tier)

[Juniper Open Learning: Design, Associate (JNCDA)][jncda-open]

[Day One: Data Center Fundamentals][day-one-dc-fundamentals], Chapter 5

#### IP-Fabric Architectures (Spine/Leaf)

[Juniper Open Learning: Design, Associate (JNCDA)][jncda-open]

[Day One: Data Center Fundamentals][day-one-dc-fundamentals], Chapters 5 & 6

#### Layer 2 and Layer 3 strategies

> I have no idea what this is supposed to mean according to the
> blueprint.  Maybe traditional "core-distribution-access" vs.
> spine-leaf?

#### Overlay Network versus Underlay Network (Very Basic)

[Day One: Data Center Fundamentals][day-one-dc-fundamentals], Chapter 7

#### EVPN/VXLAN basics/purpose

[Day One: Data Center Fundamentals][day-one-dc-fundamentals], Chapters 7 & 9

## Layer 2 Switching, VLANs and Security

### Identify the concepts, operation, or functionality of Layer 2 switching for the Junos OS

#### Ethernet switching/bridging concepts and operations

[Juniper Open Learning: JNCIA-Junos][jncia-junos-open]

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Layer 2 Networking][understanding-l2]

### Identify the concepts, benefits, or functionality of VLANs

#### Port modes

[Juniper Open Learning: JNCIA-Junos][jncia-junos-open]

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

#### VLAN Tagging

[Juniper Open Learning: JNCIA-Junos][jncia-junos-open]

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

#### IRB

[Integrated Bridging and Routing][irb], `Understanding Integrated Routing and Bridging`, `Configuring IRB Interfaces on Swithces`, and `Configuring Integrated Routing and Bridging for VLANs` 

### Identify the concepts, benefits, or functionality of Layer 2 Security

#### MACsec

[Day One: MACsec][day-one-macsec], Chapters 1 & 2

[Understanding Media Access Control Security (MACsec)][understanding-macsec]

#### MAC address control/filtering

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[MAC Address Filtering and Accounting on Ethernet Interfaces][mac-filtering], `Configuring MAC Address Filtering for Ethernet Interfaces`

[Configuring MAC Limiting][mac-limiting], `Configuring MAC Limiting (QFX Switches)`

#### Storm Control

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Understanding Storm Control][storm-control]

### Describe how to configure, monitor, or troubleshoot Layer 2 switching, VLANs, or security

#### Ethernet switching/bridging

[Layer 2 Networking][understanding-l2]

[Configuring Layer 2 Briding Interfaces][configuring-irb]

[Integrated Bridging and Routing][irb], `Understanding Integrated Routing and Bridging`, `Configuring IRB Interfaces on Swithces`, and `Configuring Integrated Routing and Bridging for VLANs` 

#### VLANs

[Bridging and VLANs][bridging-and-vlans], All sections _except_ non-ELS sections and `VPLS Ports`

#### Layer 2 security features

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Understanding Persistent MAC Learning][sticky-mac], `Understanding Persistent MAC Learning (Sticky MAC)` and `Configuring Persistent MAC Learning (ELS)`

[Configuring MAC Limiting][mac-limiting], `Configuring MAC Limiting (QFX Switches)`

[MAC Address Filtering and Accounting on Ethernet Interfaces][mac-filtering], `Configuring MAC Address Filtering for Ethernet Interfaces`

[Understanding Storm Control][storm-control]

[Enabling and Disabling Storm Control (ELS)][configure-storm-control]

## Protocol-Independent Routin

### Identify the concepts, operation, or functionality of various protocol-independent routing components

#### Static, aggregate, and generated routes

[Configure Static Routes][configure-static-routes], `Understanding Basic Static Routing`

[Configuring Route Aggregation][configuring-route-aggregation], `Understanding Route Aggregation`

[Juniper Open Learning: JNCIA-Junos][jncia-junos-open]

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Junos: Aggragate Routes vs Generate Routes][aggrevate-vs-generated-routes] (\*)

#### Martian addresses

[Recognize Martian Addresses for Routing][understanding-martian]

[Juniper Open Learning: JNCIA-Junos][jncia-junos-open]

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

#### Routing instances, including RIB groups

[Routing Instances Overview][routing-instances-overview]

[Juniper Day One Poster: Using RIB Groups for Route Sharing in the Junos OS][day-one-rib-groups]

[Juniper Open Learning: JNCIA-Junos][jncia-junos-open]

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Junos RIB Groups (1/2)][junos-rib-groups-1-2] (\*)

[Junos RIB Groups (2/2)][junos-rib-groups-2-2] (\*)

#### Load balancing

[Juniper Open Learning: JNCIA-Junos][jncia-junos-open]

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Understanding Per-Packet Load Balancing][understanding-per-packet]

#### Filter-based forwarding

[Filter-Based Forwarding Overview][fbf-overview]

[Example: Configuring Filter-Based Forwarding on the Source Address][fbf-example]

### Describe how to configure, monitor, or troubleshoot various protocol-independent routing components

#### Static, aggregate, and generated routes

[Configure Static Routes][configure-static-routes]

[Configuring Route Aggregation][configuring-route-aggregation]

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Day One: Routing the Internet Protocol][day-one-routing-ip], Chapters 1 & 8

[Junos: Aggragate Routes vs Generate Routes][aggrevate-vs-generated-routes] (\*)

#### Load balancing

[Configuring Per-Packet Load Balancing][configuring-per-packet]

## Data Center Routing Protocols BGP/OSPF

### Identify the concepts, operation, or functionality of OSPF

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Day One: Routing the Internet Protocol][day-one-routing-ip], Chapters 2 & 4

#### Link-state database

[Introduction to OSPF][intro-to-ospf], `OSPF Overview`

#### OSPF packet types

[Introduction to OSPF][intro-to-ospf], `OSPF Packets Overview`

#### Router ID

[Introduction to OSPF][intro-to-ospf], `OSPF Overview`

#### Adjacencies and neighbors

[Introduction to OSPF][intro-to-ospf], `OSPF Overview`

#### Designated router (DR) and backup designated router (BDR)

[Configuring OSPF Areas][ospf-areas-config], `OSPF Designated Router Overview`

#### OSPF area and router types

[Configuring OSPF Areas][ospf-areas-config], `Understanding OSPF Areas`, `OSPF Designated Router Overview`, `Understanding OSPF Areas and Backbone Areas`, and `Understanding OSPF Stub Areas, Totally Stubby Areas, and Not-So-Stubby Areas`

#### LSA packet types

[Introduction to OSPF][intro-to-ospf], `OSPF Packets Overview`

### Describe how to configure, monitor, or troubleshoot OSPF

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Day One: Routing the Internet Protocol][day-one-routing-ip], Chapter 4

#### Areas, interfaces, and neighbors

[Configuring OSPF Interfaces][ospf-interfaces-config]

[Configuring OSPF Areas][ospf-areas-config]

#### Additional basic options

[Configuring OSPF Authentication][ospf-auth-config]

#### Routing policy application

[Configuring OSPF Routing Policy][ospf-route-policy-config]

#### Troubleshooting tools

[Troubleshooting Network Issues][tshoot-ospf]

### Identify the concepts, operation, or functionality of BGP

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Day One: Routing the Internet Protocol][day-one-routing-ip], Chapter 7

#### BGP basic operation

[BGP Overview][bgp-overview], `Understanding BGP`, `BGP Routes Overview`, and `BGP Route Resolution Overview`

#### BGP message types

[BGP Overview][bgp-overview], `BGP Messages Overview`

#### Attributes

[Autonomous Systems for BGP][bgp-asns], `Understanding the BGP Local AS Attribute` and `Understanding the Accumulated IGP Attribute for BGP`

[Local Preference for BGP Routes][bgp-local-pref], `Understanding the Local Preference Metric for Internal BGP Routes`

[BGP MED Attribute][bgp-med], `Understanding the MED Attribute That Determines the Exit Point in an AS`

#### Route/path selection process

[BGP Overview][bgp-overview], `Understanding BGP Path Selection`

#### IBGP and EBGP functionality and interaction

[BGP Overview][bgp-overview], `Understanding BGP`

[BGP Peering Sessions][bgp-routing-sessions], `Understanding External BGP Peering Sessions` and `Understanding Internal BGP Peering Sessions`

### Describe how to configure, monitor, or troubleshoot BGP

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

[Troubleshooting Network Issues][bgp-tshoot]

[Troubleshooting BGP Sessions][bgp-tshoot-sessions]

#### Groups and peers

[BGP Peering Sessions][bgp-routing-sessions]

#### Additional basic options

[BGP Route Authentication][bgp-route-auth]

#### Routing policy application

[Basic BGP Routing Policies][bgp-routing-policies]

## High Availability (HA)

[Juniper Open Learning: JNCIA-Junos][jncia-junos-open]

[Juniper Open Learning: JNCIS-ENT][jncis-ent-open]

### Identify the concepts, benefits, applications, or requirements of high availability

#### Link aggregation groups (LAG)

[Aggregated Ethernet Interfaces][lags], `Understanding Aggregated Ethernet Interfaces and LACP for Switches`

#### Graceful restart (GR)

[Graceful Restart Concepts][gr-concepts]

[Understanding Graceful Restart for BGP][bgp-gr], `Understanding the Long-Lived BGP Graceful Restart Capability`

[Configuring Graceful Restart for OSPF][ospf-gr], `Graceful Restart for OSPF Overview`

#### Bidirectional Forwarding Detection (BFD)

[Understanding How BFD Detects Network Failures][bfd-understanding]

#### Virtual Chassis

[Understanding QFX Series Virtual Chassis][virtual-chassis]

### Describe how to configure, monitor, or troubleshoot high availability components

#### Link aggregation groups (LAG)

[Aggegrated Ethernet Interfaces][lags], `Forcing LAG Links or Interfaces with Limited LACP Capability to Be Up`, `Configuring an Aggregated Ethernet Interface`, `Configuring the Number of Aggregated Ethernet Interfaces on the Device (Enhanced Layer 2 Software)`, and `Troubleshooting an Aggregated Ethernet Interface`

#### Graceful restart (GR)

[Configuring Graceful Restart for OSPF][ospf-gr], `Example: Configuring Graceful Restart for OSPF`

[Configuring Graceful Restart][gr-config]

#### Bidirectional Forwarding Detection (BFD)

[Configuring BFD][bfd-config], `Example: Configuring BFD for Static Routes for Faster Network Failure Detection`, `Example: Configuring BFD on Internal BGP Peer Sessions`, and `Example: Configuring BFD for OSPF`

\*: Not from Juniper directly as finding clear and concise free content pertaining to the topic was challenging.

[understanding-macsec]: https://www.juniper.net/documentation/us/en/software/junos/security-services/topics/topic-map/understanding_media_access_control_security_qfx_ex.html
[jncda-open]: https://learningportal.juniper.net/juniper/user_activity_info.aspx?id=EDU-JUN-WBT-JOL-JNCDA
[jncia-junos-open]: https://learningportal.juniper.net/juniper/user_activity_info.aspx?id=EDU-JUN-WBT-JOL-JNCIA-JUNOS
[jncis-ent-open]: https://learningportal.juniper.net/juniper/user_activity_info.aspx?id=EDU-JUN-WBT-JOL-JNCIS-ENT
[jncia-dc-objectives]: jncia-dc-objective://www.juniper.net/us/en/training/certification/tracks/data-center/jncia-dc.html 
[day-one-dc-fundamentals]: https://www.juniper.net/documentation/en_US/day-one-books/DC_Fundamentals.pdf
[day-one-macsec]: https://www.juniper.net/documentation/en_US/day-one-books/DO_MACsec_UR.pdf
[irb]: https://www.juniper.net/documentation/us/en/software/junos/multicast-l2/topics/topic-map/irb-and-bridging.html
[mac-filtering]: https://www.juniper.net/documentation/us/en/software/junos/interfaces-ethernet/topics/topic-map/mac-address-filtering-accounting-ethernet-interfaces.html
[storm-control]: https://www.juniper.net/documentation/us/en/software/junos/security-services/topics/concept/rate-limiting-storm-control-understanding.html
[mac-limiting]: https://www.juniper.net/documentation/us/en/software/junos/security-services/topics/topic-map/configuring-mac-limiting.html#id-configuring-mac-limiting-qfx-switches
[sticky-mac]: https://www.juniper.net/documentation/us/en/software/junos/security-services/topics/topic-map/understanding_and_using_persistent_mac_learning.html
[configure-storm-control]: https://www.juniper.net/documentation/us/en/software/junos/security-services/topics/task/rate-limiting-storm-control-disabling-cli-els.html
[bridging-and-vlans]: https://www.juniper.net/documentation/us/en/software/junos/multicast-l2/topics/topic-map/bridging-and-vlans.html
[understanding-l2]: https://www.juniper.net/documentation/us/en/software/junos/multicast-l2/topics/topic-map/layer-2-understanding.html
[configuring-irb]: https://www.juniper.net/documentation/us/en/software/junos/multicast-l2/topics/task/interfaces-configuring-layer-2-bridging-interfaces.html
[configure-static-routes]: https://www.juniper.net/documentation/us/en/software/junos/static-routing/topics/topic-map/config_static-routes.html
[configuring-route-aggregation]: https://www.juniper.net/documentation/us/en/software/junos/static-routing/topics/topic-map/config-route-aggregation.html
[aggrevate-vs-generated-routes]: https://www.networkfuntimes.com/junos-aggregate-routes-vs-generate-routes-how-to-summarise-on-juniper-routers/
[understanding-martian]: https://www.juniper.net/documentation/us/en/software/junos/static-routing/topics/topic-map/recognize-martian-addr-routing.html
[day-one-rib-groups]: https://www.juniper.net/assets/fr/fr/local/pdf/books/day-one-poster-rib-groups.pdf
[routing-instances-overview]: https://www.juniper.net/documentation/us/en/software/junos/routing-overview/topics/concept/routing-instances-overview.html
[junos-rib-groups-1-2]: https://momcanfixanything.com/junos-rib-groups-1-2/
[junos-rib-groups-2-2]: https://momcanfixanything.com/junos-rib-groups-2-2/
[understanding-per-packet]: https://www.juniper.net/documentation/us/en/software/junos/sampling-forwarding-monitoring/topics/concept/policy-per-packet-load-balancing-overview.html
[configuring-per-packet]: https://www.juniper.net/documentation/us/en/software/junos/sampling-forwarding-monitoring/topics/concept/policy-configuring-per-packet-load-balancing.html
[fbf-overview]: https://www.juniper.net/documentation/us/en/software/junos/routing-policy/topics/concept/firewall-filter-option-filter-based-forwarding-overview.html
[fbf-example]: https://www.juniper.net/documentation/us/en/software/junos/routing-policy/topics/example/firewall-filter-option-filter-based-forwarding-example.html
[day-one-routing-ip]: https://www.juniper.net/documentation/en_US/day-one-books/DO_Routing_the_IP.pdf
[intro-to-ospf]: https://www.juniper.net/documentation/us/en/software/junos/ospf/topics/topic-map/ospf-overview.html
[tshoot-ospf]: https://www.juniper.net/documentation/us/en/software/junos/ospf/bgp/topics/topic-map/troubleshooting-network-issues.html
[ospf-route-policy-config]: https://www.juniper.net/documentation/us/en/software/junos/ospf/topics/topic-map/configuring-ospf-routing-policy.html
[ospf-interfaces-config]: https://www.juniper.net/documentation/us/en/software/junos/ospf/topics/topic-map/configuring-ospf-interfaces.html
[ospf-areas-config]: https://www.juniper.net/documentation/us/en/software/junos/ospf/topics/topic-map/configuring-ospf-areas.html
[ospf-auth-config]: https://www.juniper.net/documentation/us/en/software/junos/ospf/topics/topic-map/configuring-ospf-authentication.html
[bgp-overview]: https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/bgp-overview.html
[bgp-asns]: https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/autonomous-systems.html
[bgp-local-pref]: https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/local-preference.html
[bgp-med]: https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/med-attribute.html
[bgp-routing-policies]: https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/basic-routing-policies.html
[bgp-routing-sessions]: https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/bgp-peering-sessions.html
[bgp-tshoot]: https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/troubleshooting-network-issues.html
[bgp-tshoot-session]: https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/troubleshooting-bgp-sessions.html
[bgp-route-auth]: https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/bgp_security.html
[bgp-gr]: https://www.juniper.net/documentation/us/en/software/junos/bgp/topics/topic-map/bgp-long-lived-graceful-restart.html
[ospf-gr]: https://www.juniper.net/documentation/us/en/software/junos/ospf/topics/topic-map/configuring-graceful-restart-for-ospf.html
[gr-concepts]: https://www.juniper.net/documentation/us/en/software/junos/high-availability/topics/concept/graceful-restart-concepts.html
[gr-config]: https://www.juniper.net/documentation/us/en/software/junos/high-availability/topics/example/graceful-restart-configuring-example.html
[bfd-understanding]: https://www.juniper.net/documentation/us/en/software/junos/high-availability/topics/topic-map/bfd.html
[bfd-config]: https://www.juniper.net/documentation/us/en/software/junos/high-availability/topics/topic-map/bfd-configuring.html
[lags]: https://www.juniper.net/documentation/us/en/software/junos/interfaces-ethernet-switches/topics/topic-map/switches-interface-aggregated.html
[virtual-chassis]: https://www.juniper.net/documentation/us/en/software/junos/virtual-chassis-qfx/topics/concept/virtual-chassis-qfx-series-understanding.html
