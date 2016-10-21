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

When writing use cases, focus on "what" you want to do and "why" rather than
specific OpenStack requirements or solutions. Our aim as a working group is to
assist in distilling those requirements or solutions from the use cases
presented to ensure that we are building functionality that benefits all
relevant telecommunications use cases.

Submission of use cases that pertain to different implementations of the same
network function are welcome, as are use cases that speak to the more general
demands telecommunications workloads place upon the infrastructure that
supports them. In this initial phase of use case analysis the intent is to
focus on those workloads that run on top of the provided infrastructure before
moving focus to other areas.

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
* Check out the telcowg-use cases git repository::
  $ git clone https://git.openstack.org/stackforge/telcowg-usecases.git
  $ cd telcowg-usecases
  $ git review -s
  $ git config --global gitreview.username yourgerritusername
  $ git checkout -b my_use_case
* Change into the **usecases** directory::
  $ cd usecases
* Copy the **template.rst** into a filename that describes your use case::
  $ cp ../template.rst ./my_use_case.rst
* Edit the content until you are happy with it then use tox to verify the
  RST syntax and resultant rendered HTML::
  $ tox -e docs
* Finally submit the content to Gerrit::
  $ git add my_use_case.rst
  $ git commit
  $ git review

TODO: To update a use case

.. _review:

Review Use Case
===============

* **Owner**: Reviewer

.. _merge:

Approve Use Case
================

* **Owner**: Core Reviewer

Approving a use case is done once at least two core reviewers have assigned a
+2. To approve one of the core reviewers must set the workflow score in
Gerrit to +1. For the purposes of this document the core reviewer that sets the
workflow score in Gerrit to +1 is the approver.

It is expected that core reviewers of the **telcowg-usecases**
repository will only do this once they are confident that a positive review has
been received from a member of the development community for each impacted, or
potentially impacted, OpenStack project. This member of the development
community does not necessarily have to be a core reviewer on the relevant
OpenStack project but should be in good standing.

It is the responsibility of the **telcowg-usecases** core review group to use
their judgement in establishing that this is the case, reaching out to relevant
SMEs including the PTLs and core reviewer groups for the impacted project where
there is doubt.

In addition before approval the core reviewers of the **telcowg-usecases**
repository must ensure that the proposed use case is understandable for
consumers who are not well versed in telecommunications or network function
virtualization specific language. This includes ensuring that all domain
specific abbreviations are explained in detail.

At the time of approval the approving core reviewer must also raise a
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

* Determine potentially impacted project(s).
* Determine whether potentially impacted project(s) use some variation of an
  RFE or backlog specification process for accepting requirements.

Where potentially impacted project(s) use some variation of an RFE or backlog
specification process for accepting use cases:

* Check whether an RFE or backlog specification exists for the gap or
  requirement.
* Where an RFE or backlog specification does not exist - create one.
* Link the RFE or backlog specification from the tracker for the use case.

Where potentially impacted project(s) do not use such a process:

* Check whether a bug exists for the gap or requirement against the
  `openstack-telcowg <https://launchpad.net/openstack-telcowg>`_ Launchpad
  project.
* Where a bug does not exist - create one.
* Link the bug from the tracker for the use case.

The tracker bug(s) will be used to link to relevant blueprints, specifications,
bugs, etc. in other OpenStack projects that relate to designing and
implementing solutions for the requirement or gap in question.

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
