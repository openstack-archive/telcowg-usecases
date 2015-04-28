..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

=============================
 Virtual IMS Core
=============================

This use case is specifically about deploying the Project Clearwater open
source IMS core in OpenStack, but is more generally representative of the
issues involved in deploying in OpenStack a scalable control plane function
deployed using a series of load-balanced N+k pools.

The use case focuses on those elements rather than more generic issues like
orchestration and HA.

Project Clearwater [1] is an open source implementation of an IMS core
designed to run in the cloud and be massively scalable.  An IMS core [2]
is one of the core services provided by many Telcos, handling VoIP device
registration and call routing.  Specifically, it provides SIP-based call
control for voice and video as well as SIP based messaging apps.  As an
IMS core Project Clearwater provides P/I/S-CSCF function together with a
BGCF and an HSS cache, and includes a WebRTC gateway providing
interworking between WebRTC & SIP clients.

This use case obsoletes the version previously hosted direct on the TelcoWG
wiki [3].

Glossary
========

* IMS - IP Multimedia Subsystem
* SIP - Session Initiation Protocol
* P/I/S-CSCF - Proxy/Interrogating/Serving Call Session Control Function
* BGCF - Breakout Gateway Control Function
* HSS - Home Subscriber Server
* WebRTC - Web Real-Time-Collaboration

Problem description
===================

Project Clearwater is mainly a compute application with modest demands on
storage and network - it provides the control plane, not the media plane
(packets typically travel point-to-point between the clients) so does not
require high packet throughput rates and is reasonably resilient to jitter and
latency.

Project Clearwater must be deployable as an HA service offering Service Level
Agreement (SLA) to end users.  Here HA refers to the availability of the
service for completing new call attempts, not for continuity of existing calls,
though as it is the control plane rather than media plane the user experience
of a Clearwater failure is that audio would continue uninterrupted but any
actions requiring signalling (e.g. conferencing in a 3rd party) would fail.

Project Clearwater must be elastically scalable, enabling an NFV orchestrator
to add and remove instances in response to applied load.

Requirements
============

The problem statement above leads to the following requirements.

* Compute application

  OpenStack already provides everything needed; in particular, there are no
  requirements for an accelerated data plane, nor for core pinning nor NUMA.

* HA

  Project Clearwater itself implements HA at the application level, consisting
  of a series of load-balanced N+k pools with no single points of failure [4].

  To meet typical SLAs, it is necessary that the failure of any given host
  cannot take down more than k VMs in each N+k pool.  More precisely, given
  that those pools are dynamically scaled, it is a requirement that at no time
  is there more than a certain proportion of any pool instantiated on the
  same host.  See Gaps below.

  That by itself is insufficient for offering an SLA, though: to be deployable
  in a single OpenStack cloud (even spread across availability zones or
  regions), the underlying cloud platform must be at least as reliable as the
  SLA demands.

* Elastic scaling

  An NFV orchestrator must be able to rapidly launch or terminate new
  instances in response to applied load and service responsiveness.  This is
  basic OpenStack nova function.

* Placement zones

  In the IMS architecture there is a separation between access and core
  networks, with the P-CSCF component (Bono - see architecture link below)
  bridging the gap between the two.  Although Project Clearwater does not yet
  support this, it would in future be desirable to support Bono being deployed
  in a DMZ-like placement zone, separate from the rest of the service in the
  main MZ.

Related Use Cases
=================

* https://review.openstack.org/#/c/163399/ (placement zones)

Gaps
====

The above requirements currently suffer from these gaps.

* Affinity for N+k pools

  Affinity/anti-affinity can be expressed pair-wise between VMs, which is
  sufficient for a 1:1 active/passive architecture, but an N+k pool needs
  something more subtle.  Specifying that all members of the pool should live
  on distinct hosts is clearly wasteful. Instead, availability modelling shows
  that the overall availability of an N+k pool is determined by the time to
  detect and spin up new instances, the time between failures, and the
  proportion of the overall pool that fails simultaneously. The OpenStack
  scheduler needs to provide some way to control the latter by limiting
  the proportion of a group of related VMs that are scheduled on the same host.

Affected By
===========

None.

References
==========

* [1] http://www.projectclearwater.org
* [2] https://en.wikipedia.org/wiki/IP_Multimedia_Subsystem
* [3] https://wiki.openstack.org/wiki/TelcoWorkingGroup/UseCases#Virtual_IMS_Core
* [4] http://www.projectclearwater.org/technical/clearwater-architecture/
