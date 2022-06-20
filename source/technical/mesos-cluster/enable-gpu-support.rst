.. highlight:: console

Introduction
============

Mesos 1.0.0 added first-class support for Nvidia GPUs. The minimum
required Nvidia driver version is \ ``340.29``

Enabling GPU support in a Mesos cluster is really straightforward (as
stated in the official project documentation and as documented in this
page). It consists in the following steps:

1. configuring the agent nodes in order to expose the available gpus as
   resources to be advertised to the master nodes;
2. enabling the framework GPU_RESOURCES capability so that the master
   includes the GPUs in the resource offers sent to the frameworks.  

Mesos exposes GPUs as a simple \ ``SCALAR``\  resource in the same way
it always has for CPUs, memory, and disk.

An important remark is that currently the GPU support is available for
the Mesos containerizer and not for the Docker containerizer. Anyway the
Mesos containerizer is now able to run docker images natively through
the Universal Container Runtime (UCR). 

The following limitations can, on the other hand, have impacts on the
deployment of Long-Running services (Marathon) requiring GPUs:

-  The UCR does not support the following: runtime privileges, Docker
   options, force pull, named ports, numbered ports, bridge networking,
   port mapping.

It is important to remember that the task definition must be properly
written in order to specify the right containerizer (type=MESOS). 

For Marathon:

.. code::

   {
     "id": "test",
     "cpus": 2,
     "mem": 2048,
     [...]
     "container": {
       "type": "MESOS",
       "docker": {
         "image": "tensorflow/tensorflow"
       }
     }
   }

See
also https://mesosphere.github.io/marathon/docs/native-docker.html#provisioning-containers-with-the-ucr

For Chronos:

.. code::

   {
     "name": "test-gpu",
     "command": "",
     "cpus": 1,
     "mem": 4096,
     [...]
     "container": {
       "type": "MESOS",
       "image": "tensorflow/tensorflow"
     },
     "schedule": "..."
   }

 

The GPU support is fully implemented and officially documented in Mesos
and Marathon Framework whereas Chronos Framework does not support GPU
resources yet. Anyway there is a pull request (still open) that seems in
good shape and we have decided to give it a try.

Testbed Setup
=============

Nodes characteristics
---------------------

+----------+-----------------------------------------------------------+
| Node     | Description                                               |
+==========+===========================================================+
| Mesos    | VM 4vCPU, 16GB RAM SO: Ubuntu 16.04                       |
| Master   |                                                           |
+----------+-----------------------------------------------------------+
| Mesos    | baremetal 40 CPUs, 250GB RAM 82:00.0 3D controller:       |
| Slave    | NVIDIA Corporation GK110BGL [Tesla K40m] (rev a1) Model   |
|          | name: Intel(R) Xeon(R) CPU E5-2650L v2 @ 1.70GHz SO:      |
|          | Ubuntu 16.04                                              |
+----------+-----------------------------------------------------------+

Tested Components Versions
--------------------------

+--------------+-------------------+---------------------------------+
| Component    | Version           | Notes                           |
+==============+===================+=================================+
| Mesos Master | 1.5.0             | docker container                |
|              |                   | ind                             |
|              |                   | igodatacloud/mesos-master:1.5.0 |
+--------------+-------------------+---------------------------------+
| Mesos Slave  | 1.5.0             | package mesos from mesosphere   |
|              |                   | repo                            |
+--------------+-------------------+---------------------------------+
| Chronos      | tag 3.0.2 + patch |                                 |
+--------------+-------------------+---------------------------------+
| Marathon     | 1.5.6             | docker container                |
|              |                   | indigodatacloud/marathon:1.5.6  |
+--------------+-------------------+---------------------------------+

Prepare the agent (slave) node
==============================

Download the driver repo
from http://www.nvidia.com/Download/index.aspx?lang=en-us choosing the
proper version.

Install the downloaded .deb file (repo), install the driver and reboot:

.. code::

   dpkg -i nvidia-diag-driver-local-repo-ubuntu1604-390.30_1.0-1_amd64.deb
   apt-key add /var/nvidia-diag-driver-local-repo-390.30/7fa2af80.pub
   apt-get update
   apt-get install nvidia-390
   reboot

Alternatively you can enable the graphics-drivers PPA. Currently, it
supports Ubuntu 18.04 LTS, 17.10, 17.04, 16.04 LTS, and 14.04 LTS
operating systems (still under testing phase):

.. code::

   add-apt-repository ppa:graphics-drivers/ppa
   apt update
   apt install nvidia-390

