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

[Helm](https://helm.sh/) is both a templating system to make complex kubernetes deployment with long yaml files (you will see this later) manageable, as well as a package manager system for kubernetes.

Hint: Helm v2.x is different from Helm v3.x - the old version requires you to deploy a special component, called `Tiller`, into the cluster. This occupies some server resources, and may be a concern for those on a tight budget. Fortunately, this is no longer necessary in the new version. More confusingly, some installation procedures for important software in the k8s ecosystem may behave differently on these 2 verions. Google Cloud Shell comes pre-installed with Helm v3.x, and as it is the new version, you mostly need not worry about the version difference, as long as you stick to instruction for Helm v3.x on their official doc.

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

There is an [official dashboard](https://github.com/kubernetes/dashboard) for kubernetes, which let you more easily manage and monitor a k8s cluster using a UI. It is not installed by default, and there are different ways to install it: manual yaml deployment, helm charts, or minikube addon.

### Manual install

This method is recommended as it is applicable to real cluster as well. Apply the recommended yaml file:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```

This will create a new *namespace* named `kubernetes-dashboard`, and create a bunch of resources:

```
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

A namespace is a way to isolate workloads to avoid name conflict. This makes it more manageable as well.

For security reason, the dashboard should not be exposed to public - we will use networking method that only expose it internally. Turns out there is a special command for the dashboard: Open a new terminal, and type:

```bash
kubectl proxy
```

It will output

```
Starting to serve on 127.0.0.1:8001
```

and then wait for connection (do **not** close the terminal or Ctrl-C until you're done using it)

The URL locally is:

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

For Google Cloud Shell user, use the web preview feature again. Click <walkthrough-web-preview-icon></walkthrough-web-preview-icon> <walkthrough-spotlight-pointer cssSelector="button[spotlightid=cloud-shell-web-preview-button]">Web Preview</walkthrough-spotlight-pointer> -> "Change port", then enter "8001". In the new browser window, edit the URL and append the path portion of the URL above - i.e. `api/v1/namespaces/...` (while removing the query parameter `?authuser...`). You should see the screen below:

TODO (screenshot)

We need to create user and get the secret token to login. Following the [official guide](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md), the additional yaml files have been prepared and are available in this directory.

```bash
kubectl apply -f admin-user.yaml
```

```bash
kubectl apply -f role-binding.yaml
```

Output:

```
serviceaccount/admin-user created
```

```
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

Then run:

```bash
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

and copy the token into the login screen.

After login, click the box on the top, and select "All namespaces". This will show the full info across all namespaces.

### Minikube addon

Often in development, you need to repeatedly create fresh new k8s clusters. If you are using minikube, you can skip all of the above and get one quickly simply by typing this:

```bash
minikube dashboard
```

The output would be like:

```
* Enabling dashboard ...
  - Using image kubernetesui/dashboard:v2.1.0
  - Using image kubernetesui/metrics-scraper:v1.0.4
* Verifying dashboard health ...
* Launching proxy ...
* Verifying proxy health ...
* Opening http://127.0.0.1:34005/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
  - http://127.0.0.1:34005/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

Simply open web preview as above, changing the port number to what is indicated in the output (in the example above, it is 34005). Then, append the path portion of the URL above (`/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/`)

## Install minikube ingress addon

To make life easier during development, minikube support the use of addons to simplifies installation of some common services. Here, we will follow the guide for the [ingress addon](https://minikube.sigs.k8s.io/docs/tutorials/nginx_tcp_udp_ingress/).

Run the command:

```bash
minikube addons enable ingress
```

```
* Verifying ingress addon...
* The 'ingress' addon is enabled
```

For convinience, the test deployment files have beeb prepared. Apply them:

```bash
kubectl apply -f ingress-addon-test.yaml
```

**Hint**: Multiple k8s resources can be specified in the same file - just separate them with a line consisting of exactly three dashes (`---`).

After that, check our services:

```bash
kubectl get service -A
```

```
NAMESPACE     NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes                           ClusterIP   10.96.0.1        <none>        443/TCP                  11m
default       redis-service                        ClusterIP   10.110.192.165   <none>        6379/TCP                 9m7s
kube-system   ingress-nginx-controller-admission   ClusterIP   10.110.138.127   <none>        443/TCP                  7m34s
kube-system   kube-dns                             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   11m
```

(Sample only - note the `ingress-nginx-controller-admission` service in the `kube-system` namespace)

Now, run this command:

```bash
kubectl patch configmap tcp-services -n kube-system --patch '{"data":{"6379":"default/redis-service:6379"}}'
```

What it does is to update the `ConfigMap` resource type, which records the routing data of our nginx ingress. (You will learn more about ingress in subsequent hands on lab) In particular, we added this mapping:

```
"6379":"default/redis-service:6379"
```

Which says to route requests coming from port 6379, into the service `redis-service` in the `default` namespace, that is listening on port 6379.

You can optionally run the check as suggested by the official guide:

```bash
kubectl get configmap tcp-services -n kube-system -o yaml
```

Finally, we need the following step:

```bash
kubectl patch deployment ingress-nginx-controller --patch "$(cat ingress-nginx-controller-patch.yaml)" -n kube-system
```

The reason is that the ingress is implemented by an nginx web server, which resides in a pod. (It is what backs the `ingress-nginx-controller-admission` service you see above) Having everything configured correctly would be useless, if this nginx server that does the routing, is itself not exposed for external access! The command above changes it so that the port 6379 on the outside (`hostPort`) is connected with the same port inside the pod (`containerPort`). So the whole chain of event when we make a request, is: host@6379 -> nginx@6379 -> read off routing map, should route to `default/redis-service`@6379.

We can now test our setup. As redis works over a TCP protocol, and accept simple text command such as `hello`, `ping`, `set <key> <value>`, and `get <key>` directly, we can even use them to try out redis on the side:

```
telnet $(minikube ip) 6379
Trying 192.168.49.2...
Connected to 192.168.49.2.
Escape character is '^]'.

hello
*14
$6
server
$5
redis
$7
version
$5
6.2.1
$5
proto
:2
$2
id
:3
$4
mode
$10
standalone
$4
role
$6
master
$7
modules
*0
ping
+PONG
set foo 2
+OK
get foo
$1
2
quit
+OK
Connection closed by foreign host.
```

## Using helm to deploy common apps

**Warning**: Bitnami helm charts are of production strength and carry significant complexities beyond the bare minimum needed. In particular since the usual deployment scenario is on a public facing VM with public IP and cloud load balancer, deploying them in development enviornment can ranges from needing a few fiddling to very difficult. For example, both wordpress and joomla have absolute URL that depends on recognizing the hosts from HTTP request header, and so setting them up behind reverse proxy (like in our case) is hard to pull off. Refer to `additional-info.md` in this directory for something more.

Let's try deploying common applications onto k8s using helm charts. A popular provider is bitnami, which provide production grade images in various format as well, such as docker image and VM image file. Here we use [Drupal](https://bitnami.com/stack/drupal/helm) as an example.

First add bitnami as a helm chart repository and update it:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

```
"bitnami" has been added to your repositories
```

```bash
helm repo update
```

```
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```

We need to customize the deployment parameters - in particular, check the [section for exposure](https://github.com/bitnami/charts/tree/master/bitnami/drupal/#traffic-exposure-parameters). The default value for the k8s *Service Type* is *Load Balancer*, which is not available in any bare metal settings in general. (In the world of k8s, Load Balancer general refers to cloud-provisioned one - in particular it must be possible to provision them on the fly via an API, that may differ across vendors. However, there is project such as [MetalLB](https://metallb.universe.tf/) that aims to solve this problem directly)

In our case, we will use `NodePort` for convinience.

Create a new namespace for isolation as usual:

```bash
kubectl create ns dr
```

```
namespace/dr created
```

Install with our own parameters:

```bash
helm install -n dr test2 bitnami/drupal --set service.type=NodePort
```

Output:

```
NAME: test2
LAST DEPLOYED: Sun Mar  7 13:22:58 2021
NAMESPACE: dr
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
*******************************************************************
*** PLEASE BE PATIENT: Drupal may take a few minutes to install ***
*******************************************************************

1. Get the Drupal URL:

  Or running:

  export NODE_PORT=$(kubectl get --namespace dr -o jsonpath="{.spec.ports[0].nodePort}" services test2-drupal)
  export NODE_IP=$(kubectl get nodes --namespace dr -o jsonpath="{.items[0].status.addresses[0].address}")
  echo "Drupal URL: http://$NODE_IP:$NODE_PORT/"

2. Get your Drupal login credentials by running:

  echo Username: user
  echo Password: $(kubectl get secret --namespace dr test2-drupal -o jsonpath="{.data.drupal-password}" | base64 --decode)

```

Remember that it has only constructed the resources - the actual containers running it takes time to start. Check the progress:

```bash
kubectl get pods -n dr
```

Until it shows:

```
NAME                            READY   STATUS    RESTARTS   AGE
test2-mariadb-0                 1/1     Running   0          105s
test2-drupal-67dbfd44fb-48r2c   1/1     Running   0          106s
```

Do the usual port forwarding:

```bash
kubectl port-forward -n dr svc/test2-drupal 9102:80
```

Then open web preview similarly on port 9102. You should see the Drupal webpage. Follow the instruction in the helm deployment above to display the initial admin password.


## (Optional) Connecting to a remote cluster

Often, you want to connect to a remote k8s cluster and perform operation on it using your local computer. We demo this below.

First, obtain the kubernetes config file from your cluster provider (this varies based on the vendor, e.g. [this](https://stackoverflow.com/questions/61829214/how-to-export-kubeconfig-file-from-existing-cluster#61829308) ).

Then, export the enviornmental variable:

```bash
export KUBECONFIG=<path to your file>
```

Then, when you use the `kubectl` command line, you will be operating against that cluster.

**Hint**: It is possible to specify more than one remote cluster in the config file, or to have multiple config files, one for each remote cluster. In those cases you must specify which cluster you mean, by the command line argument `--context=<cluster name>`. See [this](https://ahmet.im/blog/mastering-kubeconfig/) for similar hints.

## Cleanup

To uninstall a helm deployment:

```bash
helm uninstall <release name> -n <namespace name>
```

To delete a kubernetes namespace:

```bash
kubectl delete ns <namespace name>
```

To stop minikube:

```bash
minikube stop
```

```
* Stopping node "minikube"  ...
* Powering off "minikube" via SSH ...
* 1 nodes stopped.
```

The data is still preserved though. Once you restart minikube you should find that the original deployments before shutdown are back.

On the other hand, if you want to completely teardown the cluster, so that you get a clean/fresh one on the next restart:

```bash
minikube delete
```

```
* Deleting "minikube" in docker ...
* Deleting container "minikube" ...
* Removing /google/minikube/.minikube/machines/minikube ...
* Removed all traces of the "minikube" cluster.
```

## Congratulations!

### Reference

https://stackoverflow.com/questions/59164010/how-to-map-localhost-to-minikube-with-ingress

https://github.com/kubernetes/minikube/issues/9116

https://gist.github.com/dockerlead/18696732e8ecb1a97772911e9a0d4637


