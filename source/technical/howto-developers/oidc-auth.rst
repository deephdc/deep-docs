Using Openstack API with OIDC tokens
====================================

|Using Openstack API| |with OIDC tokens|

Note: this guide is made for GNU/Linux distributions

Create file for OIDC
--------------------

After downloading Openstack RC file (Identity API v3) from Horizon web
panel, the following changes are necessary for the authentication with
OIDC (review the missing fields):

::

   #!/usr/bin/env bash
   export OS_AUTH_URL="https://<keystone API url ending in /v3>"
   export OS_AUTH_TYPE="v3oidcaccesstoken"
   export OS_PROJECT_ID="<project id comes with downloaded RC v3 file>"
   export OS_USERNAME="<username comes with downloaded RC v3 file>"
   export OS_REGION_NAME="<comes with downloaded RC v3 file>"
   # Don't leave a blank variable, unset it if it was empty
   if [ -z "$OS_REGION_NAME" ]; then unset OS_REGION_NAME; fi
   export OS_INTERFACE="<comes with downloaded RC v3 file>"
   export OS_IDENTITY_API_VERSION=3
   export OS_IDENTITY_PROVIDER="deep-hdc"
   export OS_PROTOCOL="oidc"
   export OS_ACCESS_TOKEN="<get access token from MitreID dashboard after login to openstack horizon>"

The keystone API url can be obtained from Horizon web panel in
Project->Access & Security->API Access, associated with the service name
``Identity``.

The access token is obtained from the login in Openstack Horizon. It’ll
create an access token that can be reviewed in MitreID dashboard.

All the other attributes for authentication are in the RC v3 file
downloaded from Horizon web panel.

Using Openstack CLI
-------------------

After loading all necessary environment variables to the shell, just run
openstack command. To create a RC file for OIDC you can do the
following:

::

   cat << EOF
   #!/usr/bin/env bash
   export OS_AUTH_URL="https://<keystone API url ending in /v3>"
   export OS_AUTH_TYPE="v3oidcaccesstoken"
   export OS_PROJECT_ID="<project id comes with downloaded RC v3 file>"
   export OS_USERNAME="<username comes with downloaded RC v3 file>"
   export OS_REGION_NAME="<comes with downloaded RC v3 file>"
   # Don't leave a blank variable, unset it if it was empty
   if [ -z "$OS_REGION_NAME" ]; then unset OS_REGION_NAME; fi
   export OS_INTERFACE="<comes with downloaded RC v3 file>"
   export OS_IDENTITY_API_VERSION=3
   export OS_IDENTITY_PROVIDER="deep-hdc"
   export OS_PROTOCOL="oidc"
   export OS_ACCESS_TOKEN="<get access token from MitreID dashboard after login to openstack horizon>"
   EOF > oidc_RCfile.sh

Then execute the file to load variables definitions into shell
environment (no need to give execution previleges):

``. ./oidc_RCfile.sh``

For example, to list all images available run the following command:

``openstack image list``

Since the list of images cloud be large, this should be better:

``openstack image list | less``

For more details about all available openstack cli commands review the
following page:

`Openstack CLI`_

.. _Openstack CLI: https://docs.openstack.org/python-openstackclient/rocky/cli/command-list.html

.. |Using Openstack API| image:: ../../_static/rocky-release-logo.png
  :scale: 50%
.. |with OIDC tokens| image:: ../../_static/OpenID_Connect.png
  :scale: 20 %
