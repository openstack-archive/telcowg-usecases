..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

..

======================================
Security Segregation (Placement Zones)
======================================

The goal of this use-case it to present the need for a (partly) segregation
of physical resources to support the well-known classic separation of DMZ
and MZ, which is still needed by several applications (VNFs) and requested
by Telco security rules.

Glossary
========

**AZ**
  Availability Zone (OpenStack terminology)

**DMZ**
  Demilitarized Zone provides access to the public network,
  but adds an additional security layer (e.g. firewall). Designed for security
  critical customer facing services (e.g. customer control center).

**EHD**
  Exposed Host Domain provides direct access from the public network.
  Designed for services which require a high traffic volume (e.g. CDN) and are
  not security critical.

**MZ**
  Militarized Zone a logical network without any access from public network.
  Designed for systems without direct customer connectivity (e.g. Databases
  containing sensitive data) and high security demand.

**PZ**
  Placement Zone is a concept to classify different securiy areas based on
  different security requirements. PZ are separated on a per host basis.

**SEC**
  Secure Network Zone for all devices providing a security function including
  devices providing connectivity between Placement Zones (e.g. virtual firewall
  for DMZ-MZ traffic).

**VNF**
  Virtual Network Function an implementation of an functional building block
  within a network infrastructure that can be deployed on a virtualization infrastructure or rather an OpenStack based cloud platform (a non virtualized
  network function is today often a phsical applicance).

Problem description
===================

The main driver therefore is that a vulnerability
of a single system must not affect further critical system or endanger
exposure of sensitive data. On the one side the benefits of virtualization
and automation techniques are mandatory for Telcos but on the other side
telecommunication data and involved systems must be protected by the
highest level of security and comply to local regulation laws (which are
often more strict in  comparison with enterprise).
Placement Zones should act as multiple lines of defense against a security
breach. f a security breach happens in a placement zone, all other
placement zones must not be affected. This must be ensured by the design.
This use-case affects all of the main OpenStack modules.

Current Situation
-----------------
Today the DMZ and MZ concept is an essential part of the security design
of nearly every Telco application deployment. Today this separation is
done on by a consequent physical segregation including hosts, network and
management systems. This separation leads to high investment and
operational costs.

Enable the following
--------------------
We want to use Placement Zones to reduce the risk, that the whole cloud platform
is affected by a security breach.

The security separation within the network can be done on a logical layer
using the same physical elements but adding segregation through VLANs,
virtual firewalls and other overlay techniques.

On the hypervisor itself, a security breach is more likely. To reduce this risk,
a physical seperation of VMs running on the same hypervisor is necessary.
We would like to use Placement Zones to ensure that only VMs following the
same security classification will run on the same group of physical hosts.
This should avoid a mix of VMs from different zones (e.g. DMZ and MZ),
coming up with different security requirements, running on the same group
of hosts. Therefore a host (respectively multiple hosts) must be classified
and assigned to only one placement zone. During the deployment process of a
VM it must be possible to assign it to one placement zone (or use the
default one), which automatically leads to a grouping of VMs. 

Examples
--------

An application presentation layer (e.g. webserver) must be decoupled from
the systems containing sensitive data (e.g. database) through at least one
security enforcement device (e.g. virtual firewall) and a separation of
underlying infrastructure (hypervisor).

https://wiki.openstack.org/wiki/File:TelcoWG_Placementzones.png


Affected By
-----------

unknown

Requirements
============

* One OpenStack installation must be capable to manage different placement
  zones. All resources (compute, network and storage) are assigned to one
  placement zone. By default, all resources are assigned to the "default"
  placement zone of OpenStack
* It must be possible to configure the allowed communication between
  placement zones on the network layer
* SEC is a special placement zone - it provides the glue to connect the
  placement zones on the network layer using vNFs. SEC VNFs may be attached to
  resources of other placement zones
* Placement zone usage requires a permission (in SEC, tenants cannot start VMs,
  this zone supports only the deployment of Xaas services FWaas, LBaas,...])
* If placement zones are required in a cloud, VMs must be assigned to one
  placement zone
* All resources, which are needed to run a VM must belong to the same placement
  zone
* Physical Hosts (compute nodes) must be able to assigned to only one placement
  zone and re-assigning should possible due to changing utilization

  * Several assignments must be restricted by the API
  * If a host is reassigned it must evacuate all existing VM

* ...and the whole thing must be optional  :-)

Related Use Cases
=================

none

Gaps
====

**Nova issues:**

* Usage of availability zone(AZ)/host aggregates to assign a vm to a placement
  zone is feasable [Ref.1], but:

  * By default a physical host can be assigned to multiple host aggregates
  * It is up to the operator to ensure security using non OpenStack mechanisms
  * Maybe Congress [Ref. 2] (Policy as a Service) could be a solution?


**Neutron issues:**

* AZs or PZs are not known to Neutron services

  * It's up to the operator to ensure that the right networks are attached to VMs

**Cinder/Manila/Storage issues:**

* Storage can be segregated with volume-types
* AZs are not known to the storage services

  * Must be ensured from the deployment tool that the right storage is accessible

**OpenStack regions** provide a segregation of all resources. The region concept
can be used to implement placement zones, but:

* Complex and resource consuming installation for the Openstack management
  systems
* Tenants must deal with additional regions
* No L2 network sharing for VMs in the SEC placement zone required to glue the
  zones together
* No real enforcement
* Complex operations

References
==========

* [1]: http://docs.openstack.org/openstack-ops/content/scaling.html
* [2]: https://wiki.openstack.org/wiki/Congress