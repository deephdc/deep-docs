# Installing nova-lxd with Juju

This is instruction of deploying Openstack with nova-lxd on single machine (All-in-One) for testing and deployment. Tested on Ubuntu Xenial and Bionic. Whole process of installation would take around 2h. Be careful with the LXD setting, leave default values when possible to avoid later problems. ("none" to IPv6, "lxdbr0" for bridge and "default" for name of LXD storage.

## Installation
1. Create/Install a new machine with Ubuntu 16/18.04 and adequate performance (16GB RAM needed)

2. Optional: Update all packages (as root)
```
    # apt update && apt dist-upgrade -y && apt autoremove -y
```
3. Install LXD (version 3.x required):
```
    # sudo snap install lxd
```
4. Configure LXD:
```
    # /snap/bin/lxd init
```
Say none to IPv6. If there a empty disk volume (block storage) attached, use it as existing block storage for LXD without mounting/formatting, otherwise use dir (with name of the storage as “default” for both cases). For other options, leave default value.

5. Optional: Add current user to LXD group (for using LXD as user later)
```
    # sudo usermod -a -G lxd $USER
    # newgrp lxd
```
6. Optional: remove old LXD version to avoid conflict
```
    # sudo /snap/bin/lxd.migrate
```
7. Optional: increase the limit of number open files (only needed for larger tests). See https://lxd.readthedocs.io/en/latest/production-setup/
```
    # echo fs.inotify.max_queued_events=1048576 | sudo tee -a /etc/sysctl.conf
    # echo fs.inotify.max_user_instances=1048576 | sudo tee -a /etc/sysctl.conf
    # echo fs.inotify.max_user_watches=1048576 | sudo tee -a /etc/sysctl.conf
    # echo vm.max_map_count=262144 | sudo tee -a /etc/sysctl.conf
    # echo vm.swappiness=1 | sudo tee -a /etc/sysctl.conf
    # sudo sysctl -p
```
Also update default profile for improving network connnection
```
    # lxc profile device set default eth0 mtu 9000
```
8. Optional: Check configuration (as user), if your configuration is correct
```
    # /snap/bin/lxc storage list
    # /snap/bin/lxc storage show default
    # /snap/bin/lxc network show lxdbr0
    # /snap/bin/lxc profile show default
```
9. Optional: Run a test container in LXD (as user), if LXD work correctly
```
    # lxc launch ubuntu:16.04 u1
    # lxc exec u1 ping www.ubuntu.com
    # lxc stop u1
    # lxc delete u1
```
10. Install juju:
```
    # sudo snap install juju --classic
```
11. Install conjure-up
```
    # sudo snap install conjure-up --classic
```
12. Optional: Start tmux terminal (to avoid unwanted termination in the case of network disruption)
```
    # tmux
```
13. Start conjure-up (in tmux terminal if tmux is used):
```
    # conjure-up
```
Choose Openstack with Nova-LXD, localhost. Other options can be left as default (lxdbr0 network bridge, “default” storage pool, ~/.ssh for SSH key), then deploy

14. Go to have a coffee for about 45-90 min (depending on performance of host machine). The installation with deploy nova-lxd with relevant services (keystone, glance, cinder, horizon). Remember the IP address of horizon dashboard from the output (e.g. http://10.110.236.154/horizon).

15. Use IP forwarding to get access to dashboard from outside by executing the following command on the host where whole openstack with nova-lxd is installed:
```
    # sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to “IP_of_horizon:80”
```
where IP_of_horizon is the IP address of the dashboard that is given when the installation finishes (in format of 10.x.x.x, e.g. 10.110.236.154)

16. Log into the dashboard (http://ip_address_of_host:8080/horizon), with “admin_domain” domain, “admin” user and “openstack” password, where ip_address_of_host is the IP of host machine, where whole Openstack is installed (e.g. http://147.213.76.100:8080/horizon)

17. Create a new VM (Launch instance) in Compute->Instances. Do not create new volume (choose NO for new volume), and add only internal network.

18. In Network->Security group, add new ingress rules for ICMP (ping) and TCP port 22 (SSH) to default security group.

19. Allocate a new floating IP from Network -> Floating IPs and assign to the VM.

20. From host machine, try ssh to floating IP of VM
```
    # ssh ubuntu@floating_ip
```
## Notes
* Ceph and Cinder are installed together with other Openstack services, however attaching block storage does not work. According to https://lists.gt.net/openstack/dev/64776, it should require some additional work.
* Although Ubuntu Bionic with LXD 3.0 was used as base OS on the host, in LXD containers are Ubuntu Xenial with LXD 2.0
