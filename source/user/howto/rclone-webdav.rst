Using rclone with webdav provider
=================================

This guide was tested on ubuntu linux distribution, but should also work
for any other distribution. For that you need at least rclone v1.39,
since only from this release WebDAV become to be supported.

`Rclone`_ (“rsync for cloud storage”) is a command line program to sync
files and directories to and from different cloud storage providers. In
this guide we will show how to use it from a local file system with
webdav provider. As webdav provider we will use Nextcloud 15 service.

Install rclone
--------------

To install rclone CLI in ubuntu run the following command:
``apt install rclone``.

A quick alternative for those with a distribution with older version is
to download the pre-compiled binary and use it as is. To do so just
fetch and unpack the following zip:

::

   wget http://downloads.rclone.org/rclone-current-linux-amd64.zip
   unzip rclone-current-linux-amd64.zip

Copy binary file rclone to directory /usr/bin. This is mandatory since
the script have hard coded references to that path as in current release
(rclone v1.45):

::

   cd rclone-*-linux-amd64
   sudo cp rclone /usr/bin

For installing it from source review the `rclone install
documentation`_.

Configure rclone
----------------

The following configuration steps were based on the `rclone webdav
documentation`_. The following procedure use single sign on
authentication with Nextcloud provider.

First start to run ``rclone config`` in a shell. This will set the
necessary parameters to access webdav provider. In any prompt during the
configuration you can type ``q`` and exit the configuration.

After running the command for the first time, the following information
appears:

::

   <date> <hour> NOTICE: Config file "/home/$USER/.config/rclone/rclone.conf" not found - using defaults
   No remotes found - make a new one
   n) New remote
   r) Rename remote
   c) Copy remote
   s) Set configuration password
   q) Quit config
   n/r/c/s/q>

Start to create a new remote typing ``n`` on the prompt followed by
enter. That will show a new prompt below with ``name>``:

::

   2019/02/07 13:30:33 NOTICE: Config file "/home/samuel/.config/rclone/rclone.conf" not found - using defaults
   No remotes found - make a new one
   n) New remote
   r) Rename remote
   c) Copy remote
   s) Set configuration password
   q) Quit config
   n/r/c/s/q>
   name>

Choose a name for your remote. In this example is used ``remote`` as
follows:

::

   [snip]
   q) Quit config
   n/r/c/s/q>
   name> remote

After click enter and accepting the name provided, it passes to remote
storage configuration:

::

   name> remote
   Type of storage to configure.
   Choose a number from below, or type in your own value
   [snip]
   24 / Webdav
      \ "webdav"
   [snip]
   Storage> webdav

Type ``webdav`` as showing above and click enter.
::

   Storage> webdav
   ** See help for webdav backend at: https://rclone.org/webdav/ **

   URL of http host to connect to
   Enter a string value. Press Enter for the default ("").
   Choose a number from below, or type in your own value
    1 / Connect to example.com
      \ "https://example.com"
   url>

Then place webdav url for Nextcloud that is just like this (replace
nextcloud.example.com with the fqdn of your provider):

::

   url> https://nextcloud.example.com/remote.php/webdav/
   Name of the Webdav site/service/software you are using
   Enter a string value. Press Enter for the default ("").
   Choose a number from below, or type in your own value
    1 / Nextcloud
      \ "nextcloud"
    2 / Owncloud
      \ "owncloud"
    3 / Sharepoint
      \ "sharepoint"
    4 / Other site/service or software
      \ "other"
   vendor>

Now is time to select the provider type, that for this case will be
Nextcloud. For that type ``1`` and click enter:

::

   vendor> 1
   User name
   Enter a string value. Press Enter for the default ("").
   user>

In this step prompt is asking for the user name. For the example the
user name is ``testuser1``:

::

   user> testuser1
   Password.
   y) Yes type in my own password
   g) Generate random password
   n) No leave this optional password blank
   y/g/n>

Since in this example the authentication method is single sign on type
``y`` and click enter. Then enter password and confirm it:

::

   y/g/n> y
   Enter the password:
   password:
   Confirm the password:
   password:
   Bearer token instead of user/pass (eg a Macaroon)
   Enter a string value. Press Enter for the default ("").
   bearer_token>

Now prompt waits for a bearer token. This is only necessary if using
another authentication method based on session tokens. More details
about `rclone authorization`_ can be reviewed in official documentation.

The documentation to use rclone with OIDC authentication provider will
be available in other user guide. It needs to use oidc-agent as
mentioned in `rclone github issue`_.

::

   bearer_token>
   Remote config
   --------------------
   [remote]
   type = webdav
   url = https://netcloud.example.com/remote.php/webdav/
   vendor = nextcloud
   user = testuser1
   pass = *** ENCRYPTED ***
   --------------------
   y) Yes this is OK
   e) Edit this remote
   d) Delete this remote
   y/e/d>

Since in this guide is used a password just press enter. Then it appears
the resulting configuration that, after checking everything is correct,
type ``y`` and enter.

In the last step just type ``q`` and press enter. You can see that there
is already registered the remote provider in the list that appeared in
the prompt as the configuration result:

::

   Current remotes:

   Name                 Type
   ====                 ====
   remote               webdav

   e) Edit existing remote
   n) New remote
   d) Delete remote
   r) Rename remote
   c) Copy remote
   s) Set configuration password
   q) Quit config
   e/n/d/r/c/s/q>
Usage
-----

A more detailed version of available commands and their documentation is
available at `rclone docs`_.

List directories in top level of your WebDAV:

``rclone lsd remote:``

For testing proposes, if there is any problem validating the certificate
add the flag ``--no-check-certificate`` after rclone. For example, using
the previous command it becomes:

``rclone --no-check-certificate lsd remote:``

List all the files in your WebDAV:

``rclone ls remote:``

To copy a local directory to an WebDAV directory called backup

``rclone copy /home/source remote:backup``

Nextcloud limitations
---------------------

As stated for current date release, it stills miss support for streaming
of files (``rcat``). However it `may be fixed`_ in the future.

.. _Rclone: https://github.com/ncw/rclone
.. _rclone install documentation: https://rclone.org/install/
.. _rclone webdav documentation: https://rclone.org/webdav/
.. _rclone authorization: https://rclone.org/commands/rclone_authorize/
.. _rclone github issue: https://github.com/ncw/rclone/issues/2380
.. _rclone docs: https://rclone.org/docs/
.. _may be fixed: https://github.com/nextcloud/nextcloud-snap/issues/365