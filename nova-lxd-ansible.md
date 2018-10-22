
# Overview

This document provide step by step deployment of Openstack site with nova-lxd via Openstack Ansible. That would allows site admins to avoid obstacles and pitfalls during deployment and create a Openstack site with nova-lxd for testing and development in a short time without studying extensive documentation of Openstack Ansible. 

This documentation also show that nova-lxd is supported (although not perfectly, see steps 10-11 for fixing LXD configuration) in mainstream Openstack deployment tool beside Ubuntu-specific Juju charms. Ubuntu 18.04 and LXD 3.0 are also supported (instead of 16.04 / LXD 2.0 used in Juju).

# Comparison between Openstack Ansible and Juju/conjure-up

Juju is Ubuntu- specific, and using Ubuntu distro packages for installing Openstack. Openstack Ansible is distro-neutral and by default it uses source code from github for Openstack installation (configurable for distro packages, however). For testing and development, installation via source code has advantages using latest codes, however, for production sites, distro packages are more stable, especially when Ubuntu offers up to 5-year software maintenance in comparison with 18-month from Openstack.

Other differences: Juju uses LXD containers for all Openstack services and has better integration with Ubuntu ecosystem (MAAS, LXD). Openstack Ansible use LXC containers for services on master (like nova-api, keystone, glance, cinder) or directly baremetal for services on workers (like nova-compute, cinder-volume).

Openstack Ansible offers wide possibilities of customization, e.g. method of installation (distro package vs source), based Linux distro (RedHat/CenOS, openSUSE and Ubuntu), selection of services to be installed. Beside Openstack installation, it also does many other tasks for security hardening and high availability (e.g. haproxy/Galera). As the result, it is more complex and need more time for deployment. (Un)fortunately, Openstack Ansible has very extensive documentation that is useful but may require time to study. See [1] and [2] from references for more information.

# Installing a All-in-One Openstack site with nova-lxd via Openstack Ansible

This installation takes rather long time (totally around ~2h according to disk performance, network connection and list of installed services). Fast disk and network connection is strongly recommended because of intensive disk operation during installation and large amount of files downloaded from repositories

1. Install Ubuntu Bionic/ Create VM with vanilla Ubuntu Bionic (at least 16GB RAM, 80GB disk). So far do not init/configure the LXD service, Ansible script may report error like “storage already exist”.

2. Optional: Update all packages, then reboot

3. Cloning Openstack-Ansible
```
    # git clone https://git.openstack.org/openstack/openstack-ansible \
        /opt/openstack-ansible
    # cd /opt/openstack-ansible
```

4. Optional: Choose version/branch if needed
```
    # git tag -l
    # git checkout master
    # git describe --abbrev=0 --tags
```

5. Bootstrap Ansible (about 6min in the test)
```
    # scripts/bootstrap-ansible.sh
```

6. Default scenario is without CEPH. Select CEPH scenario if you want and bootstrap AOI  (about 6min in the test)
```
    # export SCENARIO='ceph'
    # scripts/bootstrap-aio.sh
```

7. Setting hypervisor to LXD: 

Edit file “/etc/openstack_deploy/user_variables.yml”. Add a line “nova_virt_type: lxd” into it.
```
     nova_virt_type: lxd
```

8. Examine list of services to be installed in “/etc/openstack_deploy/conf.d/” Copy more services from "/opt/openstack-ansible/etc/openstack_deploy/conf.d/" to "/etc/openstack_deploy/conf.d/" as needed. Remember to change the filename extension from aio to yml. For example, CEPH scenario is without Horizon and we want to add it.
```
    # cp /opt/openstack-ansible/etc/openstack_deploy/conf.d/horizon.yml.aio \
        /etc/openstack_deploy/conf.d/horizon.yml
```

9. Start the core installation (in 3 big steps). It takes long time (about 15min + 30min + 30min in the test), so you may want to use tmux or screen terminal if using SSH on unreliable network.
```
    # cd /opt/openstack-ansible/playbooks
    # openstack-ansible setup-hosts.yml
    # openstack-ansible setup-infrastructure.yml
    # openstack-ansible setup-openstack.yml
```

During execution of the first command “openstack-ansible setup-hosts.yml”, it is possible that you have timeout error during LXC caching if you have slow disk and/or network connection. In this case, please increase the value of “lxc_cache_prep_timeout” in “/etc/ansible/roles/lxc_hosts/defaults/main.yml” and re-execute the command.

The test “TASK [os_tempest : Execute tempest tests]” in the last command "openstack-ansible setup-openstack.yml" will fail. Ignore it and continue to next steps.

10. The Ansible script create a storage with the name “default” and driver “dir” for LXD, however, it does not work (btrfs or zfs required). Now create a new storage for LXD with btrfs with other name, e.g. “nova”. A simplest way is to run “lxd init” and configure storage via it. Do not touch networks/bridges or other configuration.
```
 # lxd init
```

11. Setting LXD storage for nova:
Edit file /etc/nova/nova.conf, add the following section into the file. For the pool option, use the name of storage created in the previous step, e.g. "nova"
```
    [lxd]
    allow_live_migration = True
    pool ="nova"
```
  Restart nova-compute service:
```
    # systemctl restart nova-compute
    # systemctl status nova-compute
```

12. Installation is now completed, however, some post-installation configurations are needed before starting the first VM. Refer to [3] for more information. The post-installation configuration can be done via CLI or via Horizon dashboard. The following steps show the configuration via Horizon.

13. Get the IP address of load balancer from "external_lb_vip_address" in "/etc/openstack_deploy/openstack_user_config.yml" file. Use i"ptables" for IP forwarding to get the dashboard from your PC, e.g. :
```
    # iptables -t nat -A PREROUTING -p tcp -m tcp --dport 8080 -j DNAT --to-destination external_lb_vip_address:443
```
  Also remember do open firewall for the chosen port (8080 in the example).

14. Open Horizon dashboard in your browser as https://ip_address_of_host:8080. Log in using “admin” user and password stored in "keystone_auth_admin_password" item in "/etc/openstack_deploy/user_secrets.yml" file.

15. In Horizon, do the steps for configuring Openstack network: create a new private network, create a private subnet for the private network, create a router to connect private subnet to existing public network, open ports in security groups (e.g. port 22 for SSH). Also import SSH public key from "~/.ssh/id_rsa.pub" on the host.

16. Creating images does not work in default Horizon installed by Ansible, you must change Horizon setting or use command-line to create image. Use "lxc-attach" command to get into "aio1_utility_container_xxxxxx" container, load Openstack credential, download a LXD image from repository and add it to glance:
```
    # lxc-ls
    # lxc-attach aio1_utility_container_xxxxxxxx
    # cd
    # source openrc
    # wget http://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-root.tar.gz
    # glance image-create --name xenial-lxd --disk-format raw --container-format bare --file xenial-server-cloudimg-amd64-root.tar.gz
```
17. Return to Horizon, create a new VM with the newly added "xenial-lxd" image. Remember to no create a new volume. Allocating a floating IP and assign it to the VM. From command line on the host, try to connect to the VM via ssh.

18. Success.

# Notes:
* CEPH volume is still not attachable to VM by defaults, some additional work required.

# References
1. https://docs.openstack.org/openstack-ansible/latest/user/aio/quickstart.html
2. https://docs.openstack.org/project-deploy-guide/openstack-ansible/latest/
3. https://docs.openstack.org/openstack-ansible/latest/admin/openstack-firstrun.html