Verify the nvidia-driver installation
-------------------------------------

Launch the command:

::

   nvidia-smi

Output:

.. code::

   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 390.30                 Driver Version: 390.30                    |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
   |===============================+======================+======================|
   |   0  Tesla K40m          Off  | 00000000:82:00.0 Off |                    0 |
   | N/A   21C    P8    18W / 235W |      0MiB / 11441MiB |      0%      Default |
   +-------------------------------+----------------------+----------------------+

   +-----------------------------------------------------------------------------+
   | Processes:                                                       GPU Memory |
   |  GPU       PID   Type   Process name                             Usage      |
   |=============================================================================|
   |  No running processes found                                                 |
   +-----------------------------------------------------------------------------+

Mesos slave configuration
-------------------------

.. code::

   export MESOS_MASTER="zk://<master>:2181/mesos"
   export MESOS_ZK="zk://<master>:2181/mesos"
   export MESOS_EXECUTOR_REGISTRATION_TIMEOUT="10mins"
   export MESOS_CONTAINERIZERS="mesos,docker"
   export MESOS_LOG_DIR="/var/log/mesos"
   export MESOS_IP="<agent-ip>"
   export MESOS_WORK_DIR="/var/lib/mesos"
   export MESOS_HOSTNAME="<agent-hostname>"
   export MESOS_ISOLATION="docker/runtime,filesystem/linux,cgroups/devices,gpu/nvidia"
   export MESOS_IMAGE_PROVIDERS="docker"

(Re)start the mesos agent. In the log you will see the available GPU as
resource offered by the agent node:

Feb 26 23:10:07 hpc-09-02 mesos-slave[1713]: I0226 23:10:07.155503  1745
slave.cpp:533] Agent resources: gpus(*):1; cpus(*):40; mem(*):256904;
disk(*):3489713; ports(*):[31000-32000]

Testing GPU support in Mesos
----------------------------

Verify that Mesos is able to launch a task consuming GPUs:

.. code::

   mesos-execute --master=mesos-m0.recas.ba.infn.it:5050 --name=gpu-test  --docker_image=nvidia/cuda       --command="nvidia-smi"       --framework_capabilities="GPU_RESOURCES"       --resources="gpus:1"
   I0305 15:22:38.346174  4443 scheduler.cpp:188] Version: 1.5.0
   I0305 15:22:38.349104  4459 scheduler.cpp:311] Using default 'basic' HTTP authenticatee
   I0305 15:22:38.349442  4462 scheduler.cpp:494] New master detected at master@172.20.0.38:5050
   Subscribed with ID 6faa9a75-d48b-4dc6-96ee-73c35997706b-0017
   Submitted task 'gpu-test' to agent 'd33d527c-8d1f-4e53-b65d-e2b2c67c0889-S2'
   Received status update TASK_STARTING for task 'gpu-test'
     source: SOURCE_EXECUTOR
   Received status update TASK_RUNNING for task 'gpu-test'
     source: SOURCE_EXECUTOR
   Received status update TASK_FINISHED for task 'gpu-test'
     message: 'Command exited with status 0'
     source: SOURCE_EXECUTOR

Look into the task sandbox. The stdout should report the following:

.. code::

   Marked '/' as rslave
   Prepared mount '{"flags":20480,"source":"\/var\/lib\/mesos\/slaves\/d33d527c-8d1f-4e53-b65d-e2b2c67c0889-S2\/frameworks\/6faa9a75-d48b-4dc6-96ee-73c35997706b-0017\/executors\/gpu-test\/runs\/5ebbfaf3-3b8b-4c32-9337-740a85feef75","target":"\/var\/lib\/mesos\/provisioner\/containers\/5ebbfaf3-3b8b-4c32-9337-740a85feef75\/backends\/overlay\/rootfses\/e56d62ea-4334-4582-a820-2b9406e2b7f8\/mnt\/mesos\/sandbox"}'
   Prepared mount '{"flags":20481,"source":"\/var\/run\/mesos\/isolators\/gpu\/nvidia_390.30","target":"\/var\/lib\/mesos\/provisioner\/containers\/5ebbfaf3-3b8b-4c32-9337-740a85feef75\/backends\/overlay\/rootfses\/e56d62ea-4334-4582-a820-2b9406e2b7f8\/usr\/local\/nvidia"}'
   Prepared mount '{"flags":20513,"target":"\/var\/lib\/mesos\/provisioner\/containers\/5ebbfaf3-3b8b-4c32-9337-740a85feef75\/backends\/overlay\/rootfses\/e56d62ea-4334-4582-a820-2b9406e2b7f8\/usr\/local\/nvidia"}'
   Changing root to /var/lib/mesos/provisioner/containers/5ebbfaf3-3b8b-4c32-9337-740a85feef75/backends/overlay/rootfses/e56d62ea-4334-4582-a820-2b9406e2b7f8
   Mon Mar  5 14:23:41 2018
   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 390.30                 Driver Version: 390.30                    |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
   |===============================+======================+======================|
   |   0  Tesla K40m          Off  | 00000000:82:00.0 Off |                    0 |
   | N/A   21C    P8    18W / 235W |      0MiB / 11441MiB |      0%      Default |
   +-------------------------------+----------------------+----------------------+

   +-----------------------------------------------------------------------------+
   | Processes:                                                       GPU Memory |
   |  GPU       PID   Type   Process name                             Usage      |
   |=============================================================================|
   |  No running processes found                                                 |
   +-----------------------------------------------------------------------------+

