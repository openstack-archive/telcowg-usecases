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
