..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

===================================
 Virtual Network Function Bring-Up
===================================

The intended of this usecase is to align VNF vendors and NFV users on general
understanding and best-practise how a VNF gets brought-up (early initialization)
inside a NFV OpenStack setup.


This use-case might act just as a OpenStack best-practise document, without
further need of modification or blueprints for OpenStack itself.


Glossary
========

**VNF**
  Virutal Network Function

**VNF**
  Physical Network Function

**NFV**
  Network Function Virtualization

**cloud-init**
  Defacto standard Open-Source implementation for hypervisor and IaaS agnostic
  cloud instance bring-up [CLOUD-INIT].

Problem description
===================

Early adaptors/PoCs of NFV with OpenStack run in problems when attempting to
operate a multi-vendor VNF setup inside the same or across multiple OpenStack
tenants. The problems are that VNF from vendor A and vendor B behaves during
instantiation different when they come as an appliance/container/image.

The OpenStack Image Guide gives some requirements on Linux images [IMAGE-GUIDE],
which might be extended for the VNF images to align vendors how VNFs get build
for NFV OpenStack setups.

A main differentiation between regular images and VNF images is the initial
network configuration of the image. Regular images are build to bring-up the
first available network interface (e.g. "eth0") and run a DHCP client on that
interface. Later tools like cloud-init try various why to obtain and process
the provided user data and other metadata.

This might not be ideal for NFV setups due to:

 - Can we guarantee that the first interface is always acting as control-plane?
   Where is this defined? Is this hypervisor agnostic?
   Risk: if the first interface unexpected turns to be a customer-facing
   dataplane interface, the VNF might run DHCP request to an untrusted
   network.

 - DHCP response might include a default-route, which needs to be laborious
   removed during the further bring-up process in some setups. (Router
   configuration where the default-route should be later set on a different
   dataplane interface, no on the first "control-plane" interface

 - Do we obtain from the user data and other meta-data provided from
   OpenStack today enough details for complex network configuration?
   (e.g. is there need to "tag" interfaces as control-plane interface?)

 - On which information should a VNF vendor enumerate their dataplane
   and control-plane interface? To have deterministic interface naming
   inside the guest. eth0, eth1, ethN might not be good enough for VNFs
   setups where you have a mix of virtio-backed interfaces and PCI passthrough
   interfaces. And some are driven by DPDK-based dataplanes and some
   driven by the stock OS network stack.



User Stories
------------

* As an NFV cloud operator, I want to launch a VNF with a minimal configuration provided
  by the user-data and enable the VNF to get configured by an external SDN controller, dedicated
  to VNF/PNF configuration.

* As an NFV cloud operator, I want to describe my multi-vendor VNF topology in Heat templates
  which is able to auto-scales VNFs on provided KPIs.

* ...

Examples
--------

None.

Affected By
-----------

None.

Requirements
============

Use this section to define the functions that must be available or any
specific technical requirements that exist in order to successfully
support your use case. If there are requirements that are external
to OpenStack, note them as such. Please always add a comprehensible
description to ensure that people understand your need.

* All VNF images for OpenStack should hold cloud-init

* VNF vendors should enable their image to be configurable
  via common user-data parameters (enhance their version of cloud-init
  to support their VNF)
* Vendor specific configuration should/could be implemented as cloud-init
  vendor-specific part-handler

* No assumption on an pre-existing network-addressing on the tenant network
  should be made. All required meta-data about the initial/bring-up network
  configuration should be obtained from OpenStack and cloud-init.

* Initial bring-up configuration handling should be hypervisor agnostic
  (by using cloud-init).

Related Use Cases
=================

cloud-init is also interested in obtain more meta-data from OpenStack about
detailed/complex network configuration usecases.

Gaps
====

 * OpenStack VNF image requirments document is missing
 * ...

References
==========

* [CLOUD-INIT]: https://github.com/stackforge/cloud-init
* [IMAGE-GUIDE]: http://docs.openstack.org/image-guide/content/ch_openstack_images.html

