..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Upgrade from a nova "baremetal" deployment
==========================================

https://blueprints.launchpad.net/ironic/+spec/upgrade-from-nova-baremetal

This specification describes the technical components required to perform
an upgrade from a deployment of the nova.virt.baremetal driver to Ironic.
It outlines the data migration path and service upgrade process.

NOTE: The service upgrade is only supported within the same release version.
For example, if this work is completed during the Juno cycle, an upgrade from
"juno baremetal" to "juno ironic" will be supported, but a direct upgrade from
"icehouse baremetal" to "juno ironic" will NOT be supported. That should be
accomplished by first upgrading from "icehouse baremetal" to "juno baremetal."

Problem description
===================

The community has split out the functionality of provisioning bare metal
servers into the Ironic project. While the concept was originally demonstrated
by the nova.virt.baremetal driver, and that was intended to be a
proof-of-concept, it may have been deployed in some production environments,
and so a reasonable upgrade path is required.

It is unreasonable to expect operators who have chosen to use
nova.virt.baremetal to lose all state and delete all instances during an
upgrade. Thus, an upgrade path is being provided.

The upgrade process should follow this path:
* build ironic deploy ramdisk and load it in glance
* start maintenance period (prevent tenant from creating new instances)
* update flavor metadata in glance to reference new deploy kernel & ramdisk
* create empty ironic database
* migrate data from nova_bm -> ironic
* start ironic services
* observe ironic log files to ensure take over completed w/o errors
* reconfigure and restart nova-compute service
* end maintenance period

Proposed change
===============

* Add boolean field `nodes`.`legacy_baremetal` to database

* Add data migration tool which will extract the current state from the
  `nova_bm` database, and  convert it to Ironic's data structures and
  populate the `ironic` database.

* Add a flavor update tool which will update the extra_specs of baremetal
  flavors to reference new ironic-appropriate deploy kernel & ramdisk.

* Add process documentation aimed at deployers.

Alternatives
------------

One option is to not provide any upgrade path; this met with opposition from
community leaders.

Another option is to provide a data-only migration (eg, require that instances
be deleted prior to, or as part of, the migration). This was also met with
opposition.

Data model impact
-----------------

This may require an extra field in the database to indicate nodes that were
deployed by nova.virt.baremetal and imported into Ironic. This field should
default to False, and should be set to False during both the tear_down and
deploy methods. It should be set to True only by the data migration script.

This field could be dropped from the database after one release.

Proposed name for the new field: legacy_baremetal


REST API impact
---------------

Exposing the `nodes`.`legacy_baremetal` field on the /v1/nodes/XXX resource.

Driver API impact
-----------------

This does not require any unique changes to the driver API.

Nova driver impact
------------------

None.

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

None

Other deployer impact
---------------------

None

Developer impact
----------------

The only way to ensure that developers do not break the upgrade-ability
will be to create a grenade test that verifies the upgrade path.


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

* Database migration script

* Image update script

* Grenade tests

* Operator documentation


Dependencies
============

This proposal depends on the capability of the ironic-conductor processes to
"take over" an instance deployed by another conductor. This capability will
be leveraged to allow ironic-conductor to take over instances deployed
by nova-baremetal.

Testing
=======

A Grenade test will need to be developed that can:

* deploy an instance using nova-baremetal
* update nova service config to use ironic
* invoke both data migration and flavor update script
* start ironic and restart nova-compute
* confirm ironic-conductor rebuilt the PXE environment by checking
  file system and/or log files
* restart the instance and confirm that it PXE booted by SSH'ing into it


Documentation Impact
====================

Upgrade documentation must be written and maintained for one release cycle.

References
==========

https://etherpad.openstack.org/p/juno-nova-deprecating-baremetal
