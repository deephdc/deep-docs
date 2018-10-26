
# Overview

This document is a step-by-step guide of OpenStack with nova-lxd deployment via DevStack on a single machine (All-in-One). The guide was tested on a host running Xenial or Bionic. The host has pre-installed following libraries:

* Python 2.7
* PyLXD 2.2.7

# Installation steps

1. Create a new host host running Xenial or Bionic with Python 2.7

2. Install the newest version of a library PyLXD 2.2.7

   1. Install pip
   ```bash
   $ sudo apt update
   $ sudo apt install python-pip
   ```
   
   2. Set up your environment variable ```bash LC_ALL```
   ```bash
   $ export LC_ALL="en_US.UTF-8"
   $ export LC_CTYPE="en_US.UTF-8"
   $ sudo dpkg-reconfigure locales
   ```
   
   3. Download and install the library PyLXD
   ```bash
   $ git clone https://github.com/lxc/pylxd.git
   $ cd pylxd/
   $ sudo pip install .
   ```
3. *Optional step:* Install ZFS
```bash
$ sudo apt install lxd zfsutils-linux
```

4. *Optional step:* If you need to install a different LXD/LXC version, execute following steps:

   1. Unstall LXD and LXC (delete configuration and data files of LXD/LXC and it's dependencies)
   ```bash
   $ sudo apt-get purge --auto-remove lxd lxc
   ```
   
   2. Install the wanted version of LXD/LXC:
      * LXD/LXC 3.0.1 on a host runing Xenial
      ```bash
      $ sudo apt-get purge --auto-remove lxd lxc
      ```
      * the newest version
      ```bash
      $ sudo snap install lxd
      ```
      if you wish to install LXD 3.0 and then only get bugfixes and security updates. If running staging systems, you may want to run those on the candidate channels, using ```bash--channel=candidate``` and ```bash--channel=3.0/candidate``` respectively.
      ```bash
      $ sudo snap install lxd --channel=3.0
      ```
      
5. Configure LXD:
   1. In order to use LXD, the system user must be a member of the 'lxd' user group.
      ```bash
      $ sudo adduser martin lxd
      $ newgrp lxd
      $ groups
      ```
      
   2. LXD initialisation
      ```bash
      $ sudo lxd init
      ```
      The session below (LXD 3.0.1 with a zfs storage backend) is what was used to write this guide. Note that pressing Enter (a null answer) will accept the default answer (provided in square brackets).
      ```bash
      Would you like to use LXD clustering? (yes/no) [default=no]: 
      Do you want to configure a new storage pool? (yes/no) [default=yes]: 
      Name of the new storage pool [default=default]: lxd
      Name of the storage backend to use (btrfs, dir, lvm, zfs) [default=zfs]: 
      Create a new ZFS pool? (yes/no) [default=yes]: 
      Would you like to use an existing block device? (yes/no) [default=no]: 
      Size in GB of the new loop device (1GB minimum) [default=15GB]:
      Would you like to connect to a MAAS server? (yes/no) [default=no]: 
      Would you like to create a new local network bridge? (yes/no) [default=yes]: 
      What should the new bridge be called? [default=lxdbr0]: 
      What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]:     
      What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
      Would you like LXD to be available over the network? (yes/no) [default=no]: 
      Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
      Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:
      ```
6. *Optional step:* Remove old LXD version to avoid conflict
```bash
$ sudo /snap/bin/lxd.migrate
```

7. *Optional step:* Increase the limit of number open files (only needed for larger tests). See https://lxd.readthedocs.io/en/latest/production-setup/

8. *Optional step:* Check configuration (as user), if your configuration is correct
```bash
$ sudo /snap/bin/lxc storage list
$ sudo /snap/bin/lxc storage show default
$ sudo /snap/bin/lxc network show lxdbr0
$ sudo /snap/bin/lxc profile show default
```

9.*Optional step:* Run a test container in LXD (as user), if LXD work correctly
```bash
$ sudo lxc launch ubuntu:16.04 u1
$ sudo lxc exec u1 ping www.ubuntu.com
$ sudo lxc stop u1
$ sudo lxc delete u1
```

10. Create Stack user and add it to the 'lxd' user group
```bash
$ sudo useradd -s /bin/bash -d /opt/stack -m stack
$ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
$ sudo adduser martin lxd
$ newgrp lxd
$ groups
$ sudo su - stack
```

11. Download OpenStack installation scripts from DevStack repository
```bash
$ git clone https://git.openstack.org/openstack-dev/devstack
$ cd devstack
```

12. Create a **local.conf** file (a branch of nova-lxd plugin is ```bash stable/rocky```) as follows:
```bash
[[local|localrc]]

HOST_IP=127.0.0.1 # set this to your IP
FLAT_INTERFACE=ens2 # change this to your eth0

ADMIN_PASSWORD=secret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
SERVICE_TOKEN=$ADMIN_PASSWORD


# run the services you want to use
ENABLED_SERVICES=rabbit,mysql,key
ENABLED_SERVICES+=,g-api,g-reg
ENABLED_SERVICES+=,n-cpu,n-api,n-crt,n-obj,n-cond,n-sch,n-novnc,n-cauth,placement-api,placement-client
ENABLED_SERVICES+=,neutron,q-svc,q-agt,q-dhcp,q-meta,q-l3
ENABLED_SERVICES+=,cinder,c-sch,c-api,c-vol
ENABLED_SERVICES+=,horizon

# disabled services
disable_service n-net

# enable nova-lxd
enable_plugin nova-lxd https://git.openstack.org/openstack/nova-lxd stable/rocky
```
13. Start installation (it will take a 15 - 20 minutes, largely depending on the speed of your internet connection. Many git trees and packages will be installed during this process.)
```bash
$ ./stack.sh
```
