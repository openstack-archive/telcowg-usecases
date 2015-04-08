..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

=============================
BGPVPN attachment
=============================

This use case is about how to attach Service Provider IPVPNs to existing
tenant networks.

Glossary
========

**BGPVPN** - [1]

Problem description
===================

Currently, when a tenant wants to isolate its traffic outside the cloud, the
only feature he can use in Neutron is IPSec tunneling. IPsec is fine for
interconnecting sites or remote workers under the responsibility of an
openstack tenant admin. The tenant admin is in charge of the VPN
administration, which is provided by IPSec technologies.

However, Telco Service Providers rely on other technologies to interconnect
sites. A widely used Technologie is BGPVPN.

When a Telco Service Provider operates an Openstack Cloud, its tenants
might also be some BGPVPN customers.
From a Telco perspective, Openstack needs to provide some APIs for
those tenants to attach their Openstack networks to their BGPVPNs, so that
tenant services running in the openstack cloud can be reachable from
any sites of the tenant's BGPVPN.

Examples
--------

1. As stated before, the main use case is to provide tenants the capacity
   to interconnect their Telco Openstack networks to their Telco BGPVPN
2. Another use case would be to interconnect tenant networks of two Openstack
   Cloud throw an BGPVPN

Affected By
-----------

https://review.openstack.org/#/c/93329/
https://review.openstack.org/#/c/136555/
https://review.openstack.org/#/c/136929/

Requirements
============

The implementation can rely on Neutron service extension framework.
Two stackforge projects attempt to fill the gap :
http://git.openstack.org/cgit/stackforge/networking-bgpvpn
http://git.openstack.org/cgit/stackforge/networking-edge-vpn


Related Use Cases
=================

Gaps
====

References
==========

* [1]: RFC4364 BGP/MPLS IP Virtual Private Networks (VPNs) http://tools.ietf.org/html/rfc4364
