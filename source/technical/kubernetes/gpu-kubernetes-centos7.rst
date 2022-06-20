DEEP : Installing and testing GPU Node in Kubernetes - CentOS7
==============================================================

-  `Introduction <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Introduction>`__
-  `Cluster
   Status <#InstallingandtestingGPUNodeinKubernetes-CentOS7-ClusterStatus>`__
-  `Tests <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Tests>`__

   -  `Test #1 - Simple vector-add
      (CUDA8) <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Test#1-Simplevector-add(CUDA8)>`__
   -  `Test #2 - Simple vector-add with different CUDA
      driver <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Test#2-Simplevector-addwithdifferentCUDAdriver>`__
   -  `Test #3 - Simple BinomialOption with different CUDA
      driver <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Test#3-SimpleBinomialOptionwithdifferentCUDAdriver>`__
   -  `Test #4 - Job Scheduling
      features <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Test#4-JobSchedulingfeatures>`__

-  `Access PODs from outside the
   cluster <#InstallingandtestingGPUNodeinKubernetes-CentOS7-AccessPODsfromoutsidethecluster>`__

   -  `Test #5 - Long running service with
      NodePort <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Test#5-LongrunningservicewithNodePort>`__

-  `References <#InstallingandtestingGPUNodeinKubernetes-CentOS7-References>`__

   -  `Kubernetes Installation/Configuration
      docs <#InstallingandtestingGPUNodeinKubernetes-CentOS7-KubernetesInstallation/Configurationdocs>`__
   -  `GPU Node -
      CentOS7.4 <#InstallingandtestingGPUNodeinKubernetes-CentOS7-GPUNode-CentOS7.4>`__

      -  `Install
         docker <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Installdocker>`__
      -  `Driver nvidia-install
         repo <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Drivernvidia-installrepo>`__
      -  `Driver Nvidia-install
         driver <#InstallingandtestingGPUNodeinKubernetes-CentOS7-DriverNvidia-installdriver>`__
      -  `Install NVIDIA-docker
         repo <#InstallingandtestingGPUNodeinKubernetes-CentOS7-InstallNVIDIA-dockerrepo>`__
      -  `Install-NVIDIA
         docker <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Install-NVIDIAdocker>`__

-  `Related
   articles <#InstallingandtestingGPUNodeinKubernetes-CentOS7-Relatedarticles>`__

Introduction
------------

The **manual** procedure for installation and configuration od a
Kubernetes cluster is provided. The cluster is composed by a Master node
and one Worker node

.. raw:: html

   <table style="width:100%;">

.. raw:: html

   <colgroup>

.. raw:: html

   <col style="width: 44%" />

.. raw:: html

   <col style="width: 55%" />

.. raw:: html

   </colgroup>

.. raw:: html

   <thead>

.. raw:: html

   <tr class="header">

.. raw:: html

   <th>

Mater

.. raw:: html

   </th>

.. raw:: html

   <th>

Worker node

.. raw:: html

   </th>

.. raw:: html

   </tr>

.. raw:: html

   </thead>

.. raw:: html

   <tbody>

.. raw:: html

   <tr class="odd">

.. raw:: html

   <td>

.. raw:: html

   <p>

VM

.. raw:: html

   </p>

.. raw:: html

   <ul>

.. raw:: html

   <li>

.. raw:: html

   <p>

CentOS 7

.. raw:: html

   </p>

.. raw:: html

   </li>

.. raw:: html

   <li>

.. raw:: html

   <p>

K8S 1.10.0

.. raw:: html

   </p>

.. raw:: html

   </li>

.. raw:: html

   </ul>

.. raw:: html

   </td>

.. raw:: html

   <td>

.. raw:: html

   <p>

Baremetal

.. raw:: html

   </p>

.. raw:: html

   <ul>

.. raw:: html

   <li>

.. raw:: html

   <p>

2xGPU Tesla K40m

.. raw:: html

   </p>

.. raw:: html

   </li>

.. raw:: html

   <li>

.. raw:: html

   <p>

CentOS 7

.. raw:: html

   </p>

.. raw:: html

   </li>

.. raw:: html

   <li>

