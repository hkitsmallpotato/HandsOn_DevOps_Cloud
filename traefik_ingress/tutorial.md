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

