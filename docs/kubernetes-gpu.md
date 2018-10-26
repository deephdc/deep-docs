<span id="title-text"> Installing GPU node and adding it to Kubernetes cluster </span>
=============================================================================================

This is a guide on how to install a GPU node and join it in a running
Kubernetes cluster deployed with kubeadm. The guide was tested on a
Kubernetes cluster v1.9.4 installed with kubeadm. The cluster nodes are
KVM virtual machines deployed by OpenStack. VMs are running Ubuntu
16.04.4 LTS. The node with GPU has a single NVIDIA K20m GPU card.

Step-by-step guide
------------------

1.  We start with a blank node with a GPU. This is the node, we would
    like to join in our Kubernetes cluster. First, update the node and
    install graphic drivers. The version of the drivers has to be at
    least 361.93. We have installed version 387.26 and CUDA Version
    8.0.61. Drivers and CUDA installation is not a part of this guide.

    **NVIDIA drivers information** <span
    class="collapse-source expand-control" style="display:none;"><span
    class="expand-control-icon icon"> </span><span
    class="expand-control-text">Expand source</span></span> <span
    class="collapse-spinner-wrapper"></span>

    ``` syntaxhighlighter-pre
    ubuntu@virtual-kubernetes-gpu-2:~$ nvidia-smi
    Wed Mar 14 08:52:53 2018       
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 387.26                 Driver Version: 387.26                    |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  Tesla K20m          Off  | 00000000:00:07.0 Off |                    0 |
    | N/A   30C    P0    53W / 225W |      0MiB /  4742MiB |    100%      Default |
    +-------------------------------+----------------------+----------------------+
                                                                                   
    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID   Type   Process name                             Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+
    ```

    **CUDA information** <span class="collapse-source expand-control"
    style="display:none;"><span
    class="expand-control-icon icon"> </span><span
    class="expand-control-text">Expand source</span></span> <span
    class="collapse-spinner-wrapper"></span>

    ``` syntaxhighlighter-pre
    ubuntu@virtual-kubernetes-gpu-2:~$ cat /usr/local/cuda-8.0/version.txt 
    CUDA Version 8.0.61
    ```

2.  The next step is to install Docker on the GPU node. Install Docker
    CE 17.03 from Docker’s repositories for Ubuntu. Proceed with the
    following commands as a root user.

    ``` syntaxhighlighter-pre
    sudo apt-get update
    sudo apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    sudo add-apt-repository \
       "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
       $(lsb_release -cs) \
       stable"
    sudo apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')
    ```

    **Docker installation test** <span
    class="collapse-source expand-control" style="display:none;"><span
    class="expand-control-icon icon"> </span><span
    class="expand-control-text">Expand source</span></span> <span
    class="collapse-spinner-wrapper"></span>

    ``` syntaxhighlighter-pre
    root@virtual-kubernetes-gpu-2:~# docker --version
    Docker version 17.03.2-ce, build f5ec1e2
    root@virtual-kubernetes-gpu-2:~# docker run hello-world
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    ca4f61b1923c: Pull complete 
    Digest: sha256:97ce6fa4b6cdc0790cda65fe7290b74cfebd9fa0c9b8c38e979330d547d22ce1
    Status: Downloaded newer image for hello-world:latest

    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
     https://cloud.docker.com/

    For more examples and ideas, visit:
     https://docs.docker.com/engine/userguide/
    ```

3.  On the GPU node, add nvidia-docker2 package repositories, install it
    and reload Docker daemon configuration, which might be altered by
    nvidia-docker2 installation.

    ``` syntaxhighlighter-pre
    sudo curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
      sudo apt-key add -
    sudo curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu16.04/amd64/nvidia-docker.list | \
      sudo tee /etc/apt/sources.list.d/nvidia-docker.list
    sudo apt-get update
    sudo apt-get install -y nvidia-docker2
    sudo pkill -SIGHUP dockerd
    ```

    **nvidia-docker2 GPU test** <span
    class="collapse-source expand-control" style="display:none;"><span
    class="expand-control-icon icon"> </span><span
    class="expand-control-text">Expand source</span></span> <span
    class="collapse-spinner-wrapper"></span>

    ``` syntaxhighlighter-pre
    root@virtual-kubernetes-gpu-2:~# docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
    Unable to find image 'nvidia/cuda:latest' locally
    latest: Pulling from nvidia/cuda
    22dc81ace0ea: Pull complete 
    1a8b3c87dba3: Pull complete 
    91390a1c435a: Pull complete 
    07844b14977e: Pull complete 
    b78396653dae: Pull complete 
    95e837069dfa: Pull complete 
    fef4aadda783: Pull complete 
    343234bd5cf3: Pull complete 
    64e8786fc8c1: Pull complete 
    d6a4723d353c: Pull complete 
    Digest: sha256:3524adf9b563c27d9a0f6d0584355c1f4f4b38e90b66289b8f8de026a9162eee
    Status: Downloaded newer image for nvidia/cuda:latest
    Wed Mar 14 10:14:51 2018       
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 387.26                 Driver Version: 387.26                    |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  Tesla K20m          Off  | 00000000:00:07.0 Off |                    0 |
    | N/A   30C    P0    52W / 225W |      0MiB /  4742MiB |    100%      Default |
    +-------------------------------+----------------------+----------------------+
                                                                                   
    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID   Type   Process name                             Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+
    ```

