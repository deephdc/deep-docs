
# Overview

This document is a step-by-step guide of OpenStack with nova-lxd deployment via DevStack on a single machine (All-in-One). The guide was tested on a host running Xenial or Bionic. The host has pre-installed following libraries:

* Python 2.7
* PyLXD 2.2.7

# Installation steps

1. Create a new host running Xenial or Bionic with Python 2.7

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

4. *Optional step:* If you need to install a different LXD/LXC version, execute the following steps:

   1. Uninstall LXD and LXC (delete configuration and data files of LXD/LXC and it's dependencies)
   ```bash
   $ sudo apt-get purge --auto-remove lxd lxc
   ```
   
   2. Install the wanted version of LXD/LXC:
      * LXD/LXC 3.0.1 on a host running Xenial
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

8. *Optional step:* Check configuration (as a user), if your configuration is correct
```bash
$ sudo /snap/bin/lxc storage list
$ sudo /snap/bin/lxc storage show default
$ sudo /snap/bin/lxc network show lxdbr0
$ sudo /snap/bin/lxc profile show default
```

9.*Optional step:* Run a test container in LXD (as a user), if LXD work correctly
```bash
$ sudo lxc launch ubuntu:16.04 u1
$ sudo lxc exec u1 ping www.ubuntu.com
$ sudo lxc stop u1
$ sudo lxc delete u1
```

10. Create a user "Stack" and add it to the 'lxd' user group
```bash
$ sudo useradd -s /bin/bash -d /opt/stack -m stack
$ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
$ sudo usermod -G lxd -a stack
$ sudo su - stack
```

11. Download OpenStack installation scripts from DevStack repository
```bash
$ git clone https://git.openstack.org/openstack-dev/devstack
$ cd devstack
```

12. Create a **local.conf** file (a branch of a nova-lxd plugin is ```bash stable/rocky```) as follows:
```bash
[[local|localrc]]

HOST_IP=127.0.0.1 # set this to your IP
FLAT_INTERFACE=ens2 # change this to your eth0

ADMIN_PASSWORD=devstack
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

13. Start installation of an OpenStack environment (it will take a 15 - 20 minutes, largely depending on the speed of your internet connection. Many git trees and packages will be installed during this process.)
```bash
$ ./stack.sh
```

14. Configuration of ```bash nova-compute```: In order for a lxd storage pool to be recognized in nova, the ```bash/etc/nova/nova-cpu.conf``` file needs to have ```bash [lxd] section``` containing the following lines:
```bash
[lxd]
allow_live_migration = True
pool = {{ storage_pool }}
```
Restart nova-compute service:
```bash
$ systemctl restart devstack@n-cpu.service
$ systemctl status devstack@n-cpu.service
```

15. *Optional step:* if your OpenStack installation is set to incorect repository, execute the following commands (adding a Rocky cloud archive):
```bash
$ sudo rm /etc/apt/sources.list.d/{{Bad_archive}}.list
$ sudo add-apt-repository cloud-archive:rocky
```

16. *Optional step:*  Use IP forwarding to get access to the dashboard from outside by executing the following command on the host where the whole OpenStack environment with nova-lxd is installed:
```bash
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to “{{IP_of_horizon}}:80”
```
where IP_of_horizon is the IP address of the dashboard that is given when the installation finishes (in the format of 10.x.x.x, e.g. 10.110.236.154)

17. Log into the dashboard (http://{{IP_address_of_host}}:8080/horizon), with “admin_domain” domain, “admin” user and “devstack” password, where ip_address_of_host is the IP of the host machine, where the whole OpenStack environment is installed (e.g. http://147.213.76.100:8080/horizon)

18. Create a new VM (Launch instance) in Compute->Instances. **Do not create a new volume (choose NO for new volume)**, and add only internal network.

19. In Network->Security group, add new ingress rules for ICMP (ping) and TCP port 22 (SSH) to default security group.

20. Allocate a new floating IP from Network -> Floating IPs and assign to the VM.

21. From host machine, try ssh to floating IP of VM
```bash
$ ssh ubuntu@{{Floating_ip}}
```

# Handy commands:

# Notes:
* CEPH volume is still not attachable to VM by defaults, some additional work required.

# References
1. https://discuss.linuxcontainers.org/t/lxd-3-0-0-has-been-released/1491
2. https://docs.jujucharms.com/devel/en/tut-lxd
3. https://docs.openstack.org/devstack/latest/
4. https://github.com/openstack/nova-lxd/blob/master/devstack/local.conf.sample
5. https://wiki.ubuntu.com/OpenStack/CloudArchive
