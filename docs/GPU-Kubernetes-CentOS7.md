# Installing and testing GPU Node in Kubernetes - CentOS7

## Introduction

The manual procedure for installation and configuration od a Kubernetes cluster is provided. The cluster is composed by a Master node and one Worker node

## Cluster Status

Cluster Name: KubeDeep

Kubectl components
-- add table

Worker GPU specifications
-- add table

## Tests
### Test #1 - Simple vector-add (CUDA8)
This CUDA Runtime API sample is a very basic sample that implements element by element vector addition.
The examples uses CUDA8 driver.

### Test #2 - Simple vector-add with different CUDA driver
This CUDA Runtime API sample is a very basic sample that implements element by element vector addition.
The examples uses two Docker images with different version of CUDA driver.
To complete the test, a new Docker image with CUDA driver version 9 has been built and uploaded in a private repo.

### Test #3 - Simple BinomialOption with different CUDA driver
This sample evaluates fair call price for a given set of European options under binomial model.
To complete the test, two new Docker images with CUDA8 and CUDA9 has been built and uploaded in a private repo.
The test will take some seconds and GPU engage can be shown

### Test #4 - Job Scheduling features

Tests highlithing the features of the Kubernetes scheduler.
Default schedule policies are used (FIFO).

Submission of a bunch of different cuda jobs with different running time.
* Parrec (1h)
* Cuda8-binomialoption.yaml (5 min)
* Cuda9-binomialoption.yaml (5 min)
* Cuda8-vector-add.yaml (few sec)
* Cuda9-vector-add.yaml (few sec)

The parrec job has been launched as first job. One GPU has been engaged by the job; the other is still available for other jobs.

Other jobs have been submitted in the following order:

The “cuda8-binomialoption” is running, the other are in the FIFO queue in pending state. After completion, the other job will be running in the same order they have been submitted.

#Access PODs from outside the cluster

To access PODs from outside the cluster it can be possible following different procedures that strictly depend on the usecase and (cloud) providers.

NodePort, hostNetwork, hostPort, LoadBalancer and Ingress features of Kubernetes can be adopted as described in the following Reference:

http://alesnosek.com/blog/2017/02/14/accessing-kubernetes-pods-from-outside-of-the-cluster/

For example puroposes, the Test #5 example will describe and use the NodePort procedure as the cluster is defined as 1 Master and 1 Worker both with routable IPs.

## Test #5 - Long running service with NodePort

Prerequisites

1. Kubernetes Node with routable IP
2. Port range dynamically selected as from the "kubernetes-apiservice.service" configuration file
3. Nginx replica 2; V. 1.13.12 - latest (as from the YAML files)

Yaml files related to nginx deployment and nginx service


# References

The following guides have been followed for the Installation and Configuration of Kubernetes cluster - a detailed step-by-step guide will be provided soon
Kubernetes Installation/Configuration docs

https://github.com/kelseyhightower/kubernetes-the-hard-way
* GPU Node - CentOS7.4
* Install docker

https://docs.docker.com/install/linux/docker-ce/centos/#set-up-the-repository
* Driver nvidia-install repo

https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=CentOS&target_version=7&target_type=rpmnetwork
* Driver Nvidia-install driver

http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#post-installation-actions
* Install NVIDIA-docker repo

https://nvidia.github.io/nvidia-docker/
* Install-NVIDIA docker

https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0)#prerequisites
