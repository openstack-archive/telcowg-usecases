..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

..

================================
SFC -- Service Function Chaining
================================

The goal of is to add support for service function chaining to
OpenStack Neutron. Right now this feature is mainly realized by
proprietary solutions which come in form of a SDN controller plus a
Neutron plug-in. We would like to support SFC as shown in [1], [2] 
and [3] in OpenStack Neutron to provide support of service
function chaining (SFC) directly using Neutron SFC API. If the SFC
model from refs [1]-[3] is missing "required" features, these features
should be implemented.

Glossary
========

**NFV**
  Network Function Virtualization

The definitions below have been taken from section 1.3 in [1].

**Service Function Chain (SFC)**
  A service function chain defines an
  ordered set of abstract service functions (SFs) and ordering
  constraints that must be applied to packets and/or frames and/or
  flows selected as a result of classification.

**Service**
  Beyond this document, the term "service" is overloaded
  with varying definitions.  For example, to some a service is an
  offering composed of several elements within the operator's
  network, whereas for others a service, or more specifically a
  network service, is a discrete element such as a "firewall".
  Traditionally, such services (in the latter sense) host a set of
  service functions and have a network locator where the service
  is hosted.

**Classification**
  Locally instantiated matching of traffic flows
  against policy for subsequent application of the required set of
  network service functions.  The policy may be customer/network/
  service specific.

**Classifier**
  An element that performs classification.

**Service Function (SF)**
  A function that is responsible for specific
  treatment of received packets.  A Service Function can act on
  various layers of a protocol stack (e.g., at the network layer
  or other OSI layers).  As a logical component, a Service
  Function can be realized as a virtual element or be embedded in
  a physical network element.  One or more Service Functions can
  be embedded in the same network element.  Multiple occurrences
  of the Service Function can exist in the same administrative
  domain.

Problem description
===================

The first step to provide generic support of SFC in OpenStack is to define a
common API. To our best knowledge the most efficient way is to implement this
as Neutron API (extension). This API must not mandate the technology used to
provide a service, function or chain (e.g. a specific "SDN-controller").
The goal of this specification is to provide the user with a tool that captures
their high level intent without getting coupled with implementation details of
the specific deployment. This provides a path for migrating such service function
services across technology changes or for hybrid deployments utilizing various
technologies. Moreover, such API provide a common base on which vendors can innovate.
In order to support different deployments, plugins and drivers must be used to provide
the necessary abstraction from the API layer. It might be necessary to
add parameters to support implementation specific features.
The same idea can be found in L2 networks within OpenStack. Depending on the
used L2 driver, specific parameters are necessary to configure the plugin or driver.

Reference [3] provides an overview of the issues associated with the
deployment of service functions in (large) data centers.

If a SFC requires an static or dynamic classifier additional features
are necessary. These features may depend on the driver layer used to
implement SFC.

Examples
--------

A typical SFC consists of a multiple function (such as firewall,
load balancer, router, etc) connected in a particular chain. A high
flexibility in terms of configuration and multi-tenancy is needed in
order to be able to realize complex requirements. An example of such
service can be Gi-Lan described, for instance in [18]. Traffic must go
through a specific order of functions. Moreover, it must be possible
to provide multiple paths through the NFV depending on the customer
by using classifiers.

**1 from [1]**
  An example of an
  abstract service function is "a firewall".  The implied order
  may not be a linear progression as the architecture allows for
  SFCs that copy to more than one branch, and also allows for
  cases where there is flexibility in the order in which service
  functions need to be applied.

**2 from [1]**
  One or more Service Functions can be involved in the delivery of
  added-value services.  A non-exhaustive list of abstract Service
  Functions includes: firewalls, WAN and application acceleration,
  Deep Packet Inspection (DPI), LI (Lawful Intercept), server load
  balancing, NAT44 [RFC3022], NAT64 [RFC6146], NPTv6 [RFC6296],
  HOST_ID injection, HTTP Header Enrichment functions, TCP optimizer.

**3 Openstack**
  In OpenStack the network interfaces of VMs are attached to Ethernet
  based L2 transport. All other functions used to interconnect VMs,
  to connect networks to other networks or the Internet using e.g. a
  L3 router or to provide advanced services, e.g. LBaas or VPNaaS may
  be implemented by using (predefined) SFCs. Such functions
  should be implemented by defining SFCs instead of programming code
  for these specific use cases.
  SFCs may use classifiers to decide, which traffic or flows should
  use which SFC.

The ETSI document [17] shows additional use cases.

Affected By
-----------

The references [4] to [13] contain previous work done for SFC in
OpenStack. But this work was abandoned, not continued or not finished
for many (unknown) reasons.
The references [1] to [3] are describing the IETF work for SFC.

Requirements
============

* We need direct support of SFC via Neutron-API (comparable, for example,
  to today’s support of LBaaS). This would enable provisioning of multiple
  SFCs running on COTS/x86 hardware in virtualized data center and would
  enable Telco providers to orchestrate provisioning of telco services
  (“NFVs”) using a standardized Neutron-API in a multi data center
  OpenStack environment.

