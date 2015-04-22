..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

===========================
 Session Border Controller
===========================

This use case is specifically about deploying the Perimeta Session Border
Controller (SBC) Virtual Network Function (VNF) from Metaswitch Networks in
OpenStack, but is more generally representative of the issues involved in
deploying in OpenStack any VNF utilising a fast data plane or high scale SIP.
The use case focuses on those elements rather than more generic issues like
orchestration and HA.

Perimeta, like other SBCs, sits on the edge of a service provider's network and
polices SIP and RTP (i.e. VoIP) control and media traffic passing over both
* the access network between end-users and the core network
* the trunk network between the core and another service provider.

Glossary
========

**NFV**
  Network Functions Virtualization, the vision of deploying telecoms functions
  as virtual applications running on commercial off the shelf hardware.

**VNF**
  Virtual Network Function - a telecoms or other network function running as
  a virtual application.

**SIP**
  Session Initiation Protocol (RFC 3261) - a common application-layer control
  protocol for creating, modifying and destroying sessions between two or more
  participants.

**RTP**
  Real-time Transport Protocol (RFC 3550) - an end-to-end network transport
  protocol for transmitting real-time data like audio and video.

**VoIP**
  Voice over Internet Protocol - delivering voice and multimedia sessions over
  IP networks, commonly through the use of SIP + RTP.

**SBC**
  Session Border Controller, a telecoms function which polices SIP and RTP
  flows, providing security, quality of service, admission control and interop
  services.

**DDoS**
  Distributed Denial of Service - a form of packet flood attack.

**SLA**
  Service Level Agreement - contractual commitment to reach certain performance
  and availability targets.

**SR-IOV**
  Single Root I/O Virtualisation - a technique for presenting a single physical
  PCIe device (such as a NIC) as multiple virtual devices, directly presented
  to VMs.

**DPDK**
  Data Plane Development Kit - a set of libraries and drivers for fast packet
  processing.

Problem description
===================

In order to implement its security and admission control functions (e.g. DDoS
protection), Perimeta must perform line-rate processing of received packets.
For RTP streams, this equates to several million VoIP packets (each ~64-220
bytes depending on codec) per second per core.  Perimeta must be able to
guarantee this performance and offer SLAs.

Perimeta must be fully HA, with no single points of failure, and service
continuity over both software and hardware failures (i.e. all SIP sessions and
RTP sessions must continue with minimal interruption over software or hardware
failures).

Perimeta must be elastically scalable, enabling an NFV orchestrator to add and
remove instances in response to applied load.

Perimeta would ideally be able to distinguish and separate traffic from
different customers via VLANs or similar mechanism.

Perimeta signaling instances must be able to support large numbers of TCP
connections (hundreds of thousands) to cater for large numbers of clients using
TCP.

Perimeta must be able to coexist with VMs which do not have these requirements
on the same host albeit with sufficient dedicated resources.  For example, just
because Perimeta may not require security group function does not mean this can
be disabled at a host scope.


Affected By
-----------

The following bps in progress will affect this use case (see Gaps):
* https://blueprints.launchpad.net/nova/+spec/libvirt-vif-vhost-user
* https://blueprints.launchpad.net/nova/+spec/virt-driver-cpu-pinning
* https://blueprints.launchpad.net/nova/+spec/input-output-based-numa-scheduling
* https://blueprints.launchpad.net/nova/+spec/virt-driver-large-pages
* https://blueprints.launchpad.net/neutron/+spec/ml2-ovs-portsecurity
* https://blueprints.launchpad.net/neutron/+spec/nfv-vlan-trunks
* https://blueprints.launchpad.net/neutron/+spec/ovs-optimization-redundant-bridge

The following etherpads are also relevant:
* https://etherpad.openstack.org/p/kilo_sriov_pci_passthrough
* https://etherpad.openstack.org/p/telcowg-usecase-openstack-ha

Requirements
============

The problem statement above leads to the following requirements.

* Achieving packets per second target - networking implications

  A standard OpenStack/OpenvSwitch platform allows VMs to drive NICs to full
  bandwidth if using large packet sizes typical for Web applications. What
  makes VoIP different is the small packet size, which means order of magnitude
  more packets and permits only a few hundred CPU instructions per packet -
  nowhere near enough to drive a packet through the standard OpenStack
  networking stack from VM to NIC.  Instead, this requires technologies such
  as SR-IOV (https://blueprints.launchpad.net/nova/+spec/pci-passthrough-sriov
  - completed in 2014.2, though with some technical debt remaining) or a DPDK
  or similar poll mode based vSwitch in the host.

  Security groups must be disabled for network technologies where they are
  not bypassed completely.

  The packet path in the host must be optimized to avoid having to pass through
  multiple bridges and switches.

* High scale TCP - networking implications

  For ports with security group function disabled, it is desirable that host
  connection tracking function is disabled to avoid performance and occupancy
  hits for large numbers of TCP connections and the need to tune host
  parameters unnecessarily.

* Achieving packets per second target - compute implications

  * To achieve line rate all the working data for processing RTP streams
    (active flows etc.) must be kept in L3 cache - main memory look-ups are too
    slow. That requires pinning guest vCPUs to host pCPUs.

  * To optimise the data flow rate it is desirable to bind to a NIC on the host
    CPU's bus.

  * To offer performance SLAs rather than simply "best efforts" we need to
    control the placement of cores, minimise TLB misses and get accurate info
    about core topology (threads vs. hyperthreads etc).