Testing Chronos patch for GPU support
=====================================

Patch available on github: https://github.com/mesos/chronos/pull/810

Patch compilation
-----------------

The following steps are needed to test the patch:

1. Get the patched code:

   .. code::

      git clone https://github.com/mesos/chronos.git -b v3.0.2
      cd  chronos
      git fetch origin pull/810/head:chronos
      git checkout chronos

2. Compile:

   .. code::

      docker run -v `pwd`:/chronos --entrypoint=/bin/sh maven:3-jdk-8 -c "\
      curl -sL https://deb.nodesource.com/setup_7.x | bash - \
      && apt-get update && apt-get install -y --no-install-recommends nodejs \
      && ln -sf /usr/bin/nodejs /usr/bin/node \
      && cd /chronos \
      && mvn clean \
      && mvn versions:set -DnewVersion=3.0.2-1 \
      && mvn package -DskipTests"

   The jar **chronos-3.0.2-1.jar** will be created in the folder
   ./target/

3. Create the docker image **Dockerfile:**

   .. code::

      FROM ubuntu:16.04
      ENV DEBIAN_FRONTEND noninteractive
      RUN apt-key adv --keyserver keyserver.ubuntu.com --recv E56151BF && \
      echo deb http://repos.mesosphere.io/ubuntu trusty main > /etc/apt/sources.list.d/mesosphere.list && \
      apt-get update && \
      apt-get -y install --no-install-recommends mesos openjdk-8-jre-headless && \
      apt-get clean && \
      rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
      ADD chronos-3.0.2-1.jar /
      COPY entrypoint.sh /
      ENTRYPOINT ["/entrypoint.sh"]

   **File entrypoint.sh**:

   .. code::

      #!/bin/sh
      CMD="java -Xmx512m -cp chronos-3.0.2-1.jar org.apache.mesos.chronos.scheduler.Main"
      # Parse environment variables
      for k in `set | grep ^CHRONOS_ | cut -d= -f1`; do
          eval v=\$$k
          CMD="$CMD --`echo $k | cut -d_ -f2- | tr '[:upper:]' '[:lower:]'` $v"
      done
      # authentication
      PRINCIPAL=${PRINCIPAL:-root}
      if [ -n "$SECRET" ]; then
          touch /tmp/secret
          chmod 600 /tmp/secret
          echo -n "$SECRET" > /tmp/secret
          CMD="$CMD --mesos_authentication_principal $PRINCIPAL --mesos_authentication_secret_file /tmp/secret"
      fi
      echo $CMD
      if [ $# -gt 0 ]; then
          exec "$@"
      fi
      exec $CMD

**Start the patched Chronos Framework:**

Using the docker image described above you can run Chronos as follows:

.. code::

   docker run --name chronos -d --net host --env-file /etc/chronos/.chronosenv chronos:3.0.2_gpu

with the following environment:

.. code::

   LIBPROCESS_IP=172.20.0.38
   CHRONOS_HOSTNAME=172.20.0.38
   CHRONOS_HTTP_PORT=4400
   CHRONOS_MASTER=zk://172.20.0.38:2181/mesos
   CHRONOS_ZK_HOSTS=zk://172.20.0.38:2181
   CHRONOS_ZK_PATH=/chronos/state
   CHRONOS_MESOS_FRAMEWORK_NAME=chronos
   CHRONOS_HTTP_CREDENTIALS=admin:******
   CHRONOS_ENABLE_FEATURES=gpu_resources

Testing
-------

Approach: submit a batch-like job that uses the tensorflow docker image,
downloads the code available here and runs the convolutional network
example

.. code::

   apt-get update; apt-get install -y git

   git clone https://github.com/aymericdamien/TensorFlow-Examples

   cd TensorFlow-Examples/examples/3_NeuralNetworks;

   time python convolutional_network.py

The test is based on the tutorial provided by mesosphere DC/OS 

Two different versions of the tensorflow docker image will be used in
order to verify the correct execution of the job regardless of the
version of CUDA and cuDNN used to build the binaries inside the docker
image:

================== ==================== =======
Docker tag         CUDA & cuDNN version Test id
================== ==================== =======
latest-gpu (1.6.0) CUDA 9.0, cuDNN 7    #1
1.4.0-gpu          CUDA 8.0, cuDNN 6    #2
================== ==================== =======

Test #1:
~~~~~~~~

Job definition:

.. code::

   {
     "name": "test-gpu",
     "command": "cd $MESOS_SANDBOX && /bin/bash gpu_demo.sh",
     "shell": true,
     "retries": 2,
     "cpus": 4,
     "disk": 256,
     "mem": 4096,
     "gpus": 1,
     "uris": [
       "https://gist.githubusercontent.com/maricaantonacci/1a7f02903513e7bba91f451e0f4f5ead/raw/78c737fd0e2a288a2040c192368f6c4ecf8eb88a/gpu_demo.sh"
     ],
     "environmentVariables": [],
     "arguments": [],
     "runAsUser": "root",
     "container": {
       "type": "MESOS",
       "image": "tensorflow/tensorflow:latest-gpu"
     },
     "schedule": "R/2018-03-05T23:00:00.000Z/PT24H"
   }

The job is correctly run.  The following relevant info were retrieved
from the stderr file in the job sandbox:

.. code::

   Cloning into 'TensorFlow-Examples'...
   /usr/local/lib/python2.7/dist-packages/h5py/__init__.py:36: FutureWarning: Conversion of the second argument of issubdtype from `float` to `np.floating` is deprecated. In future, it will be treated as `np.float64 == np.dtype(float).type`.
     from ._conv import register_converters as _register_converters
   WARNING:tensorflow:Using temporary folder as model directory: /tmp/tmpLLDDDs
   2018-03-05 14:38:59.059890: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1212] Found device 0 with properties:
   name: Tesla K40m major: 3 minor: 5 memoryClockRate(GHz): 0.745
   pciBusID: 0000:82:00.0
   totalMemory: 11.17GiB freeMemory: 11.09GiB
   2018-03-05 14:38:59.059989: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1312] Adding visible gpu devices: 0
   2018-03-05 14:38:59.496393: I tensorflow/core/common_runtime/gpu/gpu_device.cc:993] Creating TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 10757 MB memory) -> physical GPU (device: 0, name: Tesla K40m, pci bus id: 0000:82:00.0, compute capability: 3.5)
   2018-03-05 14:39:23.210323: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1312] Adding visible gpu devices: 0
   2018-03-05 14:39:23.210672: I tensorflow/core/common_runtime/gpu/gpu_device.cc:993] Creating TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 260 MB memory) -> physical GPU (device: 0, name: Tesla K40m, pci bus id: 0000:82:00.0, compute capability: 3.5)

   real    0m32.394s
   user    0m35.192s
   sys 0m12.204s
   I0305 15:39:25.630180  4680 executor.cpp:938] Command exited with status 0 (pid: 4716)

