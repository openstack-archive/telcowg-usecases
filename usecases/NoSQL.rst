..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

=========================================================
Efficient implementation of Cassandra and other NoSQL DBs
=========================================================

This use case is about how to efficiently deploy Cassandra or other NoSQL DBs
in OpenStack.

Cassandra is an example of a high performance highly-scalable NoSQL DB
originally designed for use in bare metal environments but which has many
uses in the cloud.  In the context of Telco applications and NFV, it is
widely used to provide geo-redundant storage.

Cassandra's performance is highly dependent on the characteristics of the
storage devices available to it.  Because it was designed for bare metal
environments, those characteristics do not easily map onto current OpenStack
storage primitives, leading to inefficiencies when it is run in the cloud.

This use case is about the ability to deploy Cassandra (or other apps with
similar characteristics) efficiently in OpenStack.  Note that it specifically
relates to deploying Cassandra running as VMs in the cloud, rather than the
issue involved in using Cassandra as the DB within Trove to provide DBaaS.
While DBaaS is a perfectly valid deployment model, for many use cases it is
not appropriate, most notably the geo-redunancy one noted above where a global
geo-redundant service is spread across multiple entirely independent clouds.

Glossary
========

**NoSQL**
  Shorthand for "non relational databases".

**JBOD**
  Just a Bunch Of Disks.  A storage architecture using multiple hard drives
  without redundancy or striping.

**IMS**
  IP Multimedia Subsystem

Problem description
===================

Cassandra was designed for a bare metal environment where it would run on a
set of independent nodes, with each node providing compute and storage
(typically JBOD storage within each compute server, but could be a directly
attached storage array).  As no two nodes shared any storage, no single storage
device failure could impact two or more nodes.

Cassandra provides redundancy at the application level, by replicating
data between nodes.  If any single node fails, data is still available for
read and write access (there are qualificaitons due to the choice of
data replication factor, whether the client is insisting on quorum reads and
writes etc.).  Nodes therefore do not themselves need to be particularly
highly available.  In fact, using redundant storage within a node is
inefficient, increasing the storage space required for no real benefit. It is
viewed as an anti-pattern - the model is that redundancy is implemented
at the application level, and should not be duplicated at the storage level.

Within a node, Cassandra has two main data structures, the journal and the
SSTables.  The journal is small but has very high IO and low latency
requirements, particularly for write.  SSTables are large and are
immutable once written, but need high read IO.  Given that, it is very
strongly recommended that the journal and SSTables are separated onto
different physical media, especially if using HDDs.  If the journal and
SSTables are shared on the same HDD then the serial-write-intensive
journal pattern and the random read SSTable pattern interact very badly.

Summarising, Cassandra requires the following for efficient performance.

* Independent compute nodes attached to non-overlapping, non-redundant storage
  (i.e. the storage associated with each compute node must have no common
  points of failure with the storage for any other compute node).

* Support for separate storage pools guaranteeing physical separation of
  storage, with one pool suitable for very high IO/low latency access to
  relatively small amounts of data, and another pool for large data.

These requirements are trivially easy to meet in bare metal: standard practice
is to use compute nodes with one HDD for the host OS and software, one SSD
for the journal, and a bunch of non-RAIDed disks for the SSTables.

These requirements are not currently possible to meet in OpenStack, though.

* Nova ephemeral storage can easily be made non-overlapping and non-redundant
  by implementing it over local storage - but it can't provide separate
  storage pools.

* VMs can generally achieve separation by using a mix of nova + cinder or
  multiple cinder volumes, but this will generally be redundant and (depending
  on the back end) it may not be possible to guarantee separation (for example,
  if nova and cinder are both backed by ceph).

  Even if the underlying storage layer does support storing data
  non-redundantly, there is no easy way to express the required storage
  separation.  Cinder supports volume anti-affinity filters, but these have two
  problems.

  * Anti-affinity operates at the level of cinder back-ends, rather than being
    passed down to the storage provider to implement.  This doesn't scale.

  * Even if it were passed down, anti-affinity is expressed pair-wise so
    would need to be specified against every extant volume attached to any
    Cassandra VM, which is a complex orchestration problem.  There is no
    concept analogous to nova server group (anti)affinity.

User Stories
------------

As a Telco, I want to deploy an IMS core as a Virtual Network Function
running in OpenStack.  The IMS service needs to be scalable, global and
geo-redundant.  To do that, I want to store user config and state in a
Cassandra DB.  User data is large: Cassandra is replicating it for redundance
both locally and globally; I do not want to have to pay for further replication
and redundancy to store each individual Cassandra copy.

Examples
--------

Any application which wishes to deploy Cassandra in OpenStack.

Affected By
-----------

None.


Requirements
============

It should be possible for VMs to have access to non-redundant, non-overlapping
block device storage, with that storage able to be allocated from pools with
different characteristics in terms of performance, capacity etc.

Related Use Cases
=================

None as yet, but many other NoSQL DBs will have similar requirements.

Gaps
====

The above requirements currently suffer from these gaps.

* Nova storage can be used to provide non-redundant, non-overlapping storage,
  but not drawn from separate pools.

* Cinder has no semantic to express non-redunancy, and even if abcked with
  non-redundant storage it has no easy way of expressing the required
  anti-affinity.

This suggests a few approaches that could be taken.

* Add a concept of nova ephemeral pools, akin to cinder pools, offering
  different grades of storage. These could be used to provide one pool mapped
  to local host SSD and another to local host JBOD, used for the journal and
  SSTables respectively.

* Have cinder support a concept of separation which is passed down to the
  storage provider rather than implemented at the level of cinder back-ends.

* Add a concept of volume group anti-affinity to cinder.  Then nova ephemeral
  storage backed by local host SSD could be used for the journal, and a cinder
  pool backed by non-redundant storage could be used for the SSTables.  Each VM
  would be attached to a single volume, all belonging to the same volume
  group with hard anti-affinity specified.  Alternatively, and latency
  permitting, separate non-redundant cinder volumes could be used for both the
  journal and SSTables, with all of them belonging to the same volume
  anti-affinity group.

References
==========

* http://www.datastax.com/dev/blog/what-is-the-story-with-aws-storage
* https://academy.datastax.com/demos/datastax-enterprise-deployment-guide-google-compute-engine
