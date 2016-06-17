Guide to the Datera Cinder Fuel Plugin
**************************************

:Version: 1.0-1.0.0-1
:MOS Fuel Version: 7.0 and 8.0
:Openstack Version: Kilo and Liberty
:Product: Fuel Cinder Driver Plugin

This document provides instructions for installing, configuring, troubleshooting and using
the Datera Cinder Plugin for Fuel 7.0 and 8.0.

.. contents::
    :depth: 2

Key terms, acronyms and abbreviations
=====================================

Cluster Management VIP:
    Management Virtual IP (MVIP) is the IP address (or hostname) of
    the management interface to the Datera Elastic Data Fabric.

Admin login:
    The Cluster Admin or Openstack login on a Datera EDF.

Datera Replication Factor:
    The number of replicas of a volume will be made in the Datera EDF.

Datera Cinder
=============

The Datera Cinder Fuel plugin automates the addition of the required
lines to the cinder.conf file to leverage the Datera EDF. It also
installs the correct cinder driver. The Datera EDF API has moved to v2
and prior to Mitaka only v1 is supported by the bundled Datera cinder
driver, v2 is not backwards compatible.
The Datera Fuel plugin also extends the UI and enables a user to provide
the necessary details to configure cinder correctly,

License
-------

=======================   ==================
Component                  License type
=======================   ==================
Datera Cinder Driver      Apache 2.0

=======================   ==================

Requirements
------------

======================= ==================
Requirement             Version/Comment
======================= ==================
Fuel                    7.0
----------------------- ------------------
Fuel                    8.0
======================= ==================

Prerequisites
--------------

* The Datera EDF should be conffigured and reachable for the compute and
  controller+cinder nodes.

* The EDF has to be configured and reachable prior to starting Cinder with
  the Datera EDF configuration in place.

Limitations
-----------

* Fuel 7.0 does not support multiple storage backends in Cinder,
  therefore the Datera Cinder Fuel plugin when applied to 7.0 does not
  support multiple backends.

* The Datera Cinder Fuel Plugin does support creating a multi-backend
  configuration by selecting the checkbox in Fuel 8.0.

Installation Guide
==================

.. _install:

Datera Cinder plugin installation
---------------------------------

#. Download the plugin from
   `Fuel Plugins Catalog <https://www.mirantis.com/products/openstack-drivers-and-plugins/fuel-plugins/>`_.

#. Copy the plugin to Fuel Master node. If no Fuel master node is present
   please follow the instructions of the Mirantis Fuel documentation::

    scp fuel-plugin-datera-cinder-1.0-1.0.0-1.noarch.rpm root@:<the_Fuel_Master_node>:

#. Log into the Fuel Master node and install the plugin::

    fuel plugins --install fuel-plugin-datera-cinder-1.0-1.0.0-1.noarch.rpm

#. Verify the installation of the Datera Cinderf Fuel plugin::

    fuel plugins --list

.. _configure:

Datera Cinder plugin configuration
----------------------------------

The configuration for both Fuel 7.0 and 8.0, differences will be marked.

#. After the plugin is installed, create a new OpenStack environment following:
    `The openstack creation guide for Mirantis Fuel 7.0 <https://docs.mirantis.com/openstack/fuel/fuel-7.0/user-guide.html#create-a-new-openstack-environment>`_. or
    `The quickstart guide for Mirantis Fuel 8.0 <https://docs.mirantis.com/openstack/fuel/fuel-8.0/quickstart-guide.html>`_.

#. Configure your environment following:
    `The Mirantis Fuel 7.0 Configuration Documentation <https://docs.mirantis.com/openstack/fuel/fuel-7.0/user-guide.html#configure-your-environment>`_. or
    `The Mirantis Fuel 8.0 Install Guide <http://docs.openstack.org/developer/fuel-docs/userdocs/fuel-install-guide.html>`_.
    ** Make sure to also add the Cinder role for the controllers, if the Cinder role is not selected the Datera EDF plugin will NOT configure Cinder **

#. In Fuel 7.0: Open the *Settings tab* of the Fuel web UI, scroll down,
   select 'Fuel plugin to enable Datera driver in Cinder.' on the left.

#. In Fuel 8.0: Open the *Settings tab* of the Fuel web UI, scroll down
   select 'storage' on the left and select the Datera Fuel plugin.

#. 'Multibackend enabled ([datera] context)':
    By default the Datera plugin will not use Multibackend as a configuration option. This means that the configuration options will be set in the 'default' context of the cinder.conf file. In Fuel 8.0 the 'Multibackend enabled' checkbox can be set to support  multiple backends in cinder, which will put the configuration options in it's own Datera context.

#. 'Cluster Management VIP (san_ip)':
    The IP or DNS name of the management VIP for the Datera EDF.

#. 'Login for Admin account (san_login)':
    The username of the account that has the correct rights to provision storage, this can be the Openstack user, the admin user or a specific tenant user.

#. 'Password for Admin account (san_password)':
    The password for the previously mentioned account.

#. 'Data replication factor (datera_num_replicas)':
    This setting dictates how many copies of an app-instance will be distributed over the Datera EDF using its smart placement policies.

#. When configuration is complete the network check can be run and the environment can be deployed.

.. _verify:

Plugin Verification
-------------------

After configuring the plugin and the environement is deployed verification of the plugin can commence.

#. Use a web browser and go to the Datera web UI, login and go to the *Bell Symbol* in the upper right hand corner and select *User Activity*.

An alternative to the WebUI is using the REST API. The interactive browser for the REST API can be found on port 7800 of the Cluster management VIP adres.