Test #2
~~~~~~~

Job definition:

.. code::

   {
     "name": "test2-gpu",
     "command": "cd $MESOS_SANDBOX && /bin/bash gpu_demo.sh",
     "shell": true,
     "retries": 2,
     "cpus": 4,
     "disk": 256,
     "mem": 4096,
     "gpus": 1,
     "uris": [
       "https://gist.githubusercontent.com/maricaantonacci/1a7f02903513e7bba91f451e0f4f5ead/raw/78c737fd0e2a288a2040c192368f6c4ecf8eb88a/gpu_demo.sh"
     ],
     "environmentVariables": [],
     "arguments": [],
     "runAsUser": "root",
     "container": {
       "type": "MESOS",
       "image": "tensorflow/tensorflow:1.4.0-gpu"
     },
     "schedule": "R/2018-03-05T23:00:00.000Z/PT24H"
   }

As you can see, the only difference wrt Test#1 is the docker image: here
we are using the tag 1.4.0-gpu of the tensorflow docker image that has
been built using a different CUDA and cuDNN version.

Also in this case the job is correcly run:

.. code::

   Cloning into 'TensorFlow-Examples'...
   WARNING:tensorflow:Using temporary folder as model directory: /tmp/tmpomIcq9
   2018-03-05 16:36:24.518455: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX
   2018-03-05 16:36:25.261578: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1030] Found device 0 with properties:
   name: Tesla K40m major: 3 minor: 5 memoryClockRate(GHz): 0.745
   pciBusID: 0000:82:00.0
   totalMemory: 11.17GiB freeMemory: 11.09GiB
   2018-03-05 16:36:25.261658: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: Tesla K40m, pci bus id: 0000:82:00.0, compute capability: 3.5)
   2018-03-05 16:36:52.299346: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: Tesla K40m, pci bus id: 0000:82:00.0, compute capability: 3.5)

   real    0m35.273s
   user    0m37.828s
   sys 0m13.164s
   I0305 17:36:54.803642  7405 executor.cpp:938] Command exited with status 0 (pid: 7439)