.. raw:: html

   <p>

CUDA 9.1

.. raw:: html

   </p>

.. raw:: html

   </li>

.. raw:: html

   <li>

.. raw:: html

   <p>

NVIDIA Driver 390.30

.. raw:: html

   </p>

.. raw:: html

   </li>

.. raw:: html

   <li>

Docker 17.12

.. raw:: html

   </li>

.. raw:: html

   </ul>

.. raw:: html

   </td>

.. raw:: html

   </tr>

.. raw:: html

   </tbody>

.. raw:: html

   </table>

Cluster Status
--------------

Cluster Name: KubeDeep

Kubectl components

.. code::

   # kubectl get componentstatuses
   NAME                 STATUS    MESSAGE              ERROR
   controller-manager   Healthy   ok
   scheduler            Healthy   ok
   etcd-0               Healthy   {"health": "true"

.. code::

   # kubectl get nodes
   NAME        STATUS    ROLES     AGE       VERSION
   Node1_GPU   Ready     <none>    13d       v1.10.0

Worker GPU specifications

.. code::

   # nvidia-smi
   Mon Apr  2 23:13:37 2018
   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 390.30                 Driver Version: 390.30                    |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name     Persistence-M   | Bus-Id       Disp.A  | Volatile Uncorr. ECC |
   | Fan  Temp Perf  Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
   |===============================+======================+======================|
   |   0 Tesla K40m          On    | 00000000:02:00.0 Off |                    0 |
   | N/A   30C    P8    20W / 235W |      0MiB / 11441MiB |      0%      Default |
   +-------------------------------+----------------------+----------------------+
   |   1 Tesla K40m          On    | 00000000:84:00.0 Off |                    0 |
   | N/A   32C    P8    20W / 235W |      0MiB / 11441MiB |      0%      Default |
   +-------------------------------+----------------------+----------------------+
   +-----------------------------------------------------------------------------+
   | Processes:                                                       GPU Memory |
   |  GPU       PID   Type Process name                               Usage      |
   |=============================================================================|
   |  No running processes found                                                 |
   +-----------------------------------------------------------------------------+

Tests
-----

Test #1 - Simple vector-add (CUDA8)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This CUDA Runtime API sample is a very basic sample that implements
element by element vector addition. The examples uses CUDA8 driver.

.. code::

   #cat vector-add.yaml
   apiVersion: v1
   kind: Pod
   metadata:
    name: vector-add
   spec:
    restartPolicy: OnFailure
    containers:
       - name: cuda-vector-add
        # https://github.com/kubernetes/kubernetes/blob/v1.7.11/test/images/nvidia-cuda/Dockerfile
        image: "k8s.gcr.io/cuda-vector-add:v0.1"
        resources:
          limits:
            nvidia.com/gpu: 1

.. code::

   # kubectl apply -f vector-add.yaml
   pod "vector-add" created


   # kubectl get pods --show-all
   NAME                   READY     STATUS      RESTARTS   AGE
   vector-add       0/1       Completed   0     4s

Test #2 - Simple vector-add with different CUDA driver
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This CUDA Runtime API sample is a very basic sample that implements
element by element vector addition. The examples uses two Docker images
with different version of CUDA driver. To complete the test, a new
Docker image with CUDA driver version 9 has been built and uploaded in a
private repo.

.. code::

   # cat cuda8-vector-add.yaml
   apiVersion: v1
   kind: Pod
   metadata:
    name: cuda8-vector-add
   spec:
    restartPolicy: OnFailure
    containers:
       - name: cuda-vector-add
        # https://github.com/kubernetes/kubernetes/blob/v1.7.11/test/images/nvidia-cuda/Dockerfile
        image: "k8s.gcr.io/cuda-vector-add:v0.1"
        resources:
          limits:
            nvidia.com/gpu: 1

   # cat cuda9-vector-add.yaml
   apiVersion: v1
   kind: Pod
   metadata:
    name: cuda9-vector-add
   spec:
    restartPolicy: OnFailure
    containers:
       - name: cuda-vector-add
        image: <private repo>/deep/cuda-vector-add:v0.2
        resources:
          limits:
            nvidia.com/gpu: 1

.. code::

   # kubectl apply -f cuda9-vector-add.yaml -f cuda8-vector-add.yaml
   pod "cuda9-vector-add" created
   pod "cuda8-vector-add" created

   # kubectl get pods --show-all
   NAME                   READY     STATUS      RESTARTS   AGE
   cuda8-vector-add       0/1       Completed   0     2s
   cuda9-vector-add       0/1       Completed   0     2s

Test #3 - Simple BinomialOption with different CUDA driver
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This sample evaluates fair call price for a given set of European
options under binomial model. To complete the test, two new Docker
images with CUDA8 and CUDA9 has been built and uploaded in a private
repo.The test will take some seconds and GPU engage can be shown

.. code::

   # cat cuda8-binomialoption.yaml
   apiVersion: v1
   kind: Pod
   metadata:
    name: cuda8-binomialoption
   spec:
    restartPolicy: OnFailure
    containers:
       - name: cuda8-binomilaoption
        image:  <private_repo>/deep/cuda-binomialoption:v0.1
        resources:
          limits:
            nvidia.com/gpu: 1

   # cat cuda9-binomialoption.yaml
   apiVersion: v1
   kind: Pod
   metadata:
    name: cuda9-binomialoption
   spec:
    restartPolicy: OnFailure
    containers:
       - name: cuda9-binomialoption
        image: <private_repo>/deep/cuda-binomialoption:v0.2
        resources:
          limits:
            nvidia.com/gpu: 1

.. code::

   # kubectl apply -f cuda8-binomialoption.yaml -f cuda9-binomialoption.yaml
   pod "cuda8-binomialoption" created
   pod "cuda9-binomialoption" created

   # kubectl get pods --show-all
   NAME                   READY     STATUS      RESTARTS   AGE
   cuda8-binomialoption   1/1     Running     0          2s
   cuda9-binomialoption   1/1     Running     0          2s

   # kubectl get pods --show-all
   NAME                   READY     STATUS      RESTARTS   AGE
   cuda8-binomialoption   1/1     Running     0          22s
   cuda9-binomialoption   1/1     Running     0          22s

   # kubectl get pods --show-all
   NAME                   READY     STATUS      RESTARTS   AGE
   cuda8-binomialoption   0/1     Completed   0     1m
   cuda9-binomialoption   0/1     Completed   0     1m

   # nvidia-smi
   Mon Apr  2 23:35:17 2018
   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 390.30                 Driver Version: 390.30                    |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name     Persistence-M   | Bus-Id        Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp Perf  Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
   |===============================+======================+======================|
   |   0 Tesla K40m          On    | 00000000:02:00.0 Off |                    0 |
   | N/A   31C    P0    63W / 235W |     80MiB / 11441MiB |      0%      Default |
   +-------------------------------+----------------------+----------------------+
   |   1 Tesla K40m          On    | 00000000:84:00.0 Off |                    0 |
   | N/A   33C    P0    63W / 235W |     80MiB / 11441MiB |      0%      Default |
   +-------------------------------+----------------------+----------------------+
   +-----------------------------------------------------------------------------+
   | Processes:                                                       GPU Memory |
   |  GPU       PID   Type Process name                               Usage      |
   |=============================================================================|
   |    0      3385      C   ./binomialOptions                             69MiB |
   |    1      3369      C   ./binomialOptions                             69MiB |
   +-----------------------------------------------------------------------------+

Test #4 - Job Scheduling features
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Tests highlithing the features of the Kubernetes scheduler. Default
schedule policies are used (FIFO).

Submission of a bunch of different cuda jobs with different running
time.

-  Parrec (1h)

-  Cuda8-binomialoption.yaml (5 min)

-  Cuda9-binomialoption.yaml (5 min)

-  Cuda8-vector-add.yaml (few sec)

-  Cuda9-vector-add.yaml (few sec)

The parrec job has been launched as first job. One GPU has been engaged
by the job; the other is still available for other jobs.

.. code::

   # kubectl apply -f parrec.yaml
   pod "parrec" created

   # kubectl get pods -o wide
   NAME      READY     STATUS    RESTARTS   AGE     IP            NODE
   parrec    1/1       Running   0          22s     172.30.0.52   gpu-node-01

Other jobs have been submitted in the following order:

.. code::

   # kubectl apply -f cuda8-binomialoption.yaml -f cuda9-binomialoption.yaml -f cuda8-vector-add.yaml -f cuda9-vector-add.yaml
   pod "cuda8-binomialoption" created
   pod "cuda9-binomialoption" created
   pod "cuda8-vector-add" created
   pod "cuda9-vector-add" created

   # kubectl get pods -o wide
   NAME                   READY     STATUS    RESTARTS   AGE     IP            NODE
   cuda8-binomialoption   1/1       Running   0          4s      172.30.0.53   gpu-node-01
   cuda8-vector-add       0/1       Pending   0          4s      <none>        <none>
   cuda9-binomialoption   0/1       Pending   0          4s      <none>        <none>
   cuda9-vector-add       0/1       Pending   0          4s      <none>        <none>
   parrec                 1/1       Running   0          1m      172.30.0.52   gpu-node-01

The “cuda8-binomialoption” is running, the other are in the FIFO queue
in pending state. After completion, the other job will be running in the
same order they have been submitted.

.. code::

   # kubectl get pods -o wide

   NAME                   READY     STATUS    RESTARTS   AGE     IP            NODE
   cuda8-binomialoption   1/1       Running   0          31s     172.30.0.53   gpu-node-01
   cuda8-vector-add       0/1       Pending   0          31s     <none>        <none>
   cuda9-binomialoption   0/1       Pending   0          31s     <none>        <none>
   cuda9-vector-add       0/1       Pending   0          31s     <none>        <none>
   parrec                 1/1       Running   0          2m      172.30.0.52   gpu-node-01

   # kubectl get pods -o wide
   NAME                   READY     STATUS      RESTARTS   AGE     IP            NODE
   cuda8-binomialoption   0/1       Completed   0          49s     172.30.0.53   gpu-node-01
   cuda8-vector-add       0/1       Pending     0          49s     <none>        <none>
   cuda9-binomialoption   0/1       Pending     0          49s     <none>        <none>
   cuda9-vector-add       0/1       Pending     0          49s     <none>        <none>
   parrec                 1/1       Running     0          2m      172.30.0.52   gpu-node-01

   # kubectl get pods -o wide
   NAME                   READY     STATUS              RESTARTS   AGE     IP            NODE
   cuda8-binomialoption   0/1       Completed           0          1m      172.30.0.53   gpu-node-01
   cuda8-vector-add       0/1       Pending             0          1m      <none>        <none>
   cuda9-binomialoption   0/1       ContainerCreating   0          1m      <none>        gpu-node-01
   cuda9-vector-add       0/1       Pending             0          1m      <none>        <none>
   parrec                 1/1       Running             0          2m      172.30.0.52   gpu-node-01

   # kubectl get pods -o wide
   NAME                   READY     STATUS      RESTARTS   AGE     IP            NODE
   cuda8-binomialoption   0/1       Completed   0          1m      172.30.0.53   gpu-node-01
   cuda8-vector-add       0/1       Pending     0          1m      <none>        <none>
   cuda9-binomialoption   1/1       Running     0          1m      172.30.0.54   gpu-node-01
   cuda9-vector-add       0/1       Pending     0          1m      <none>        <none>
   parrec                 1/1       Running     0          2m      172.30.0.52   gpu-node-01

   # kubectl get pods -o wide
   NAME                   READY     STATUS      RESTARTS   AGE     IP            NODE
   cuda8-binomialoption   0/1       Completed   0          2m      172.30.0.53   gpu-node-01
   cuda8-vector-add       0/1       Completed   0          2m      172.30.0.55   gpu-node-01
   cuda9-binomialoption   0/1       Completed   0          2m      172.30.0.54   gpu-node-01
   cuda9-vector-add       0/1       Pending     0          2m      <none>        <none>
   parrec                 1/1       Running     0          3m      172.30.0.52   gpu-node-01

   # kubectl get pods -o wide
   NAME                   READY     STATUS      RESTARTS   AGE     IP            NODE
   cuda8-binomialoption   0/1       Completed   0          2m      172.30.0.53   gpu-node-01
   cuda8-vector-add       0/1       Completed   0          2m      172.30.0.55   gpu-node-01
   cuda9-binomialoption   0/1       Completed   0          2m      172.30.0.54   gpu-node-01
   cuda9-vector-add       0/1       Completed   0          2m      172.30.0.56   gpu-node-01
   parrec                 1/1       Running     0          4m      172.30.0.52   gpu-node-01

Access PODs from outside the cluster
------------------------------------

To access PODs from outside the cluster it can be possible following
different procedures that strictly depend on the usecase and (cloud)
providers.

NodePort, hostNetwork, hostPort, LoadBalancer and Ingress features of
Kubernetes can be adopted as described in the following Reference:

http://alesnosek.com/blog/2017/02/14/accessing-kubernetes-pods-from-outside-of-the-cluster/

For example puroposes, the Test #5 example will describe and use the
NodePort procedure as the cluster is defined as 1 Master and 1 Worker
both with routable IPs.

Test #5 - Long running service with NodePort
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Prerequisites

1. Kubernetes Node with routable IP
2.  Port range dynamically selected as from the
   “kubernetes-apiservice.service” configuration file
3. Nginx replica 2; V. 1.13.12 - latest (as from the YAML files)

Yaml files related to nginx deployment and nginx service

.. code::

   # cat ngnix.deploy.yaml
   apiVersion: extensions/v1beta1
   kind: Deployment
   metadata:
    name: nginx-example
    namespace: default
    labels:
       app: nginx
   spec:
    replicas: 2
    strategy:
       type: RollingUpdate
       rollingUpdate:
        maxSurge: 1
        maxUnavailable: 0
    template:
       metadata:
        labels:
          app: nginx
          name: nginx
       spec:
        containers:
        - image: nginx:latest
          name: ingress-example
          ports:
          - name: http
            containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
              scheme: HTTP
          livenessProbe:
            httpGet:
              path: /
              port: 80
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1

.. code::

   # cat ngnix.svc.yaml
   apiVersion: v1
   kind: Service
   metadata:
    name: nginx
    namespace: default
   spec:
    type: NodePort
    ports:
    - name: http
       port: 80
       targetPort: 80
       protocol: TCP
    selector:
       app: nginx

Creation of nginx POD and nginx service. The following commands will
return the Node hostname and the port associated to the nginx.

.. code::

   # kubectl apply -f ngnix.deploy.yaml -f ngnix.svc.yaml
   deployment.extensions "nginx-example" created
   service "nginx" created
   # kubectl get pods
   NAME                             READY     STATUS      RESTARTS   AGE
   nginx-example-78847794b7-8nm8t   0/1     Running       0          11s
   nginx-example-78847794b7-n8nxs   0/1     Running       0          11s

   # kubectl get pods
   NAME                             READY     STATUS      RESTARTS   AGE
   nginx-example-78847794b7-8nm8t   1/1     Running       0          30s
   nginx-example-78847794b7-n8nxs   1/1     Running       0          30s

   # kubectl get svc
   NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
   nginx        NodePort    192.168.0.130   <none>     80:30916/TCP   51s

Test of nginx

.. code::

   # curl http://gpu-node-01:30916
   <!DOCTYPE html>
   <html>
   <head>
   <title>Welcome to nginx!</title>
   <style>
   …

Delete one of the two PODs from nginx

.. code::

   # kubectl delete pod nginx-example-78847794b7-8nm8t
   pod "nginx-example-78847794b7-8nm8t" deleted

A new POD is creating while the old POD is getting deleted. No service
downtime is registered from the user

.. code::

   # kubectl get pods
   NAME                             READY     STATUS        RESTARTS   AGE
   nginx-example-78847794b7-6gvnn   0/1     Running         0          4s
   nginx-example-78847794b7-8nm8t   0/1     Terminating     0          12m
   nginx-example-78847794b7-n8nxs   1/1     Running         0          12m

   # kubectl get pods
   NAME                             READY     STATUS      RESTARTS   AGE
   nginx-example-78847794b7-6gvnn   0/1     Running       0          12s
   nginx-example-78847794b7-n8nxs   1/1     Running       0          12m

   # kubectl get pods -o wide
   NAME                             READY     STATUS      RESTARTS   AGE     IP            NODE
   nginx-example-78847794b7-6gvnn   1/1     Running       0          28s     172.30.0.61   gpu-node-01
   nginx-example-78847794b7-n8nxs   1/1     Running       0          12m     172.30.0.59   gpu-node-01

Delete the seconf POD. The internal POD IP is changing, but the public
endpoint is the same.

.. code::

   # kubectl delete pod nginx-example-78847794b7-n8nxs
   pod "nginx-example-78847794b7-n8nxs" deleted

   # kubectl get pods -o wide
   NAME                             READY     STATUS        RESTARTS   AGE     IP            NODE
   nginx-example-78847794b7-2szlv   0/1     Running         0          4s      172.30.0.62   gpu-node-01
   nginx-example-78847794b7-6gvnn   1/1     Running         0          50s     172.30.0.61   gpu-node-01
   nginx-example-78847794b7-n8nxs   0/1     Terminating     0          13m     172.30.0.59   gpu-node-01
   # kubectl get pods -o wide
   NAME                             READY     STATUS      RESTARTS   AGE     IP            NODE
   nginx-example-78847794b7-2szlv   1/1     Running       0          13s     172.30.0.62   gpu-node-01
   nginx-example-78847794b7-6gvnn   1/1     Running       0          59s     172.30.0.61   gpu-node-01

Changing the version of nginx with an older version (v. 1.12).

.. code::

   # cat ngnix.deploy.yaml
   apiVersion: extensions/v1beta1
   …
        containers:
        - image: nginx:1.12

Apply changes. New PODs are created while old PODs are deleted. No nginx
downtime is registered from the user. Public endpoint and port remain
unchanged.

.. code::

   # kubectl apply -f ngnix.deploy.yaml -f ngnix.svc.yaml
   deployment.extensions "nginx-example" configured
   service "nginx" unchanged

   # kubectl get pods
   NAME                             READY     STATUS      RESTARTS   AGE
   nginx-example-5d9764f848-8kc8b   0/1     Running       0          9s
   nginx-example-78847794b7-2szlv   1/1     Running       0          2m
   nginx-example-78847794b7-6gvnn   1/1     Running       0          2m

   # kubectl get pods -o wide
   NAME                             READY     STATUS        RESTARTS   AGE     IP            NODE
   nginx-example-5d9764f848-8kc8b   1/1     Running         0          23s     172.30.0.63   gpu-node-01
   nginx-example-5d9764f848-xwr77   1/1     Running         0          7s      172.30.0.64   gpu-node-01
   nginx-example-78847794b7-6gvnn   0/1     Terminating     0          3m      172.30.0.61   gpu-node-01

   # kubectl get pods -o wide
   NAME                             READY     STATUS      RESTARTS   AGE     IP            NODE
   nginx-example-5d9764f848-8kc8b   1/1     Running       0          54s     172.30.0.63   gpu-node-01
   nginx-example-5d9764f848-xwr77   1/1     Running       0          38s     172.30.0.64   gpu-node-01

References
----------

The following guides have been followed for the Installation and
Configuration of Kubernetes cluster - a detailed step-by-step guide will
be provided soon

Kubernetes Installation/Configuration docs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

https://github.com/kelseyhightower/kubernetes-the-hard-way

GPU Node - CentOS7.4
~~~~~~~~~~~~~~~~~~~~

Install docker
^^^^^^^^^^^^^^

https://docs.docker.com/install/linux/docker-ce/centos/#set-up-the-repository

Driver nvidia-install repo
^^^^^^^^^^^^^^^^^^^^^^^^^^

https://developer.nvidia.com/cuda-downloads?target_os=Linux

Driver Nvidia-install driver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#post-installation-actions

Install NVIDIA-docker repo
^^^^^^^^^^^^^^^^^^^^^^^^^^

https://nvidia.github.io/nvidia-docker/

Install-NVIDIA docker
^^^^^^^^^^^^^^^^^^^^^

https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0)#prerequisites
