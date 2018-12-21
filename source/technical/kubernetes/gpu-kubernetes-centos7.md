<span id="title-text"> DEEP : Installing and testing GPU Node in Kubernetes - CentOS7 </span>
=============================================================================================

-   [Introduction](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Introduction)
-   [Cluster
    Status](#InstallingandtestingGPUNodeinKubernetes-CentOS7-ClusterStatus)
-   [Tests](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Tests)
    -   [Test \#1 - Simple vector-add
        (CUDA8)](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Test#1-Simplevector-add(CUDA8))
    -   [Test \#2 - Simple vector-add with different CUDA
        driver](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Test#2-Simplevector-addwithdifferentCUDAdriver)
    -   [Test \#3 - Simple BinomialOption with different CUDA
        driver](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Test#3-SimpleBinomialOptionwithdifferentCUDAdriver)
    -   [Test \#4 - Job Scheduling
        features](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Test#4-JobSchedulingfeatures)
-   [Access PODs from outside the
    cluster](#InstallingandtestingGPUNodeinKubernetes-CentOS7-AccessPODsfromoutsidethecluster)
    -   [Test \#5 - Long running service with
        NodePort](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Test#5-LongrunningservicewithNodePort)
-   [References](#InstallingandtestingGPUNodeinKubernetes-CentOS7-References)
    -   [Kubernetes Installation/Configuration
        docs](#InstallingandtestingGPUNodeinKubernetes-CentOS7-KubernetesInstallation/Configurationdocs)
    -   [GPU Node -
        CentOS7.4](#InstallingandtestingGPUNodeinKubernetes-CentOS7-GPUNode-CentOS7.4)
        -   [Install
            docker](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Installdocker)
        -   [Driver nvidia-install
            repo](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Drivernvidia-installrepo)
        -   [Driver Nvidia-install
            driver](#InstallingandtestingGPUNodeinKubernetes-CentOS7-DriverNvidia-installdriver)
        -   [Install NVIDIA-docker
            repo](#InstallingandtestingGPUNodeinKubernetes-CentOS7-InstallNVIDIA-dockerrepo)
        -   [Install-NVIDIA
            docker](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Install-NVIDIAdocker)
-   [Related
    articles](#InstallingandtestingGPUNodeinKubernetes-CentOS7-Relatedarticles)

Introduction
------------

The **manual** procedure for installation and configuration od a
Kubernetes cluster is provided. The cluster is composed by a Master node
and one Worker node

<table style="width:100%;">
<colgroup>
<col style="width: 44%" />
<col style="width: 55%" />
</colgroup>
<thead>
<tr class="header">
<th>Mater</th>
<th>Worker node</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><span style="color: rgb(0,0,0);text-decoration: none;">VM<br />
</span></p>
<ul>
<li><p><span style="color: rgb(0,0,0);text-decoration: none;">CentOS 7</span></p></li>
<li><p><span style="color: rgb(0,0,0);text-decoration: none;">K8S 1.10.0</span></p></li>
</ul></td>
<td><p><span style="color: rgb(0,0,0);text-decoration: none;"> Baremetal</span></p>
<ul>
<li><p><span style="color: rgb(0,0,0);text-decoration: none;"> 2xGPU Tesla K40m</span></p></li>
<li><p><span style="color: rgb(0,0,0);text-decoration: none;">CentOS 7</span></p></li>
<li><p><span style="color: rgb(0,0,0);text-decoration: none;">CUDA 9.1</span></p></li>
<li><p><span style="color: rgb(0,0,0);text-decoration: none;">NVIDIA Driver 390.30</span></p></li>
<li><span style="color: rgb(0,0,0);text-decoration: none;">Docker 17.12<br />
</span></li>
</ul></td>
</tr>
</tbody>
</table>

Cluster Status
--------------

Cluster Name: KubeDeep

Kubectl components

``` syntaxhighlighter-pre
# kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"
```

``` syntaxhighlighter-pre
# kubectl get nodes
NAME        STATUS    ROLES     AGE       VERSION
Node1_GPU   Ready     <none>    13d       v1.10.0
```



<span style="color: rgb(0,0,0);text-decoration: none;">
</span>

<span style="color: rgb(0,0,0);text-decoration: none;">Worker GPU
specifications
</span>

``` syntaxhighlighter-pre
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
```

Tests
-----

### <span style="color: rgb(0,0,0);text-decoration: none;">Test \#1 <span style="color: rgb(0,0,0);text-decoration: none;">- Simple vector-add (CUDA8)</span></span>

<span style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;">This CUDA Runtime API
sample is a very basic sample that implements element by element vector
addition.
The examples uses CUDA8 driver.</span></span>

``` syntaxhighlighter-pre
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
```

``` syntaxhighlighter-pre
# kubectl apply -f vector-add.yaml
pod "vector-add" created


# kubectl get pods --show-all
NAME                   READY     STATUS      RESTARTS   AGE
vector-add       0/1       Completed   0     4s
```



### <span style="color: rgb(0,0,0);text-decoration: none;">Test \#2 <span style="color: rgb(0,0,0);text-decoration: none;">- Simple vector-add <span style="color: rgb(0,0,0);text-decoration: none;">with different CUDA driver</span> </span></span>

<span style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;">This CUDA Runtime API
sample is a very basic sample that implements element by element vector
addition.
The examples uses two Docker images with different version of CUDA
driver.
<span style="color: rgb(0,0,0);text-decoration: none;">To complete the
test, a new Docker image with CUDA driver version 9 has been built and
uploaded in a private repo.</span>
</span></span>

``` syntaxhighlighter-pre
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
```

``` syntaxhighlighter-pre
# kubectl apply -f cuda9-vector-add.yaml -f cuda8-vector-add.yaml
pod "cuda9-vector-add" created
pod "cuda8-vector-add" created

# kubectl get pods --show-all
NAME                   READY     STATUS      RESTARTS   AGE
cuda8-vector-add       0/1       Completed   0     2s
cuda9-vector-add       0/1       Completed   0     2s
```



### <span style="color: rgb(0,0,0);text-decoration: none;">Test \#3 <span style="color: rgb(0,0,0);text-decoration: none;">- Simple BinomialOption <span style="color: rgb(0,0,0);text-decoration: none;">with different CUDA driver</span></span></span>

<span style="color: rgb(0,0,0);text-decoration: none;">This sample
evaluates fair call price for a given set of European options under
binomial model.
T<span style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;">o complete the test,
two new Docker images with CUDA8 and CUDA9 has been built and uploaded
in a private repo.</span><span
style="color: rgb(0,0,0);text-decoration: none;">
</span><span style="color: rgb(0,0,0);text-decoration: none;">The test
will take some seconds and GPU engage can be shown</span>
</span></span>



``` syntaxhighlighter-pre
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
```



``` syntaxhighlighter-pre
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
```

### <span style="color: rgb(0,0,0);text-decoration: none;"><span style="color: rgb(0,0,0);text-decoration: none;">Test \#4 - Job Scheduling features </span></span>

<span style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;">Tests highlithing the
features of the Kubernetes scheduler.
Default schedule policies are used (FIFO).</span></span>

<span style="color: rgb(0,0,0);text-decoration: none;">Submission of a
bunch of different cuda jobs with different running time.</span>

-   <span style="color: rgb(0,0,0);text-decoration: none;">Parrec (1h)
    </span>

-   <span
    style="color: rgb(0,0,0);text-decoration: none;">Cuda8-binomialoption.yaml
    (5 min) </span>

-   <span
    style="color: rgb(0,0,0);text-decoration: none;">Cuda9-binomialoption.yaml
    (5 min)</span>

-   <span
    style="color: rgb(0,0,0);text-decoration: none;">Cuda8-vector-add.yaml
    (few sec)</span>

-   <span
    style="color: rgb(0,0,0);text-decoration: none;">Cuda9-vector-add.yaml
    (few sec)</span>

<span style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;">
</span></span>

<span style="color: rgb(0,0,0);text-decoration: none;">The parrec job
has been launched as first job. One GPU has been engaged by the job; the
other is still available for other jobs.</span>



``` syntaxhighlighter-pre
# kubectl apply -f parrec.yaml
pod "parrec" created

# kubectl get pods -o wide
NAME      READY     STATUS    RESTARTS   AGE     IP            NODE
parrec    1/1       Running   0          22s     172.30.0.52   gpu-node-01
```



<span style="color: rgb(0,0,0);text-decoration: none;">Other jobs have
been submitted in the following order:</span>



``` syntaxhighlighter-pre
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
```



<span style="color: rgb(0,0,0);text-decoration: none;">The
“cuda8-binomialoption” is running, the other are in the FIFO queue in
pending state. After completion, the other job will be running in the
same order they have been submitted.</span>



``` syntaxhighlighter-pre
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
```

<span style="color: rgb(0,0,0);text-decoration: none;"><span style="color: rgb(0,0,0);text-decoration: none;">Access PODs from outside the cluster</span></span>
----------------------------------------------------------------------------------------------------------------------------------------------------------------

<span style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;">To access PODs from
outside the cluster it can be possible following different procedures
that strictly depend on the usecase and (cloud) providers.</span></span>

<span style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;"> ***<span
style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;">***NodePort***</span></span>,
hostNetwork***, ***hostPort***, ***LoadBalancer*** and ***Ingress***
features of Kubernetes can be adopted as described in the following
Reference:
</span></span>

<span style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;"><a href="http://alesnosek.com/blog/2017/02/14/accessing-kubernetes-pods-from-outside-of-the-cluster/" class="external-link">http://alesnosek.com/blog/2017/02/14/accessing-kubernetes-pods-from-outside-of-the-cluster/</a></span></span>

<span style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;">For example puroposes,
the **Test \#5** example will describe and use the **<span
style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;">*NodePort*</span></span>**
procedure as the cluster is defined as 1 Master and 1 Worker both with
routable IPs.</span></span>

### Test \#5 <span style="color: rgb(0,0,0);text-decoration: none;"> - Long running service with NodePort </span>

<span
style="color: rgb(0,0,0);text-decoration: none;">Prerequisites</span>

1.  <span style="color: rgb(0,0,0);text-decoration: none;">Kubernetes
    Node with routable IP</span>
2.  <span style="color: rgb(0,0,0);text-decoration: none;"> Port range
    dynamically selected as from the "kubernetes-apiservice.service"
    configuration file</span>
3.  <span style="color: rgb(0,0,0);text-decoration: none;">Nginx replica
    2; <span style="color: rgb(0,0,0);text-decoration: none;">V.
    1.13.12 - latest (as from the YAML files)</span></span>


<span style="color: rgb(0,0,0);text-decoration: none;">Yaml files
related to nginx deployment and nginx service</span>

``` syntaxhighlighter-pre
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
```

``` syntaxhighlighter-pre
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
```

<span style="color: rgb(0,0,0);text-decoration: none;">Creation of nginx
POD and nginx service. The following commands will return the Node
hostname and the port associated to the nginx.</span>

``` syntaxhighlighter-pre
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
```

<span style="color: rgb(0,0,0);text-decoration: none;">Test of
nginx</span>

``` syntaxhighlighter-pre
# curl http://gpu-node-01:30916
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
…
```

<span style="color: rgb(0,0,0);text-decoration: none;">Delete one of the
two PODs from nginx</span>

``` syntaxhighlighter-pre
# kubectl delete pod nginx-example-78847794b7-8nm8t
pod "nginx-example-78847794b7-8nm8t" deleted
```

<span style="color: rgb(0,0,0);text-decoration: none;">A new POD is
creating while the old POD is getting deleted. No service downtime is
registered from the user</span>

``` syntaxhighlighter-pre
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
```

<span style="color: rgb(0,0,0);text-decoration: none;"><span
style="color: rgb(0,0,0);text-decoration: none;">Delete the seconf POD.
The internal POD IP is changing, but the public endpoint is the same.
</span></span>

``` syntaxhighlighter-pre
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
```

<span style="color: rgb(0,0,0);text-decoration: none;">Changing the
version of nginx with an older version (v. 1.12).</span>

``` syntaxhighlighter-pre
# cat ngnix.deploy.yaml
apiVersion: extensions/v1beta1
…
     containers:
     - image: nginx:1.12
```

<span style="color: rgb(0,0,0);text-decoration: none;">Apply changes.
New PODs are created while old PODs are deleted. No nginx downtime is
registered from the user. Public endpoint and port remain
unchanged.</span>

``` syntaxhighlighter-pre
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
```



References
----------

The following guides have been followed for the Installation and
Configuration of Kubernetes cluster - a detailed step-by-step guide will
be provided soon

### <span style="color: rgb(0,0,0);text-decoration: none;">Kubernetes Installation/Configuration docs</span>

<a href="https://github.com/kelseyhightower/kubernetes-the-hard-way" class="external-link"><span style="color: rgb(17,85,204);text-decoration: underline;">https://github.com/kelseyhightower/kubernetes-the-hard-way</span></a>

### <span style="color: rgb(0,0,0);text-decoration: none;">GPU Node - CentOS7.4</span>

#### <span style="color: rgb(0,0,0);text-decoration: none;">Install docker</span>

<a href="https://docs.docker.com/install/linux/docker-ce/centos/#set-up-the-repository" class="external-link"><span style="color: rgb(17,85,204);text-decoration: underline;">https://docs.docker.com/install/linux/docker-ce/centos/#set-up-the-repository</span></a>

#### <span style="color: rgb(0,0,0);text-decoration: none;">Driver nvidia-install repo</span>

<a href="https://developer.nvidia.com/cuda-downloads?target_os=Linux" class="external-link"><span style="color: rgb(17,85,204);text-decoration: underline;">https://developer.nvidia.com/cuda-downloads?target_os=Linux</span></a>

#### <span style="color: rgb(0,0,0);text-decoration: none;">Driver Nvidia-install driver</span>

<a href="http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#post-installation-actions" class="external-link"><span style="color: rgb(17,85,204);text-decoration: underline;">http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#post-installation-actions</span></a>

#### <span style="color: rgb(0,0,0);text-decoration: none;">Install NVIDIA-docker repo</span>

<a href="https://nvidia.github.io/nvidia-docker/" class="external-link"><span style="color: rgb(17,85,204);text-decoration: underline;">https://nvidia.github.io/nvidia-docker/</span></a>

#### <span style="color: rgb(0,0,0);text-decoration: none;">Install-NVIDIA docker</span>

<a href="https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0)#prerequisites" class="external-link"><span style="color: rgb(17,85,204);text-decoration: underline;">https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0)#prerequisites</span></a>