* Define a common REST API in Neutron for service function chaining on the
  northbound interface: OpenStack should provide programmable API to configure
  and deploy standardized SC connectivity, for that Neutron API must be
  expanded to include a set of standardized commands which will allow creation
  of SFC. At the end, a tenant (or customer) does not care about the
  infrastructure, where a service chain is implemented. This might be an
  Openstack based cloud or a classic network without any Openstack dependency
  using an SDN Controller and physical network components.
  
* SFC available in Horizon in Network (comparable to Load Balancers, FW, etc)

* The implementation of SFC should be done by plugins and drivers to allow
  the usage of different implementation models (use a similar model for SFC
  as for ML2 using plugins, type and mechanism drivers)

* We need to have an OpenSource reference implementation. OVS may be used.

SFC hat two flavours, both flavours must be supported

* the fixed service chaining. The endpoints of two VNFCs
  are using interfaces or vlan tags on interfaces. A virtual link is
  created between these endpoints by the SDN controller. This is always
  a peer to peer connection of two systems per link. Such a model provides
  only a predefined chaining. Classic data center services may be implemented
  using this chaining model (firewall, load balancer, proxy-server, firewall,
  web server, database)
* the flexible implementation. This flavour of service chaining  assumes,
  that the sending endpoint has the choice to sent packets to multiple
  destinations. The ordering of the chain is defined by the first hop of
  the service chain and might be modified without notifying the SDN
  controller. An implementation of the flexible service chaining may
  use NSH or stacked MPLS labels. Network functions which make
  decisions based on  the sender IP address or the payload may be
  implemented by using this chaining model. The flexible model
  decouples any decision how to forward packets in a tenant's
  application (service chain) from the network controller of the
  infrastructure provider.  The SDN controller has only the function
  to provide a transport network and the ingress/egress points for
  the service chain.

Both flavours are needed. The fixed flavour is implemented by a few
vendors by using a vendor specific API. The flexible model is partly
implemented by only one vendor, using a vendor specific API.

SFC should support the transparent insertion of other SFCs (stacking of SFCs)
in an existing SFC. E.g. there is a running SFC from a tenant and
the infrastructure provider is forced to insert an IDS/DDOS
prevention/firewall function to mitigate threads.

Implementation Ideas
====================

The first goal is to provide the necessary API in neutron for static and
dynamic (e.g. NSH) SFC

A reference implementation in Neutron may use

* a plugin/driver using ML2/OVS for static SFC
* a plugin/driver to forward the SFC definitions and actions to a SDN
  controller (ODL for reference?)

Service Function Chain (SFC):  A service function chain defines a set of
service functions (realized as VMs in OpenStack or by a module on a hypervisor)
and ordering constraints that must be applied to packets (in case of L3
service) and/or frames (in case of L2 service) selected as a result of
classification.

The Main SFC data plane components include:

* traffic classifier
* service path
* service overlay
* context

SFC consists out of SFC data, control and management planes. In [1] the
name “policy“ was user for management plane.

SFC data plane components:

* Service classifier (SC)
* Service path (SP)
* Service overlay (SO)
* Service function (SF)
* Service function chain (SFC)
* Service function forwarder (SFF)
* Service function proxy/gateway (SFG)
* Context

Service path/plane can be created with the help of Network Service
Header (NSH) [19]. Traffic in SFC may need to traverse SF more than
one time (“cycle”). SFs in the chain must be able to attend metadata
to data packets to transport classification information from one VM
to the other.

Assumption here: SC is built in OVS, e.g. SFF is built in OVS.
An alternative would be to run SC as a separate VM
 
The main data plane modes are: 

* Transparent/bridged (example: L2 Firewall, IDS, IPS)
* In-network (example: NAT, L3 Firewall, LB)
* In-network NAT (example: NAT services)
* One-arm (example: mirroring, LB)

* SFC data plane components include traffic classifier, service path,
  service overlay and context
* SFC consists out of SFC data, control and management plane
* SFC data plane components include: Service classifier (SC), Service
  path (SP), Service function (SF),  Service function chain (SFC),
  Service function forwarder (SFF), Service function proxy/gateway
  (SFG), Context
* SP is created with the help of Network Service Header (NSH) [19]
* Traffic in SFC may need to traverse SF more than one time (“cycle”).
  SFs in the chain must be able to attend metadata to data packets to
  transport classification information from one VM to the other
* SFs in the chain must be able to attend metadata to data packets to
  transport classification information from one VM to the other.

API design ideas
----------------

Neutron API should support following

* create, modify, delete a chain (SFC/VNF)
* create, modify, delete a function (SF/VNFC)
* create, modify, delete a Service function path element (SFPE)
* create, modify, delete a path (SP)
* assign, deassign a service path element (SPE) to a path (SP)
* assign, deassign SFPEs to path elements
* assign, deassign a path (SP) to SFC
* assign, deassign a SFPEs to path elements

* create, start, stop, delete a Service chain instance (SFCI)
* list and show commands must be supported for all commands

* a sfci verify command must be supported for SFCIs.
  This command MUST check if the SFC has been deployed as expected.

