..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

..
  This template should be in ReSTructured text. Please do not delete any
  of the sections in this template.  If you have nothing to say for a
  whole section, just write: None.
  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html
  Please respect a line width of 80 to ensure an easy review in gerrit. After
  review the text will be converted by Sphinx to a HTML website. Sphinx will
  take care of necessary line breaks. 

=============================
 The title of your use case
=============================

Introduction paragraph -- what function are you attempting to support
with OpenStack?

Glossary
========

Provide a list of acronyms, their expansions, and what they actually mean in
general language here. Define any terms that are specific to your problem
domain. If there are devices, appliances, or software stacks that you expect
to interact with OpenStack, list them here.

Remember: OpenStack is used for a large number of deployments, and
the better you communicate your use case the easier it will be to implement.

A good idea to list your acronyms and appropriate description is the use of a
table:

+---------------------+--------------------------------------------------------+
| Term / Abbreviation | Description                                            |
+=====================+========================================================+
| reST                | reStructuredText is a simple markup language           |
+---------------------+--------------------------------------------------------+
| TLA                 | Three-Letter Abbreviation is an abbreviation           |
|                     | consisting of three letters                            |
+---------------------+--------------------------------------------------------+
| xyz                 | Another example abbreviation                           |
+---------------------+--------------------------------------------------------+

Problem description
===================

A detailed description of the problem. This should include the types of
functions that you expect to run on OpenStack and their interactions both
with OpenStack and with external systems.

Examples
--------

In order to explain your use case, if possible, provide an example of a
currently implemented or documented planned solution.

If you have multiple examples (the more the merrier) you may want to use
a numbered list, like the following:

1. 1st Example
2. 2nd Example
3. Sometimes it could be helpful to *format text italic*, e.g. to add
   some background information

Affected By
-----------

If you are aware of any work in progress that will affect this use case,
please list it here.  Include links to a spec or blueprint or bug report
where applicable.

Requirements
============

Use this section to define the functions that must be available or any
specific technical requirements that exist in order to successfully
support your use case. If there are requirements that are external
to OpenStack, note them as such. Please always add a comprehensible
description to ensure that people understand your need.

* 1st Requirement
* 2nd Requirement
* *and so on*

References
----------

If any of your requirements specifically call for the implementation
of a standard or protocol or other well-defined mechanism, use this
section to list them.

* [1]: http://sphinx-doc.org/rest.html


Related Use Cases
=================

If there are related use cases that have some overlap in the problem
domain or that you perceive may partially share requirements or a
solution, reference them here.

Gaps
====

This section should be used to provide information on the difference
between requirements that are currently met within OpenStack and
those that are not. Requirements not met by OpenStack but that the
author feels should be addressed by it should be listed here.

If you are already aware of any gaps that exist in OpenStack that
prevent the implementation of this use case, provide them here and
highlight the related module if possible.
To highlight text you can format it **bold** but you can also make
use of a 
This section can often be left with "None currently known." It is
the purpose of this working group and repository to use the
use cases presented here to identify what the gaps are.

**NAME-THE-MODULE issues:**

* You can list a gaps in bulleted list 

  * with some subitems
  * if it is necessary
  * and refer to a reference [Ref. 1] mentioned above

* Always pay attention on the correct syntax and use blank lines
  between formatted paragraphs.
* Take a look at [Ref. 1] for more details about concepts and syntax
  of reStructuredText (reST).

**ANOTHER-MODULE issues:**

If you want to display lines of code to emphasize the gap you can
make use of a literal code block::

  This a an example code

by using two colons and additonal line breaks at beginning and end.
