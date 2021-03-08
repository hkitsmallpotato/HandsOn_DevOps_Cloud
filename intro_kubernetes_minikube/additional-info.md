# Additional Info

A sort of sketch pad at this moment.

## Mounting volume in minikube

At various point you may want to feed some files into the pod/container. The main way to do this is by mounting volume. In docker this is easily done, while in k8s it is a bit more complicated.

Referring to the document, two way to do it are by using the [ConfigMap](https://kubernetes.io/docs/concepts/storage/volumes/#configmap) and the [hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) type. For the host path type, there is an additional catch in minikube: the "host" here refers to the VM/docker container etc housing your minikube k8s cluster instance, *not* your computer! Therefore, to use that type require one more step: mount the real host into some path inside that VM/docker first, then mount a path inside that VM/docker into the pod.

(Reference: https://dev.to/coherentlogic/learn-how-to-mount-a-local-drive-in-a-pod-in-minikube-2020-3j48)

To do this, supply the following arguments when starting minikube (For Google Cloud Shell, adding mount for a running minikube instance is not supported):

```bash
minikube start --mount --mount-string="<hostpath>:<mountpath>"
```

## Copying Files between pods and host

It is well known that there is a command for printing log of a pod, and for ssh-ing into the pod. What is perhaps less well known is that there is an analogue of `scp` as well.

```bash
kubectl cp /path/to/file my-pod:/path/to/file
```

(It works both way by exchanging position)

## (Meta) Template value in helm

In bitnami, some values are themselve of type yaml template. In those case please use the argument flag `--set-file` instead of `--set`, and supply an actual yaml file.

**Advanced**: Bitnami used their own template function instead of plain `toYaml` function - this is to ensure it support yaml _template_ and not just yaml values.

## Ingress for bitnami

To configure ingress, use this reference setting:

```bash
... --set ingress.enabled=true \
--set ingress.annotations='nginx.ingress.kubernetes.io/upstream-vhost: "example.com"'
```

Note that the inner quote is needed.
