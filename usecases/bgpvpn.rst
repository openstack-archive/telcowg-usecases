..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

=============================
BGP based IP VPNs attachment
=============================

This use case is about how to attach Service Provider BGP-based IP VPNs
to existing tenant networks.

Glossary
========

**BGP-based IP VPNs** - [1] : VPNs provided by a service provider on top
of its IP backbone

Problem description
===================

Currently, when a tenant wants to isolate its traffic outside the cloud, the
only feature he can use in Neutron is IPSec tunneling. IPsec is fine for
interconnecting sites or remote workers under the responsibility of an
openstack tenant admin. The tenant admin is in charge of the VPN
administration, which is provided by IPSec technologies.

However, Telco Service Providers rely on other technologies to interconnect
sites. A widely used technology is BGP based IP VPNs, which uses MPLS to
isolate trafic.
These BGP/MPLS VPNs allow to connect multiple sites together and ensure isolation
of traffic between the differents VPNs, which can use overlapping address spaces.
They do not typically provide means to encrypt the traffic, but are widely used
nonetheless by enterprises to interconnect their sites: customers routers are not
exposed to the wide Internet or other customers, and traffic
authentication/secrecy can be applied at the application level on a
per-application basis.

When a Telco Service Provider operates an Openstack Cloud, its tenants
might also be some BGP-based IP VPN customers.
It is essential to provide IP-VPN BGP support in Openstack to enable
interoperability with existing technologies as also easy migration towards
Openstack cloud.

Telcos need a way to give the capacity to their tenants to attach their Openstack
networks to their BGP based IP VPNs, so that tenant's services running in the
openstack cloud can be reachable from any sites of the tenant's VPN.

Examples
--------

1. As stated before, the main use case is to provide tenants the capacity
   to interconnect their Telco Openstack networks to their Telco BGP based IP VPN.
2. Another use case would be to interconnect tenant's networks of two Openstack
   Cloud through a BGP based IP VPN.

Affected By
-----------

BGPVPN attachment APIs and service framework : https://review.openstack.org/#/c/93329/
Programmable cloud edge networks : https://review.openstack.org/#/c/136555/
Define APIs to provision an Edge VPN service : https://review.openstack.org/#/c/136929/
L2Gateway : https://wiki.openstack.org/wiki/Meetings/L2Gateway

Requirements
============

The implementation can rely on Neutron service extension framework.
Two stackforge projects attempt to fill the gap :
http://git.openstack.org/cgit/stackforge/networking-bgpvpn
http://git.openstack.org/cgit/stackforge/networking-edge-vpn


Related Use Cases
=================

Ethernet VPNs :  E-VPN [2] has been specified to support Ethernet VPNs with an
architecture very similar to BGP/MPLS IP VPNs. Although the technology is more recent,
this use case applies to E-VPN as well, in particular in the perspective of providing
Ethernet connectivity between Openstack neutron networks and customer sites,
or inter-connectivity between multiple DCs.

Gaps
====

References
==========

* [1]: RFC4364 BGP/MPLS IP Virtual Private Networks (VPNs) http://tools.ietf.org/html/rfc4364
* [2]: RFC7432 BGP MPLS-Based Ethernet VPN https://tools.ietf.org/html/rfc7432