**Additional note:**

Running the job without gpu (using the image
tensorflow/tensorflow:latest and “gpus”: 0) we got for the same script:

.. code::

   real   2m15.647s
   user    22m33.384s
   sys 15m51.164s

Testing GPU support in Marathon
===============================

In order to enable GPU support in Marathon, you need to start the
framework with the commandline option –enable_features=gpu_resources (or
using the env variable MARATHON_ENABLE_FEATURES):

**Start Marathon Framework:**

.. code::

   docker run -d --name marathon --net host --env-file /etc/marathon/.marathonenv  indigodatacloud/marathon:1.5.6

with the following enviroment:

.. code::

   LIBPROCESS_IP=<mesos-master-ip>
   MARATHON_HOSTNAME=<mesos-master-fqdn/ip>
   MARATHON_HTTP_ADDRESS=<mesos-master-ip>
   MARATHON_HTTP_PORT=8080
   MARATHON_MASTER=zk://<mesos-master>:2181/mesos
   MARATHON_ZK=zk://<mesos-master>:2181/marathon
   MARATHON_FRAMEWORK_NAME=marathon
   MARATHON_HTTP_CREDENTIALS=admin:******
   MARATHON_ENABLE_FEATURES=gpu_resources

**Test:**

The following application has been submitted to Marathon:

.. code::

   {
     "id": "tensorflow-gpus",
     "cpus": 4,
     "gpus": 1,
     "mem": 2048,
     "disk": 0,
     "instances": 1,
     "container": {
       "type": "MESOS",
       "docker": {
         "image": "tensorflow/tensorflow:latest-gpu"
       }
     }
   }

 Running tensorflow docker container
====================================

1) Using “DOCKER” containerizer on a Mesos cluster without GPUs

Submit to Marathon the following application:

.. code::

   {
     "id": "/tensorflow-app",
     "cmd": "PORT=8888 /run_jupyter.sh --allow-root",
     "cpus": 2,
     "mem": 4096,
     "instances": 1,
     "container": {
       "type": "DOCKER",
       "docker": {
         "image": "tensorflow/tensorflow:latest",
         "network": "BRIDGE",
         "portMappings": [
           {
             "containerPort": 8888,
             "hostPort": 0,
             "servicePort": 10000,
             "protocol": "tcp"
           }
         ],
         "privileged": false,
         "forcePullImage": true
       }
     },
     "env": {
       "PASSWORD": "s3cret"
     },
     "labels": {
       "HAPROXY_GROUP": "external"
     }
   }

Then you can access the service through the cluster LB on port 10000
(servicePort)

2) Using “MESOS” containerizer on a Mesos cluster with GPUs

.. code::

   {
    "id": "tensorflow-gpus",
    "cpus": 4,
    "gpus": 1,
    "mem": 2048,
    "disk": 0,
    "instances": 1,
    "container": {
      "type": "MESOS",
      "docker": {
        "image": "tensorflow/tensorflow:latest-gpu"
      }
    },
    "portDefinitions": [
          {"port": 10000, "name": "http"}
     ],
    "networks": [ { "mode": "host" } ],
    "labels":{
      "HAPROXY_GROUP":"external"
    },
    "env": {
      "PASSWORD":"s3cret"
    }
   }

Then you can access the service through the cluster LB on port 10000. 

If the “port” field in portDefinitions is set to 0 then Marathon will
assign a random service port (that you can know with a GET request to
/v2/apps/app-name) 

References
==========