#. Here's an example, where 192.168.123.20 should be replaced with your Datera Management VIP, using the REST API from a bash shell::

    DATKEY=`curl -X PUT -H "auth-token: " -H "content-type: application/json" --data '{"name":"admin","password":"password"}' http://192.168.123.20:7717/v2/login | grep key | awk '{ print $2 }'`

    watch "curl -X GET -H \"auth-token: $DATKEY\" -H \"content-type: application/json\" http://192.168.123.20:7717/v2/audit_logs | tac"

#. Plugin verification is done by running the OpenStack Test Framework (OSTF):
    Open the *Health Check tab* in the Fuel web UI, check the *Select All box* and push the *Run Tests button*. The section of main interest for the plugin is the *Fuctional tests section*.

#. If OSTF fails in the *functional tests section* check the troubleshooting_ guide.

User Guide
==========

Once Openstack is deployed by Fuel the Datera plugin provides no further
configuration or maintenance options.
The logs for the Datera EDF driver will output all the logging in the
cinder-volume log.

.. _troubleshooting:

Troubleshooting
===============

Prior to using this troubleshooting guide make sure that the install_ of the plugin had no problems, the deployment of the environment was successful and the OpenStack UI, Horizon, is reachable from the Fuel UI. If this is not the case please refer to the Official Mirantis documentation mentioned in configure_.

When the OSTF checks fail, or if deployment of volumes and/or instances with regard to their respective block devices fails use the following guidelines to troubleshoot.

1. All connections to the OpenStack nodes are through the Fuel Master node, so make sure you log in there first::

    ssh root@<the_Fuel_Master_node>

2. Go to the nodes with the Cinder role and check the */var/log/cinder/cinder-volume.log*. Typical *Failed to initialize driver.* errors here are:

  .. _connectivity:

  a) Connectivity error::

        Failed to make a request to Datera cluster endpoint due to the following reason: ('Connection aborted.', error(113, 'EHOSTUNREACH'))

    Check the connectivity from the nodes with the Cinder role to the Datera Management VIP and make sure connectivity can be established::

        root@node-2:~# nc -z -v 192.168.123.20 7717
        Connection to 192.168.123.20 7717 port [tcp/*] succeeded!

    If this is not the case make sure the network that the Datera EDF lives in is reachable from the Fuel networks.

  .. _apiversion:

  b) API Version error::

        DateraAPIException: Request to Datera cluster returned bad status: 400 | Bad Request

    This can only occur if someone altered the */etc/cinder/cinder.conf* on the Cinder nodes and manually added the *datera_api_version* and set it to *1*, only *2* is supported which is the default.

  .. _credentials:

  c) Username and Password Errors::

         Logging into the Datera cluster failed. Please check your username and password set in the cinder.conf and start the cinder-volumeservice again.

    Make sure the Username and Password are correct and were correctly entered in configure_.

  .. _targetunreachable:

  d) iSCSI is unreachable for the Compute nodes::

         ImageCopyFailure: Failed to copy image to volume: iscsiadm: No session found.

    or::

         Exception during message handling: Failed to copy image to volume: iscsiadm: No session found.

    or in general::

         iscsiadm: No session found.

    Validate connectivity from the Compute nodes to an Application Instance IP. Log in to the Datera web UI and use the *Provision Storage button*. Fill in the required fields, marked with a `*` and click the *Provision button*. If the application instance is not created and/or does not come on line please contact your Datera Systems or Support Engineer, as your problem is not related to Fuel or OpenStack. If successful the *Application Instance* will get an *Access IP* and a *Target IQN*::

         root@node-5:~# nc -z -v 192.168.123.119 3260
         Connection to 192.168.123.119 3260 port [tcp/iscsi-target] succeeded!

    Validate that the discovery of the iSCSI targets actually works from the Compute nodes::

         root@node-4:~#  iscsiadm -m discovery -t sendtargets -p 192.168.123.119
         192.168.123.119:3260,1 iqn.2013-05.com.daterainc:tc:01:sn:b3ef26a6b011aaab

  e) Use the Datera UI or CLI *event* and *audit* logs as mentioned in the verify_ to verify that the system is actually receiving requests from Openstack.

  f) For other errors related please contact the `maintainers <https://launchpad.net/fuel-plugin-datera-cinder>`_ of the Datera Cinder Fuel Plugin or create a bug in `launchpad <https://bugs.launchpad.net/fuel-plugin-datera-cinder>`_.

3. When OSTF fails and the Datera Cinder plugin is configured incorrectly or the VIP is not reachable OSTF will have 2 errors that are the same::

    1) Create volume and boot instance from it::

      Failed to get to expected status. In error state. Please refer to OpenStack logs for more details.
      ...
      2. Wait for volume status to become "available".

    2) Create volume and attach it to instance::

      Failed to get to expected status. In error state. Please refer to OpenStack logs for more details.
      ...
      2. Wait for volume status to become "available".

4. When the iSCSI port of the Application Instances is not available for the Compute nodes OSTF will show 2 different errors::

    1) Create volume and boot instance from it::

      Failed to get to expected status. In error state. Please refer to OpenStack logs for more details.
      ...
      2. Wait for volume status to become "available".

    2) Create volume and attach it to instance::

      Time limit exceeded while waiting for volume becoming 'in-use' to finish. Please refer to OpenStack logs for more details.
      ...
      7. Check volume status is "in use".

Known issues
============

Due to Fuels lack of support for multiple cinder backends in Fuel 7.0, only a
single storage vendor backend may be automatically configured within Fuel
If multiple vendors are required the cinder.conf needs to be edited manually for
Fuel 7.0.

Appendix
========
* `Datera EDF <http://www.datera.io/>`_
* `Mirantis Fuel Plugins <https://www.mirantis.com/validated-solution-integrations/fuel-plugins/>`_