4.  Set nvidia-runtime as the default runtime for Docker on the GPU
    node. Edit the `/etc/docker/daemon.json` configuration file and set
    the <span
    style="font-family: monospace;">"default-runtime"</span> parameter
    to nvidia. This also allows us to ommit the <span
    style="font-family: monospace;">--runtime=nvidia</span> parameter
    for Docker.

    ``` syntaxhighlighter-pre
    {
        "default-runtime": "nvidia",
        "runtimes": {
            "nvidia": {
                "path": "/usr/bin/nvidia-container-runtime",
                "runtimeArgs": []
            }
        }
    }
    ```

5.  As a root user on the GPU node, add Kubernetes package repositories
    and install kubeadm, kubectl and kubelet. Then turn the swap off as
    it is not supported by Kubernetes.

    ``` syntaxhighlighter-pre
    apt-get update && apt-get install -y apt-transport-https
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    apt-get update
    apt-get install -y kubelet kubeadm kubectl
    # turn off swap or comment the swap line in /etc/fstab
    sudo swapoff -a
    ```

    **Specific version installation; e.g., 1.9.3-00** <span
    class="collapse-source expand-control" style="display:none;"><span
    class="expand-control-icon icon"> </span><span
    class="expand-control-text">Expand source</span></span> <span
    class="collapse-spinner-wrapper"></span>

    ``` syntaxhighlighter-pre
    # install aptitude, an interface to package manager 
    root@virtual-kubernetes-gpu-2:~# apt install aptitude -y

    # show available kubeadm versions in the repositories
    root@virtual-kubernetes-gpu-2:~# aptitude versions kubeadm
    Package kubeadm:                        
    p   1.5.7-00   kubernetes-xenial   500 
    p   1.6.1-00   kubernetes-xenial   500 
    p   1.6.2-00   kubernetes-xenial   500
    ...
    p   1.9.3-00   kubernetes-xenial   500
    p   1.9.4-00   kubernetes-xenial   500

    # install specific version of kubelet, kubeadm and kubectl
    root@virtual-kubernetes-gpu-2:~# apt-get install -y kubelet=1.9.3-00 kubeadm=1.9.3-00 kubectl=1.9.3-00
    ```

6.  On the GPU node, edit the <span
    style="font-family: monospace;">/etc/systemd/system/kubelet.service.d/10-kubeadm.conf</span> file
    add the following environment argument to enable <span
    style="font-family: monospace;">DevicePlugins</span> feature gate.
    If there is already <span
    style="font-family: monospace;">Accelerators</span> feature gate set
    , remove it.

    ``` syntaxhighlighter-pre
    Environment="KUBELET_EXTRA_ARGS=--feature-gates=DevicePlugins=true"
    ```

    **/etc/systemd/system/kubelet.service.d/10-kubeadm.conf** <span
    class="collapse-source expand-control" style="display:none;"><span
    class="expand-control-icon icon"> </span><span
    class="expand-control-text">Expand source</span></span> <span
    class="collapse-spinner-wrapper"></span>

    ``` syntaxhighlighter-pre
    [Service]
    Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
    Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true"
    Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
    Environment="KUBELET_DNS_ARGS=--cluster-dns=10.96.0.10 --cluster-domain=cluster.local"
    Environment="KUBELET_AUTHZ_ARGS=--authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt"
    Environment="KUBELET_CADVISOR_ARGS=--cadvisor-port=0"
    Environment="KUBELET_CERTIFICATE_ARGS=--rotate-certificates=true --cert-dir=/var/lib/kubelet/pki"
    Environment="KUBELET_EXTRA_ARGS=--feature-gates=DevicePlugins=true"
    ExecStart=
    ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CADVISOR_ARGS $KUBELET_CERTIFICATE_ARGS $KUBELET_EXTRA_ARGS
    ```

7.  On the GPU node, reload and restart kubelet to apply previous
    changes to the configuration.

    ``` syntaxhighlighter-pre
    sudo systemctl daemon-reload
    sudo systemctl restart kubelet
    ```

8.  If not already done, enable GPU support on the Kubernetes master by
    deploying following Daemonset.

    ``` syntaxhighlighter-pre
    kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.9/nvidia-device-plugin.yml
    ```

