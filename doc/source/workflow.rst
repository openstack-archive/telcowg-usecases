..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License http://creativecommons.org/licenses/by/3.0/legalcode

==========
 Workflow
==========

Roles
=====

* **Submitter**
* **Reviewer**
* **Core Reviewer**
* **Approver**

Process
=======

1. submit_
2. review_
3. merge_
4. raise_
5. assign_
6. create_
7. implement_
8. verify_

.. _submit:
Submit Use Case
===============

* **Owner**: Submitter

.. _review:
Review Use Case
===============

* **Owner**: Reviewer

.. _merge:
Approve Use Case
================

* **Owner**: Core Reviewer

Approving a use case is done once at least two core reviewers have assigned a
+2. To approve one of the core reviewers must grant the +W (workflow) score in
Gerrit. At the time of approval the approving core reviewer must also raise a
bug corresponding to the use case. This will function as the master tracker and
be used to link all bugs raised in subsequent steps back to the use case.

The master tracker must be:

* Raised in the `openstack-telcowg <https://launchpad.net/openstack-telcowg>`_
  Launchpad project.
* Titled **Master: <use case>** where use case is replaced by the name of the
  use case as present in the git submission, e.g. **sec_segregation**.

The description must include a link back to the merged use case, e.g.
http://git.openstack.org/cgit/stackforge/telcowg-usecases/tree/usecases/sec_segregation.rst

.. _raise:
Raise Tracker Bugs
==================

* **Owner**: Approver

For each identified gap or requirement:

* Check whether a bug exists for the gap or requirement against the
  `openstack-telcowg <https://launchpad.net/openstack-telcowg>`_ Launchpad
  project.
* Where a bug does not exist - create one.
* Tag the relevant bug, newly created or existing, using the name of the use
  case as present in git, e.g. **sec_segregation**.

.. _assign:
Assign Tracker Bugs
===================

* **Owner**:

.. _create:
Create Project Bug/Blueprint
============================

* **Owner**:

.. _implement:
Implement Solution to Bug/Blueprint
===================================

* **Owner**:

.. _verify:
Verify Solution to Bug/Blueprint
================================

* **Owner**:
