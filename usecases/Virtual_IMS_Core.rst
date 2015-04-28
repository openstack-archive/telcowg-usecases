..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

=============================
 Virtual IMS Core
=============================

This use case is about deploying a virtual IMS core as an NFV function in
OpenStack, though is more broadly representative of the issues involved in
deploying in a scalable Telco-grade control plane function deployed as a
series of load-balanced N+k pools.

The use case focuses on those elements rather than more generic issues like
supporting NFV orchestration and cloud platform HA.

This use case obsoletes the version previously hosted direct on the TelcoWG
wiki [1].

Glossary
========

* NFV - Networks Functions Virtualisation, see http://www.etsi.org/technologies-clusters/technologies/nfv
* IMS - IP Multimedia Subsystem
* SIP - Session Initiation Protocol
* P/I/S-CSCF - Proxy/Interrogating/Serving Call Session Control Function
* BGCF - Breakout Gateway Control Function
* HSS - Home Subscriber Server
* WebRTC - Web Real-Time-Collaboration

Problem description
===================

An IMS core [2] is a key element of Telco infrastructure, handling VoIP device
registration and call routing.  Specifically, it provides SIP-based call
control for voice and video as well as SIP based messaging apps.

An IMS core is mainly a compute application with modest demands on
storage and network - it provides the control plane, not the media plane
(packets typically travel point-to-point between the clients) so does not
require high packet throughput rates and is reasonably resilient to jitter and
latency.

As a core Telco service, the IMS core must be deployable as an HA service
capable of meeting strict Service Level Agreements (SLA) with users.  Here
HA refers to the availability of the service for completing new call
attempts, not for continuity of existing calls.  As a control plane rather
than media plane service the user experience of an IMS core failure is
typically that audio continues uninterrupted but any actions requiring
signalling (e.g.  conferencing in a 3rd party) fail.  However, it is not
unusual for client to send periodic SIP "keep-alive" SIP pings during a
call, and if the IMS core is not able to handle them the client may tear
down the call.

An IMS core must be highly scalable, and as an NFV function it will be
elastically scaled by an NFV orchestrator running on top of OpenStack.
The requirements that such an orchestrator places on OpenStack are not
addressed in this use case.

User Stories
------------

As a Telco, I want to deploy an IMS core as a Virtual Network Function
running in OpenStack.

Examples
--------

Project Clearwater [3] is an open-source implementation of an IMS core
designed to run in the cloud and be massively scalable.  It provides
P/I/S-CSCF functions together with a BGCF and an HSS cache, and includes a
WebRTC gateway providing interworking between WebRTC & SIP clients.

Affected By
-----------

None.

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
  SLA demands.  Those requirements will be addressed in a separate use case.

* Elastic scaling

  An NFV orchestrator must be able to rapidly launch or terminate new
  instances in response to applied load and service responsiveness.  This is
  basic OpenStack nova function.

* Placement zones

  In the IMS architecture there is a separation between access and core
  networks, with the P-CSCF component (Bono - see [4]) bridging the gap
  between the two.  Although Project Clearwater does not yet support this,
  it would in future be desirable to support Bono being deployed in a
  DMZ-like placement zone, separate from the rest of the service in the main
  MZ.

Related Use Cases
=================

* https://review.openstack.org/#/c/163399/ (placement zones)

Gaps
====

The above requirements currently suffer from these gaps.

* Affinity for N+k pools

  An N+k pool is a pool of identical, stateless servers, any of which can
  handle requests for any user.  N is the number required purely for
  capacity; k is the additional number required for redundancy.  k is
  typically greater than 1 to allow for multiple failures.  During normal
  operation N+k servers should be running.

  Affinity/anti-affinity can be expressed pair-wise between VMs, which is
  sufficient for a 1:1 active/passive architecture, but an N+k pool needs
  something more subtle.  Specifying that all members of the pool should live
  on distinct hosts is clearly wasteful. Instead, availability modelling shows
  that the overall availability of an N+k pool is determined by the time to
  detect and spin up new instances, the time between failures, and the
  proportion of the overall pool that fails simultaneously. The OpenStack
  scheduler needs to provide some way to control the last of these by limiting
  the proportion of a group of related VMs that are scheduled on the same host.

References
==========

* [1] https://wiki.openstack.org/wiki/TelcoWorkingGroup/UseCases#Virtual_IMS_Core
* [2] https://en.wikipedia.org/wiki/IP_Multimedia_Subsystem
* [3] http://www.projectclearwater.org
* [4] http://www.projectclearwater.org/technical/clearwater-architecture/