9.  For the simplicity, generate a new token on the Kubernetes master
    and print the join command.

    ``` syntaxhighlighter-pre
    ubuntu@virutal-kubernetes-1:~$ sudo kubeadm token create --print-join-command
    kubeadm join --token 6e112b.a598ccc2e90671a6 KUBERNETES_MASTER_IP:6443 --discovery-token-ca-cert-hash sha256:863250f81355e64074cedf5e3486af32253e394e939f4b03562e4ec87707de0a
    ```

10. Go back to the GPU node and use the printed join command to add GPU
    node into the cluster.

    ``` syntaxhighlighter-pre
    ubuntu@virtual-kubernetes-gpu-2:~$ sudo kubeadm join --token 6e112b.a598ccc2e90671a6 KUBERNETES_MASTER_IP:6443 --discovery-token-ca-cert-hash sha256:863250f81355e64074cedf5e3486af32253e394e939f4b03562e4ec87707de0a
    [preflight] Running pre-flight checks.
        [WARNING FileExisting-crictl]: crictl not found in system path
    [discovery] Trying to connect to API Server "KUBERNETES_MASTER_IP:6443"
    [discovery] Created cluster-info discovery client, requesting info from "https://KUBERNETES_MASTER_IP:6443"
    [discovery] Requesting info from "https://KUBERNETES_MASTER_IP:6443" again to validate TLS against the pinned public key
    [discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "KUBERNETES_MASTER_IP:6443"
    [discovery] Successfully established connection with API Server "KUBERNETES_MASTER_IP:6443"

    This node has joined the cluster:
    * Certificate signing request was sent to master and a response
      was received.
    * The Kubelet was informed of the new secure connection details.

    Run 'kubectl get nodes' on the master to see this node join the cluster.
    ```

11. Run following command to see the GPU node (virtual-kubernetes-gpu-2)
    status on the cluster.

    ``` syntaxhighlighter-pre
    ubuntu@virutal-kubernetes-1:~$ kubectl get nodes
    NAME                       STATUS     ROLES     AGE       VERSION
    virtual-kubernetes-gpu     Ready      <none>    1d        v1.9.4
    virtual-kubernetes-gpu-2   NotReady   <none>    13s       v1.9.4
    virutal-kubernetes-1       Ready      master    5d        v1.9.4
    virutal-kubernetes-2       Ready      <none>    5d        v1.9.4
    virutal-kubernetes-3       Ready      <none>    5d        v1.9.4
    ```

12. After a while, the node is ready.

    ``` syntaxhighlighter-pre
    virtual-kubernetes-gpu-2   Ready     <none>    7m        v1.9.4
    ```

13. Now we have 2 GPU nodes ready in our Kubernetes cluster. We can
    label the recently added node (virtual-kubernetes-gpu-2) with the
    accelerator type by running following command on the master.

    ``` syntaxhighlighter-pre
    kubectl label nodes virtual-kubernetes-gpu-2 accelerator=nvidia-tesla-k20m
    ```

14. To check nodes for accelerator label, run <span
    style="font-family: monospace;">kubectl get nodes -L
    accelerator</span> on Kubernetes master.

    ``` syntaxhighlighter-pre
    ubuntu@virutal-kubernetes-1:~/kubernetes$ kubectl get nodes -L accelerator
    NAME                       STATUS    ROLES     AGE       VERSION   ACCELERATOR
    virtual-kubernetes-gpu     Ready     <none>    1d        v1.9.4    nvidia-tesla-k20m
    virtual-kubernetes-gpu-2   Ready     <none>    24m       v1.9.4    nvidia-tesla-k20m
    virutal-kubernetes-1       Ready     master    5d        v1.9.4    
    virutal-kubernetes-2       Ready     <none>    5d        v1.9.4    
    virutal-kubernetes-3       Ready     <none>    5d        v1.9.4    
    ```

15. To test the GPU nodes, go to the master and create a file with the
    following content and execute it.

    **gpu-test.yml**

    ``` syntaxhighlighter-pre
    apiVersion: v1
    kind: Pod
    metadata:
      name: cuda-vector-add
    spec:
      restartPolicy: OnFailure
      containers:
        - name: cuda-vector-add
          # https://github.com/kubernetes/kubernetes/blob/v1.7.11/test/images/nvidia-cuda/Dockerfile
          image: "k8s.gcr.io/cuda-vector-add:v0.1"
          resources:
            limits:
              nvidia.com/gpu: 1 # requesting 1 GPU per container
      nodeSelector:
        accelerator: nvidia-tesla-k20m # or nvidia-tesla-k80 etc.
    ```

    ``` syntaxhighlighter-pre
    ubuntu@virutal-kubernetes-1:~/kubernetes$ kubectl create -f gpu-test.yml
    pod "cuda-vector-add" created
    ubuntu@virutal-kubernetes-1:~/kubernetes$ kubectl get pods -a
    NAME                            READY     STATUS      RESTARTS   AGE
    cuda-vector-add                 0/1       Completed   0          19s
    ```

