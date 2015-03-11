..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

..
  This template should be in ReSTructured text. Please do not delete any
  of the sections in this template. If you have nothing to say for a
  whole section, just write: None.
  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

======================================
Security Segregation (Placement Zones)
======================================

The goal of this use-case it to present the need for a (partly) segregation
of physical resources to support the well-known classic separation of DMZ
and MZ, which is still needed by several applications (VNFs) and requested
by Telco security rules.

Glossary
========

AZ = Availability Zone
EHD = Exposed Host Domain
DMZ = Demilitarized Zone
MZ = Militarized Zone
SEC = Secure Network Zone


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
breach. This use-case affects all of the main OpenStack modules.

Current Situation
-----------------
Today the DMZ and MZ concept is an essential part of the security design
of nearly every Telco application deployment. Today this separation is
done on by a consequent physical segregation including hosts, network and
management systems. This separation leads to high investment and
operational costs.

Enable the following
--------------------
We would like to use Placement Zones to ensure that only VMs following the
same security classification will run on the same group of physical hosts.
This should avoid a mix of VMs from different zones (e.g. DMZ and MZ),
coming up with different security requirements, running on the same group
of hosts. Therefore a host (respectively multiple hosts) must be classified
and assigned to only one placement zone. During the deployment process of a
VM it must be possible to assign it to one placement zone (or use the
default one), which automatically leads to a grouping of VMs. The security
separation within the network can be done on a logical layer using the same
physical elements but adding segregation through VLANs, virtual firewalls
and other overlay techniques.

Examples
--------

An application presentation layer (e.g. webserver) must be decoupled from
the systems containing sensitive data (e.g. database) through at least one
security enforcement device (e.g. virtual firewall) and a separation of
underlying infrastructure (hypervisor).

https://wiki.openstack.org/wiki/File:TelcoWG_Placementzones.png
[logical view ; data center router and switching infrastructure intentionally missing]

Affected By
-----------

unknown

Requirements
============

* One OpenStack installation must be capable to manage different
  placement zones. All resources (compute, network and storage) are
  assigned to one placement zone. By default, all resources are
  assigned to the "default" placement zone of OpenStack
* SEC is a special placement zone - it provides the glue to connect
  the placement zones on the network layer using vNFs. SEC VNFs may
  be attached to resources of other placement zones
* Placement zone usage requires a permission (in SEC, tenants cannot
  start VMs, this zone supports only the deployment of Xaas services
  FWaas, LBaas,...])
* If placement zones are required in a cloud, VMs must be assigned to
  one placement zone
* All resources, which are needed to run a VM must belong to the same
  placement zone
* Physical Hosts (compute nodes) must be able to assigned to only one
  placement zone (re-assigning possible due to changing utilization)
  * Several assignments must be restricted by the API
  * If a host is reassigned it must evacuate all existing VM
* ...and the whole thing must be optional  :-)

References
----------

[1]: http://docs.openstack.org/openstack-ops/content/scaling.html
[2]: https://wiki.openstack.org/wiki/Congress

Related Use Cases
=================

none

Gaps
====

Nova issues
* Usage of availability zone(AZ)/host aggregates to assign a vm to a
  sec (placement??)  zone [Ref. 1]
   * By default a host can be assigned to multiple availability zones
   * It's up to the operator to ensure security
   * Maybe Congress [Ref. 2] (Policy as a Service) could be a solution?
* In case of a reassignment the running VM (potential from a different
  availability zone) remain on the host

Neutron issues:
* AZ's are not known to Neutron services
  * It's up to the operator to ensure that the right networks are attached

Cinder/Manila/Storage issues:
* Storage can be segregated with volume-types.
* AZ's are not known to the storage services
  * Must be ensured from the deployment tool that the right storage is
    accessible

OpenStack regions provide a segregation of all resources. They cloud be used
to implement placement zones, BUT:
* Complex and resource consuming installation for the Openstack management
  systems
* Tenants must deal with additional regions
* No L2 network sharing for VMs in the SEC placement zone required to glue the
  zones together
* No real enforcement
* Complex operations
