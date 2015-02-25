..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

=============================
 Virtual IMS Core
=============================

Project Clearwater, http://www.projectclearwater.org/. An open source
implementation of an IMS core designed to run in the cloud and be massively
scalable. It provides SIP-based call control for voice and video as well as SIP
based messaging apps. As an IMS core it provides P/I/S-CSCF function together
with a BGCF and an HSS cache, and includes a WebRTC gateway providing
interworking between WebRTC & SIP clients.

TODO: Move Project Clearwater info to glossary and/or problem description.
TODO: Generalize above.

Glossary
========

* IMS -
* SIP -
* P/I/S-CSCF -
* BGCF -
* HSS -

Problem description
===================

A detailed description of the problem. This should include the types of
functions that you expect to run on OpenStack and their interactions both
with OpenStack and with external systems.

* Mainly a compute application: modest demands on storage and networking.
* Fully HA, with no SPOFs and service continuity over software and hardware
  failures; must be able to offer SLAs.
* Elastically scalable by adding/removing instances under the control of the
  NFV orchestrator.

Examples
--------

In order to explain your use case, if possible, provide an example of a
currently implemented or documented planned solution.

Gaps
----


If you are already aware of any gaps that exist in OpenStack that
prevent the implementation of this use case, provide them here.

Affected By
-----------

If you are aware of any work in progress that will affect this use case,
please list it here.  Include links to a spec or blueprint or bug report
where applicable.

Requirements
============

* Compute application:

  * OpenStack already provides everything needed; in particular, there are no
     requirements for an accelerated data plane, nor for core pinning nor NUMA

* HA:

  * implemented as a series of N+k compute pools; meeting a given SLA requires
    being able to limit the impact of a single host failure
  * potentially a scheduler gap here: affinity/anti-affinity can be expressed
    pair-wise between VMs, which is sufficient for a 1:1 active/passive
    architecture, but an N+k pool needs a concept equivalent to
    "group anti-affinity" i.e. allowing the NFV orchestrator to assign each VM
    in a pool to one of X buckets, and requesting OpenStack to ensure no single
    host failure can affect more than one bucket (there are other approaches
    which achieve the same end e.g. defining a group where the scheduler ensures
    every pair of VMs within that group are not instantiated on the same host)
    for study whether this can be implemented using current scheduler hints

* Elastic scaling:

  * as for compute requirements there is no gap - OpenStack already provides
    everything needed.

References
----------

* https://wiki.openstack.org/wiki/TelcoWorkingGroup/UseCases#Virtual_IMS_Core

Related Use Cases
=================

