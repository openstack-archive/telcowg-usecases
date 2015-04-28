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

The Telecommunications Working group welcomes use cases from Communication
Service Providers (CSPs), Network Equipment Providers (NEPs) and other
organizations in the telecommunications industry. To begin adding a use case
simply copy the "Template" section of this page to the bottom of the list and
rename it to a name that describes your use case.

When writing use cases focus on "what" you want to do and "why" rather than
specific OpenStack requirements or solutions. Our aim as a working group is to
assist in distilling those requirements or solutions from the use cases
presented to ensure that we are building functionality that benefits all
relevant telecommunications use cases. Submission of use cases that pertain to
different implementations of the same network function (e.g. vEPC) are welcome
as are use cases that speak to the more general demands telecommunications
workloads place upon the infrastructure that supports them. In this initial
phase of use case analysis the intent is to focus on those workloads that run
on top of the provided infrastructure before moving focus to other areas.

To submit a use case:

* Review existing use cases `in the repository <http://git.openstack.org/cgit/stackforge/telcowg-usecases/tree/usecases>`_
  and `in the review queue <https://review.openstack.org/#/q/status:open+project:stackforge/telcowg-usecases,n,z>`_
  as it is possible somebody has already submitted the same use case or a
  similar use case. In such cases consider submitting an update to the existing
  use case instead.
* Where no existing use case exists you will want to submit a new use case.
  First ensure that you have followed the steps in the
  `Developer's Guide <http://docs.openstack.org/infra/manual/developers.html>_`
  to set your system up for working with OpenStack git repositories.
* Check out the telcowg-use cases git repository:
  $ git clone https://git.openstack.org/stackforge/telcowg-usecases.git
  $ cd telcowg-usecases
  $ git review -s
  $ git config --global gitreview.username yourgerritusername
* Change into the **usecases** directory:
  $ cd usecases
* Copy the **template.rst** into a filename that describes your use case:
  $ cp ../template.rst ./my_use_case.rst
* Edit the content until you are happy with it, then submit to Gerrit:
  $ git add my_use_case.rst
  $ git commit
  $ git push gerrit HEAD:refs/for/master

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
