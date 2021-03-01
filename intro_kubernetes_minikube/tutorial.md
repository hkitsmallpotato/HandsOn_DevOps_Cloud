# Introduction to Kubernetes via Minikube

## Overview

In this tutorial, we will introduce you on how to integrate Kubernetes into your software development workflow. You will use a local development enviornment through Minikube.

TODO: List what you will learn

**Time Needed**: About 30 minutes

**Difficulty**: Introductory


## What is Kubernetes?

TODO

[Kubernetes](https://kubernetes.io/) (Also called k8s) is the dominant *container orchestration platform*.

[Minikubes](https://minikube.sigs.k8s.io/docs/) is a tool to make local development with k8s easier, by making it possible to quickly and easily spin up a local k8s cluster.

## Bootup a local minikube cluster

`minikube` is already installed by default on Google Cloud Shell. First, let's check the status by entering the following command into the console:

```bash
minikube status
```

You should see the following output:

```
* Profile "minikube" not found. Run "minikube profile list" to view all profiles.
  - To start a cluster, run: "minikube start"
```

However, we will not do that yet. There are many configurations available, and we will focus on the most relevant ones, which we examine next.

### Minikube Driver

Minikube need a way to start a k8s cluster, and there are many ways to do so, each with their pros and cons. Let's see our options:

```bash
minikube config defaults driver
```

You should see

```
* virtualbox
* vmwarefusion
* kvm2
* vmware
* none
* docker
* podman
* ssh
```

The VM based option starts a VM inside your computer, then install k8s as usual inside the VM. This provides good isolation and can faithfully replicate production enviornment, but is not usuable in our case as Google Cloud Shell runs inside a restricted docker container. Therefore, we should choose `docker` (which is the default).

### Memory, and starting up

By default, `minikube` starts our cluster with 2200MB of memory. As we may want to deploy applications of moderate complexities, we will change it via the `--memory` parameter. Start the local cluster with:

```bash
minikube start --cpus 2 --memory 4400 --driver docker
```

Be patient and wait, and it should eventually boot up with this log:

```
* minikube v1.17.1 on Debian 10.8 (amd64)
  - MINIKUBE_FORCE_SYSTEMD=true
  - MINIKUBE_HOME=/google/minikube
  - MINIKUBE_WANTUPDATENOTIFICATION=false
* Using the docker driver based on user configuration
* Starting control plane node minikube in cluster minikube
* Pulling base image ...
* Downloading Kubernetes v1.20.2 preload ...
    > preloaded-images-k8s-v8-v1....: 491.22 MiB / 491.22 MiB  100.00% 75.71 Mi
* Creating docker container (CPUs=2, Memory=4400MB) ...
* Preparing Kubernetes v1.20.2 on Docker 20.10.2 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Verifying Kubernetes components...
* Enabled addons: default-storageclass, storage-provisioner
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Check our status again:

```bash
minikube status
```

```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
timeToStop: Nonexistent
```

## The client command line

`kubectl` is a *client side* command line for performing operations on any k8s cluster. A conformant k8s cluster is required by spec to expose a set of API, and what the client does is to simply issue the appropiate API call and process the response.

Try this now. Check the composition of our local cluster:

```bash
kubectl get nodes
```

The result shows that it consists of a single node, as expected:

```
NAME       STATUS   ROLES                  AGE   VERSION
minikube   Ready    control-plane,master   30s   v1.20.2
```

Next, check the *deployment* and *pods*:

```bash
kubectl get deployment -A
```

```
NAMESPACE     NAME      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   coredns   1/1     1            1           46s
```

```bash
kubectl get pods -A
```

```
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-jgkpt            1/1     Running   0          40s
kube-system   etcd-minikube                      0/1     Running   0          53s
kube-system   kube-apiserver-minikube            1/1     Running   0          53s
kube-system   kube-controller-manager-minikube   0/1     Running   0          53s
kube-system   kube-proxy-4qnfh                   1/1     Running   0          40s
kube-system   kube-scheduler-minikube            0/1     Running   0          53s
kube-system   storage-provisioner                1/1     Running   0          53s
```

The `-A` option is shorthand for `--all-namespaces`. The `coredns` deployment is a basic service of k8s - it is necessary for the k8s's **standard networking model**. You also see that some pods are not ready yet (Indicated by `0/1` in the `READY` column). Don't worry - k8s is designed to be self-healing, and you can continue even when it is technically "deploying" - k8s will automatically manages everything and ensure that eventually, everything is up and running.

## Deploy our first application

We will follow the [tutorial](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/) on the official k8s documentation.

### Manual deployment

A k8s resource is defined in a declarative, *Infrastructure as Code* style, using the yaml format. The command to deploy any resources is `kubectl apply -f {filename/url}`

We gather all the resource files in that tutorial, and apply them one by one:

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/mongo-deployment.yaml
```

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/mongo-service.yaml
```

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-deployment.yaml
```

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-service.yaml
```

You can use the command in the last steps to check the status of the deployment.

### Port forwarding

Networking in k8s is a big topic, which we will not explore in depth here. Instead, we will just show some common operations associated with them.

Check the *service* we have:

```bash
kubectl get services
```

We can access the application during development, by doing a **port-forward**:

```bash
kubectl port-forward svc/frontend 8080:80
```

Use the web preview feature of Google Cloud Shell to see the result. Click <walkthrough-web-preview-icon></walkthrough-web-preview-icon> <walkthrough-spotlight-pointer cssSelector="button[spotlightid=cloud-shell-web-preview-button]">Web Preview</walkthrough-spotlight-pointer> -> "Preview on port 8080".

When you are done, press `Ctrl-C` to stop the port-forwarding process.

## Install k8s dashboard


## Using helm to deploy common apps


## (Optional) Connecting to a remote cluster


## Cleanup


## Congratulations!

