# Overview

As the nova-lxd plugin in OpenStack is still experimental, we need to deploy and test its current status to find out the working configuration before implementing new features

## Testing of nova-lxd with different software configurations
OpenStack with nova-lxd has been deployed by different methods: OpenStack DevStack, JuJu, and OpenStack Ansible. Various combinations with OpenStack/LXD and base OS version have been tested to find out the working configuration of OpenStack nova-lxd for production. 

Since nova-lxd plugin and OpenStack DevStack (deployment of OpenStack Cloud by Python scripts) have extremely shallow documentation, the first step was finding out the best starting configuration for development. We reached the following results (without advanced post-configuration efforts):

**Host OS** | **OpenStack (DevStack)version** | **LXD/LXC version** | **Storage type** | **Nova-lxd recognized** | **Notes**
----------- | --------------------------- | --------------- | ------------ | ------------------- | -----
Ubuntu 16.04.5 LTS | Queens | 2.0.11 | Dir | Yes | VM creation successful, Volume attachment error.
Ubuntu 16.04.5 LTS | Queens | 2.0.11 | zfs | Yes | VM creation successful, Volume attachment error.
Ubuntu 16.04.5 LTS | Queens | 3.0.1 | zfs | No | Couldn't find the lxd storage from its zpool.
Ubuntu 16.04.5 LTS | Queens | 3.0.1 | btrfs | Yes | VM creation error.
Ubuntu 16.04.5 LTS | Queens | 3.0.1 | Dir | Yes | VM creation error.
Ubuntu 16.04.5 LTS | Queens | 3.0.1 | lvm | Yes | VM creation error.
Ubuntu 16.04.5 LTS | Rocky | 2.0.11 | Dir | Yes | VM creation successful, Volume attachment error.
Ubuntu 16.04.5 LTS | Rocky | 2.0.11 | zfs | No | Error during deployment.
Ubuntu 16.04.5 LTS | Rocky | 3.0.1 | zfs | No | Couldn't find the lxd storage from its zpool.
Ubuntu 16.04.5 LTS | Rocky | 3.0.1 | btrfs | Yes | VM creation error.
Ubuntu 18.04.1 LTS | Rocky | 3.0.1 | zfs | No | Couldn't find the lxd storage from its zpool.
Ubuntu 18.04.1 LTS | Rocky | 3.0.1 | btrfs | Yes | VM creation error.
Ubuntu LTS (both tested versions) | OpenStack devstack (both tested versions) | The last version (from its Snap repository) | N/A | No | LXD/LXC isn't recognized by OpenStack devstack installation script stack.sh or nova-lxd plugin

The hosts have the following static parameters:

**Tool/Library** | **Version** | **Notes**
----------- | --------------------------- | ---------------
Python | 2.7 | It is possible to use Python 3.X also.
pip | 9.0.3 | The tool was installed by OpenStack devstack script stack.sh
PyLXD | 2.2.7 | The library has to be pre-installed on a host.
Nova-lxd plugin | According to an OpenStack version, there are following options: <ul><li>17.0.1 (stable/queens)</li><li>18.0.0 (stable/rocky), 18.0.0.0rc2.dev1 (master)</li></ul> | The master branch was used for hosts with storage backend zfs running LXD/LXC 3.0.1

There are two other deployment options for OpenStack alongside DevStack which are the following: Juju, and Ansible. The main advantage of these two approaches is automated deployment (with a post-configuration) of an OpenStack Cloud environment. The main differences between the deployment approaches are the configuration of an OpenStack environment, and LXD/LXC support. Juju supports LXD/LXC 2.0.11. It creates inside a Xenial/Bionic host another virtual layer by an LXD daemon from the host, and so Xenial with LXD/LXC 2.0.11 is installed in a container. All in all, it doesn't support higher versions of LXD/LXC which supports GPUs.

On the other hand, Ansible supports LXD/LXC 3.0.1. However, the situation about a nova-lxd plugin integration is the same, but the configuration of an OpenStack environment is different. According to Alex Kavanagh (one of the maintainers for the nova-lxd plugin) suggestions, the plugin has to be post-configured within a nova-compute configuration that is not documented in any official sources. The configuration file has to contain the following lines:

```bash
[DEFAULT]
compute_driver = nova_lxd.nova.virt.lxd.LXDDriver

[lxd]
allow_live_migration = True
pool = {{ storage_pool }}
```

A problem with the post-configuration is that it has to be performed in a different way within an OpenStack Cloud environment deployed by DevStack. The environment has a dedicated configuration file nova-cpu.conf. The other approaches create the standard deployment of an OpenStack Cloud environment, and so the configuration of the nova-lxd plugin is performed by editing of a nova.conf file.

## Working configuration
According to the performed test mentioned above, we chose OpenStack Ansible repository for deployment of an OpenStack Cloud. The main reason is that it deploys a standard OpenStack Cloud with LXD/LXC 3.0.1 which supports GPUs.