* HA

  Perimeta must be deployable to provide a 5 9's level of availability.  If
  deployed in a single cloud instance, that instance must therefore itself be
  more than 5 9's available.  As that is hard to achieve with today's state of
  the art, Perimeta is designed to be able to span multiple independent cloud
  instances, so that the failure of any one cloud has a minor impact.  The
  requirements that creates are still being discussed and will be addressed in
  a future use case.

  When deploying Perimeta within a single cloud instance, Perimeta uses an
  active/standby architecture with an internal heartbeat mechanism allowing the
  standby to take over within seconds of failure of the active, including
  taking over its IP address.  To support these application level HA mechanisms
  requires:

  * support for anti-affinity rules to permit the active and standby being
    instantiated on the same host

  * support for application-controlled virtual IPs via gratuitous ARP based
    scheme (for IPv4) and NDP Neighbour Advertisements (for IPv6); in both
    cases the standby sends messages saying it now owns the virtual IP address.

  The former is supported through standard anti-affinity nova schduler rules,
  and the latter through the neutron allowed-address-pairs extension.

  If using SR-IOV, Perimeta does not need multiple SR-IOV ports, as
  application level redundancy copes with the failure of a single NIC. However,
  it can take advantage of local link redundancy using multiple SR-IOV vNICs.
  For this to be of any benefit requires the SR-IOV VFs forming a redundant
  pair to be allocated on separate PFs.

  Additionally, it is clearly desirable that the underlying cloud instance is
  as available as possible e.g. no SPOFs in the underlying network or storage
  infrastructure.

* Elastic scaling

  An NFV orchestrator must be able to rapidly launch or terminate new Perimeta
  instances in response to applied load and service responsiveness.  This is
  basic OpenStack nova function.

* Support for a scalable mechanism to support multiple networks in a VM

  There must be a scalable mechanism to present multiple networks to Perimeta,
  of order hundreds or thousands, so far exceeding the number of vNICs that can
  be attached.  Various mechanisms are possible; a common one, and the one
  that Perimeta supports, is for different customer networks to be presented
  over VLANs.  This creates a guest requirement for VLAN trunking support.

  There are multiple possible ways of mapping networks to these VLANs within
  OpenStack, for example, trunking external VLAN networks directly to the VMs
  with minimal OpenStack knowledge or configuration (already supported in Kilo)
  or configuring the mapping between OpenStack networks and VLANs as covered in
  VLAN aware VMs: https://blueprints.launchpad.net/neutron/+spec/vlan-aware-vms

Related Use Cases
=================

None as yet, but any VNF that needs to do line rate processing of VoIP packets
will likely have similar requirements and gaps.

Gaps
====

The above requirements currently suffer from these gaps.

* SR-IOV

  Initial support for this has been released, but there remains technical debt
  being tracked at https://etherpad.openstack.org/p/kilo_sriov_pci_passthrough
  which would improve the usability and robustness of an SR-IOV-based solution.

  Making use of multiple bonded SR-IOV VFs requires the following two bps:

  * SR-IOV VFs distributed across multiple pNICs

    * https://blueprints.launchpad.net/nova/+spec/distribute-pci-allocation

  * Allow user to control setting of spoof checking for the SR-IOV ports

    * https://blueprints.launchpad.net/neutron/+spec/sriov-spoofchk

* DPDK

  As noted above, DPDK is one enabling technology which can support line-rate
  VMs.  Some steps have been taken to support this ("Open vSwitch to use patch
  ports", https://blueprints.launchpad.net/neutron/+spec/openvswitch-patch-port-use,
  patched 2014.2) but other bps are needed:

  * "Support vhost user in libvirt vif driver",
    https://blueprints.launchpad.net/nova/+spec/libvirt-vif-vhost-user

* CPU pinning

  https://blueprints.launchpad.net/nova/+spec/virt-driver-cpu-pinning

* Binding to NIC on host CPU's bus

  https://blueprints.launchpad.net/nova/+spec/input-output-based-numa-scheduling

* Finishing off NUMA & large pages support

  https://blueprints.launchpad.net/nova/+spec/virt-driver-large-pages

* VLAN trunking

  https://blueprints.launchpad.net/neutron/+spec/nfv-vlan-trunks

* Disabling security groups on OVS networks

  https://blueprints.launchpad.net/neutron/+spec/ml2-ovs-portsecurity

The following are not yet addressed.

* Removing redundant bridge

  https://blueprints.launchpad.net/neutron/+spec/ovs-optimization-redundant-bridge

* Disabling connection tracking in host

  [No current blueprint]

* HA

  As above, to deliver 5 9's service Perimeta expects to be deployed spanning
  multiple cloud instances, but if deployed in a single instance it is
  desirable for that cloud to be available as possible.  This is being
  tracked at https://etherpad.openstack.org/p/telcowg-usecase-openstack-ha.

This use case also implicitly places requirement on elements outside core
OpenStack, such as the DPDK OVS mechanism driver
(https://github.com/stackforge/networking-ovs-dpdk).

References
==========

N/a.

