..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================
Libvirt-lxc User Namespace Support
==================================

https://blueprints.launchpad.net/nova/+spec/libvirt-lxc-user-namespaces

User namespaces provide a way for a process running in a container to appear to
be running as root, but are in fact running as a different user on the host.
The objective of this feature is to allow deployers to enable and configure
which users and groups are mapped between container and host.

Problem description
===================

It is a security risk to allow user processes to run as root on container
hosts. In order to mitigate this risk, it is a good idea to run processes in
those containers as non-root users. The problem with this is some processes
may like to run (or at least appear to run as root inside the container).


Proposed change
===============

User namespaces allow processes inside a container to appear to be run as root,
but are in fact running as another user. Libvirt exposes this feature through
idmaps. This change would introduce a set of elements on the instance's domain
xml to indicate which user and group ids should map between container and host.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

This change will improve the security of containers in Nova significantly.
Before this change, processes running in containers built by Nova will be run
as the host's root user. After this change, a deployer can restrict which
user(s) processes will be run as.

Notifications impact
--------------------

None

Other end user impact
---------------------

Images need to be deliberately created to be run in a user namespaced
environment. The contents of an image's filesystem need to be owned by the
target uid/gid. There are a few different ways this could be handled:

* Images could be chowned by creator before upload to Glance

* Images could be chowned by Glance on import

* Images could be chowned by Nova when caching

* Images could be chowned by Nova on boot

Performance Impact
------------------

Depending on where the image is chowned, there will be a performance impact.

* Chown by image creator: No performance impact

* Chown by Glance on import: Image will take longer to become active

* Chown by Nova when cached: Initial boot on all hosts will take longer

* Chown by Nova on boot: All boots will take longer

Other deployer impact
---------------------

Config for this feature will be disabled by default. It will be up to the
deployer to enable and configure it.

New config options in libvirt group:

* idmap_uid: user id on host that root inside the container will be mapped to

* idmap_uid_count: number of subsequent users allowed inside container

* idmap_gid: group id on host that root inside the container will be mapped to

* idmap_gid_count: number of subsequent groups allowed inside container

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  andrew-melton

Other contributors:
  rconradharris
  thomas-maddox

Work Items
----------

* Add Libvirt Config objects and implement config options in Libvirt driver

  * https://review.openstack.org/#/c/94454/

Other potential items

* Add chown support on Glance import

* Add chown support on Nova when image is cached

* Add chown support on Nova boot


Dependencies
============

* Potential dependency on implementation of chown on import in Glance

Testing
=======

Making sure that the nova config options are properly mapped to libvirt domain
objects can easily be handled by unit testing. Functional testing for this will
not be possible until libvirt-lxc is included in the CI environment. Depending
on how chowning is implemented, functional testing could be a bit tricky.

Documentation Impact
====================

Other than documentation of new config options. Documentation would need to be
provided if end users are required to chown images before import to let them
know that is necessary. If the chown is done by Glance on import, deployers
would need to know that Glance would need to be configured to chown on import
when this feature is enabled. If chown is down by Nova on boot or cache, there
would be no further documentation.

References
==========

* http://libvirt.org/formatdomain.html#elementsOSContainer

* http://libvirt.org/drvlxc.html#secureusers

* https://lwn.net/Articles/532593/
