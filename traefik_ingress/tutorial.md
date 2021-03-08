# Traefik for Ingress

## Overview

In this tutorial, you will install Traefik, and configure it as an ingress for k8s service.

**Time needed**: Around 25 minutes

**Difficulty**: Intermediate

## Background

### Kubernetes concepts

In k8s, an [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) is an abstract resource type that expresses the intention, or desire, to expose a service for external access from outside the cluster. It is usually fulfilled by instantiating a corresponding _ingress controller_. In informal discussion this distinction is often elided, and the word "ingress" is used to refer to both concepts simultaneously.

A [controller](https://kubernetes.io/docs/concepts/architecture/controller/) is a class of program in k8s that are responsible for performing the needed operation on the cluster in response to user specified Infrastructure as Code. They achieve this by using the _synchronization loop pattern_, which is inspired by [PID controller](https://en.wikipedia.org/wiki/PID_controller) in control theory. In that example, the current system state is compared against the desired system state, and some function of this error is used to drive the system. The hope is that by designing a suitable _negative feedback_, the system will tend toward the equilibrium point, that is, when the system state is same as the desired state. In our case, it is the _cluster state_, such as the number and kinds of containers instantiated on various nodes, the routing configurations, etc, that is being controlled in this manner by the action of a controller. By using this pattern, the end user can take a declarative view of the cluster while the controller manages the potentially messy procedures underneath.

An ingress controller is then a particular kind of k8s controller, one that is responsible for maintaining the correct ingress state.

### Traefik as an ingress



## Initial Install of Traefik

We will use the helm chart method from the [official doc](https://doc.traefik.io/traefik/v2.2/getting-started/install-traefik/). Notice that version 1.x of Traefik is considered legacy, and so we will use Traefik v2.x. Also remember the usual note about helm 2 vs helm 3. In Google Cloud Shell, helm 3 is installed and enabled by default, so we need not worry.

Hint: Remember to start the minikube cluster first.

First, add the chart repo and update it:

```bash
helm repo add traefik https://traefik.github.io/traefik-helm-chart
```

```
"traefik" has been added to your repositories
```

```bash
helm repo update
```

```
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "traefik" chart repository
Update Complete. ⎈Happy Helming!⎈
```

To make it easier to manage (and undo if you mess up), create a dedicated namespace for it:

```bash
kubectl create ns traefik-v2
```

```
namespace/traefik-v2 created
```

Peform a plain install:

```bash
helm install --namespace=traefik-v2 \
     traefik traefik/traefik
```

```
W0306 07:32:16.057948    8060 warnings.go:70] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
...(snipped)...
NAME: traefik
LAST DEPLOYED: Sat Mar  6 07:32:18 2021
NAMESPACE: traefik-v2
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

After waiting for the deployment to become ready, we expose Traefik's internal dashboard:

```bash
kubectl port-forward -n traefik-v2 $(kubectl get pods -n traefik-v2 --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
```

```
Forwarding from 127.0.0.1:9000 -> 9000
```

Using Google Cloud Shell's Web Preview, open port 9000 in a new browser window. You should see the v2.x dashboard as follows:

TODO (Screenshot)

## Updating Traefik with Static Configuration

To make it usable, we have to configure it. Our detailed configuration is primarily from this [blog post](https://traefik.io/blog/install-and-configure-traefik-with-helm/), with help from extensively reading the official doc. For simplicity, in this tutorial we use a customized config that is stripped down to the bare bone to illustrate the principles.

### Detailed Explanation

We need to supply the static configuration. In our case, we only enable the k8s ingress provider. This provider perform dynamic configuration by reading from the cluster's ingress resource data and use that as the desired state, that is, traefik acts as an ingress controller in the k8s sense. However, Traefik is more powerful than the minimal requirement and provide more advanced functionalities that you can turn on as needed.

There are many ways to supply a static configuration. One of the way is by having an actual config file inside the Traefik pod. (Traefik supports multiple formats for that file, and Traefik do support other ways of doing static config, but for consistency, we use yaml format and an honest config file.) Our static config is then as follows:

```yaml
providers:
  kubernetesIngress:
    namespaces:
      - "default"
      - "test-traefik"
    labelselector: "routes = true"
```

What happens is that we instruct Traefik that for the k8s ingress provider, it should only manages the `default` and the `test-traefik` namespaces. Moreover, we provide a way for user to exert fine grained control by imposing that Traefik should only manages those ingress resources that contain a metadata entry of label type, whose key-value pair is `routes: true`.

We still need a way to "feed" this file into the pod. To do this, we do two things:

1. Declare a k8s `ConfigMap` resource type to save the data of this config file.
2. Mount the config file as an volume into the pod, and supply the path of the config file to Traefik.

Step 1 is done by a plain k8s manifest, while step 2 is done by a [Helm chart values](https://helm.sh/docs/chart_template_guide/values_files/) file. We used `additonalArguments`, a _Helm chart value_, to supply the command line argument that are used when the container starts Traefik. By contrast, the command line argument we used for the `traefik` binary is `--providers.file.filename`.

### Execution

For convinience, the files have been prepared under the directory of this tutorial.

First create the `ConfigMap` resource:

```bash
kubectl apply -f traefik-config.yaml
```

```
configmap/traefik-config created
```

Then update the helm installation:

```bash
helm upgrade --cleanup-on-fail traefik traefik/traefik -n traefik-v2 -f traefik-chart-values.yaml
```

```
Release "traefik" has been upgraded. Happy Helming!
NAME: traefik
LAST DEPLOYED: Sat Mar  6 08:04:43 2021
NAMESPACE: traefik-v2
STATUS: deployed
REVISION: 2
TEST SUITE: None
```

(Notice that what we did is actually to override the default chart values. `--set` override individual values directly on the command line, while `-f` uses a file.)

## Testing Traefik Basic Routing

Now we test that the installation works. We have prepared a sample deployment manifest in this directory. For better isolation, this test will be done within the `test-traefik` namespace. Let's create the namespace first:

```bash
kubectl create ns test-traefik
```

Then apply the deployment and the service:

```bash
kubectl apply -f whoami-deployment.yaml
```

```bash
kubectl apply -f whoami-service.yaml
```

We are using the [test container](https://github.com/traefik/whoami) provided by Traefik, whose docker image name is `traefik/whoami`.

Let's inspect the ingress file. In the metadata section:

```yaml
metadata:
  name: "foo"
  namespace: test-traefik
  labels:
    routes: true
```

Notice that we need to have the `routes: true` key to enable management by Traefik due to our custom designed static config.

For the matching rules, notice that there is `path` argument but no `host` argument. This is deliberate as we are behind a reverse proxy in web preview mode, so the host will be `127.0.0.1:port`. Unfortunately, k8s ingress's host field only support real domain name (including [some support](https://github.com/traefik/traefik/issues/792) for wildcard domain). Hence, we omit the field entirely, so the matching rule will match any/all hosts.

Finally, let's apply the ingress:

```bash
kubectl apply -f whoami-ingress.yaml
```

### Verification

To actually tests it, we need to learn one more concept in Traefik: Entrypoint. Traefik centralizes routes into a single entry point, which is a specific port on the main pod hosting Traefik. This port number can be examined in the Traefik dashboard's home page. By default, it should be 8000. Therefore, we run this command:

```bash
kubectl port-forward -n traefik-v2 $(kubectl get pods -n traefik-v2 --selector "app.kubernetes.io/name=traefik" --output=name) 8000:8000
```

(That is, the same command as that used to expose the dashboard, but ports changed to `8000:8000`)

Then, using web preview on port `8000`, with the URL path being `/foo` or `/bar` as our ingress configured, should take you to the `whoami` pod with the following output:

```
Hostname: whoami-deployment-8557b59f65-tc5hj
IP: 127.0.0.1
IP: 172.17.0.6
RemoteAddr: 172.17.0.7:44830
GET /foo HTTP/1.1
Host: 127.0.0.1:8000
User-Agent: Mozilla/5.0...(snipped)
Accept: text/html,application/xhtml+xml,...(snipped)
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;...(snipped)
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";...(snipped)
Sec-Ch-Ua-Mobile: ?0
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Sec-Gpc: 1
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: 127.0.0.1:8000
X-Forwarded-Port: 8000
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-c47c6c548-xvhvt
X-Real-Ip: 127.0.0.1
```

## Congratulations!

