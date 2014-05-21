..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================
Standardize logging and improve log coverage
============================================

https://blueprints.launchpad.net/ironic/+spec/improve-logging

This blueprint will outline the appropriate log level to use for different
circumstances, and will be retained as a wiki page upon completion.

Problem description
===================

At the moment, logging in Ironic is sporadic and inconsistent, making it hard
for operators to debug issues, and hard for developers to know when to add
logging or which log level to use. It does not adhere to the OpenStack log
standards, which are outlined here:

https://wiki.openstack.org/wiki/LoggingStandards


Proposed change
===============

* ensure third-party libraries only log at WARN level
* Use the new oslo log level macros (_LE, _LW, _LI, etc).
* ensure all DEBUG logs are not translated
* ensure INFO logs are only used to record a "unit of work" completion
* ensure WARNING are only used to indicate non-critical problems that
  may suggest investigation, but do not require it.
* ensure ERROR and TRACE are only used for fatal conditions that require
  operator investigation.

Additionally, ensure all log messages are human-readable and indicative of the
cause of the problem. Cryptic messages, or messages which require deep
knowledge of the codebase, are not helpful to operators and should be edited.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Driver API impact
-----------------

None

Nova driver impact
------------------

None

Security impact
---------------

None

Other end user impact
---------------------

None

Scalability impact
------------------

None

Performance Impact
------------------

While this may increase the total number of log lines, it should have no
significant performance impact.

Other deployer impact
---------------------

This should significantly improve the ease of running and manitaining an Ironic
deployment by making the logs more consistent and easier to understand, and
filling in gaps in our current log coverage.

Developer impact
----------------

This will require future developers (and core reviewers) to be cognizant of
the expected logging behavior and continue to maintain consistency.


Implementation
==============

Assignee(s)
-----------

TBD

Primary assignee:
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Work Items
----------

* remove _() around all debug strings

* ensure debug messages are useful without being overly verbose

* change LOG.error/warn/info to _LE/_LW/_LI

* ensure existing error and warning logs are representative of real errors and
  warnings

* ensure info log messages are recorded for all units-of-work that are
  pertinent to operators. This may require a significant audit of the codebase
  and several patches to land.


Dependencies
============

None

Testing
=======

XXX: Should we add a hacking rule to enforce any of the above changes?

Documentation Impact
====================

No impact on deployer docs.

Developer docs will require a section describing our log policy.


References
==========

https://wiki.openstack.org/wiki/LoggingStandards

