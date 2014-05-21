..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================================
Upgrade from a nova "baremetal" deployment to Ironic
====================================================

https://blueprints.launchpad.net/ironic/+spec/upgrade-from-nova-baremetal

This specification describes the technical components required to perform
an upgrade from a deployment of the nova.virt.baremetal driver to the
nova.virt.ironic driver. It outlines the data migration path and service
upgrade process.

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

NOTE: The service upgrade is only supported within the same release version,
and will only be supported in the first integrated release containing Ironic.

For example, if this work is completed during the Juno cycle, an upgrade from
"juno baremetal" to "juno ironic" will be supported, but a direct upgrade from
"icehouse baremetal" to "juno ironic" will NOT be supported. That should be
accomplished by first upgrading from "icehouse baremetal" to "juno baremetal"
and then to "juno ironic".

Proposed change
===============

This proposal is to add the following to the Nova codebase:

* Add a data migration tool which will import both nova.virt.baremetal.db.api
  and ironic.db.api, extract the current state from the nova_bm database,
  convert it to Ironic's data structures and populate the `ironic` database.

* Add a flavor update tool which will update the extra_specs of baremetal
  flavors to reference new ironic-appropriate deploy kernel & ramdisk.

* Add process documentation aimed at deployers.

Additionaly, one change will be made to Ironic to facilitate this:

* Add boolean field `nodes`.`imported_from_nova_bm` to ironic database.

At the end of the release cycle following the cycle in which this work is
completed, all of the above will be removed from the code base. Additionally,
the nova.virt.baremetal driver and all related artefacts (eg, the baremetal
host manager) will be deleted from Nova.


The proposed upgrade process should follow this path:

* build ironic deploy ramdisk and load it in glance

* start maintenance period (prevent tenant from creating new instances)

* update flavor metadata in glance to reference new deploy kernel & ramdisk

* create empty ironic database

* migrate data from nova_bm -> ironic

* start ironic services

* observe ironic log files to ensure take over completed w/o errors

* reconfigure and restart nova-compute service

* end maintenance period

Alternatives
------------

Three alternatives have been discussed.

* Do not provide any upgrade path; this met with significant opposition.

* Provide a data-only migration (eg, require that instances be deleted
  prior to, or as part of, the migration). This was also met with opposition.

* Rather than a database migration script, one could enroll instances
  via Ironic's REST API. This would require the REST API to accept nodes
  that have non-null provision_state and power_state, which it expressly
  does not allow today. This change would require significant changes
  to the provisioning API and state management within the conductor service.

Data model impact
-----------------

No data model impact in Nova.

In Ironic, this will require an extra field in the database to indicate nodes
that were deployed by nova.virt.baremetal and imported into Ironic. This field
should default to False, and should be set to False during both the tear_down
and deploy methods. It should be set to True only by the data migration script.

This field could be dropped from the ironic database after one release.

Proposed name for the new field: imported_from_nova_bm


REST API impact
---------------

None

Driver API impact
-----------------

None

Nova driver impact
------------------

Upon completion of this work, the nova.virt.baremetal driver should be marked
deprecated and removed at the beginning of the following release.

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

None


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

* Flavor update script

* Grenade tests

* Operator documentation


Dependencies
============

This proposal depends primarily upon the acceptance of the
nova.virt.ironic driver into the Nova codebase.

It also depends on the capability of the ironic-conductor processes to "take
over" an instance deployed by another conductor. This capability will be
leveraged to allow ironic-conductor to take over instances deployed by
nova-baremetal.

It also depends on adding a new field to the ironic database to indicate
nodes with active instances that were imported from Nova.


Testing
=======

A Grenade test will need to be developed that can:

* deploy an instance using nova.virt.baremetal

* update nova configuration to use the nova.virt.ironic driver

* invoke both data migration and flavor update scripts

* start ironic services and restart nova-compute

* confirm ironic-conductor rebuilt the PXE environment by checking
  file system and/or log files

* request that Nova restart the instance

* confirm that Ironic restarted the instance and that it booted properly


Documentation Impact
====================

Upgrade documentation must be written and maintained for one release cycle.


References
==========

https://etherpad.openstack.org/p/juno-nova-deprecating-baremetal