* a sfci statistics command must be supported for SFCIs.
  This command MUST report all relevant statistics -- maybe by
  polling ceilometer data when SFC is used in Openstack

The API design must be discussed in detail.

Example with a simple firewall - no dynamics (e.g. NSH) and
no classifiers. For simple services this looks quite complex...

* create the SFC::

      neutron sfc-create sfc-fw  static


* create the SF with two ports (interfaces) to be used by SPEs::

      neutron sf-create  sf-fw 2

* assign an image to boot to create the SF using a VM::

      neutron sf-update image <id of sf-fw> <glance image id>

  here it should also be possible to use neutron ports to
  attach anything - even real devices.

* assign the SF to the SFC::

      neutron sf-assign <id of sf-fw> <id of sfc-fw>

* create a service path::

      neutron sp-create sp-fw

* assign the SP to the SFC::

      neutron sp-assign <id of sp-fw> <id of sfc-fw>

* create two service path elements::

      neutron spe-create spe1
      neutron spe-create spe2

* assign one side (Left or Right) of each spe to the service function::

      neutron spe-assign <id of spe1> <id of sp-fw> R
      neutron spe-assign <id of spe2> <id of sp-fw> L

* assign the other side, Left or Right, of each spe to the outside world
  using a name and the key --external::

      neutron spe-assign --external <id of spe1> <id of sp-fw> L ingress
      neutron spe-assign --external <id of spe2> <id of sp-fw> R egress

* Now the SFC is defined in the inventory and can be instantiated, even multiple times,
  using a neutron command::

      neutron sfci-create sfci-fw1 <id of sfc-fw>

  the SPEs marked as external are assigned to neutron ports

* Assign the external ports of the SFCI to the real world::

      neutron sfci-portlist <id of sfci-fw1>
      neutron sfci-assign <id of sfci-fw1> <ingress> <netid to attach the ingress port>
      neutron sfci-assign <id of sfci-fw1> <egress>  <netid to attach the egress port>

* And finally: start the SFC instance::

      neutron sfci-start <id of sfc-fw>

  This might be optional for non VM based SFs.

Advanced topics

* create, modify, remove a classifier (SC) from/to a path element (SPE)
* change the order of SPEs and SF in the SP

Current state in OpenStack
--------------------------

Nova issues:

* Nova is not able to schedule SFCs

Neutron issues:

* SFC is not known to Neutron - A really weak point is, that there
  is no API in Openstack available to decouple the description of
  service chaining from the implementation in the infrastructure
* Neutron sees only separate networks
* No notion of SFC is defined

Related Use Cases
=================

Reference [2] shows use cases for data centers

Gaps
====

None currently known

Comments
========

A comment from unknown on the original Etherpad document:
Are folks aware there is going to be an overall proposal by a group of
vendors/carriers on simplifying neutron, use heat templates to define
network templates which then get fed through neutron to SDN controllers
(e.g. ODL, ONOS...) and let the SDN controllers do the heavy lifting
of network changes. I'm concerned this is not going in the same direction.


References
==========

* [1]: https://datatracker.ietf.org/doc/draft-ietf-sfc-architecture/
* [2]: https://datatracker.ietf.org/doc/draft-ietf-sfc-dc-use-cases/
* [3]: https://datatracker.ietf.org/doc/draft-ietf-sfc-problem-statement/
* [4]: https://review.openstack.org/#/c/93524/13/specs/juno/service-chaining.rst
* [5]: https://review.openstack.org/#/c/93128/22/specs/juno/service-base-and-insertion.rst
* [6]: https://review.openstack.org/#/c/92477/7/specs/juno/traffic-steering.rst
* [7]: https://review.openstack.org/#/c/92200/5/specs/juno/advanced-services-common-framework.rst
* [8]: https://blueprints.launchpad.net/neutron/+spec/neutron-services-insertion-chaining-steering
* [9]: https://docs.google.com/file/d/0B05WnTIhCwXhUV94a3VXbDN3OUU/edit?usp=sharing
* [10]: https://wiki.openstack.org/wiki/Neutron/ServiceInsertion
* [11]: https://docs.google.com/document/d/1fmCWpCxAN4g5txmCJVmBDt02GYew2kvyRsh0Wl3YF2U
* [12]: https://wiki.openstack.org/wiki/Neutron/ServiceInsertionAndChaining/API
* [13]: https://review.openstack.org/#/c/117676/
* [14]: https://etherpad.opnfv.org/p/Network_Function_Chaining
* [15]: https://etherpad.openstack.org/p/kKIqu2ipN6
* [16]: https://wiki.opendaylight.org/view/Service_Function_Chaining:Main#SFC_TOD
* [17]: http://nfvwiki.etsi.org/images/PoC_proposal_Scalable_Service_Chaining-revisedv3%28final%29.pdf
* [18]: http://www.ietf.org/proceedings/88/slides/slides-88-sfc-4.pdf
* [19]: https://datatracker.ietf.org/doc/draft-quinn-sfc-nsh/
* [20]: https://wiki.opnfv.org/requirements_projects/openstack_based_vnf_forwarding_graph
