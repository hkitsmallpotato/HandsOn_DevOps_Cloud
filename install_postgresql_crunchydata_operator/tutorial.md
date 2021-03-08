# Install PostgreSQL vis CrunchyData Operator

## Overview

In this tutorial, you will install PostgreSQL on a minikube k8s cluster by using the CrunchyData operator.

**Time needed**: Around 20 - 30 minutes

**Difficulty**: Easy

## Introduction

PostgreSQL is a popular open source database.

A kubernetes operator is a powerful way to install applications onto the cluster. Whereas a normal deployment operators at a user level and directly creates various resources such as deployment, services, pods, an operator creates a *Custom Resource Definition (CRD)*. This afford them great flexibility and power - usually, an operator can not only perform complex deployment automatically, it can also deal with upgrading existing software seamlessly, which is a common aspect of what has come to be called *Day 2 operation*.

In this tutorial, we will be using the [official guide](https://access.crunchydata.com/documentation/postgres-operator/4.6.1/quickstart/) of the CrunchyData operator.

## Install the operator

Run the following command:

```bash
kubectl create namespace pgo
```

```
namespace/pgo created
```

```bash
kubectl apply -f https://raw.githubusercontent.com/CrunchyData/postgres-operator/v4.6.1/installers/kubectl/postgres-operator.yml
```

```
serviceaccount/pgo-deployer-sa created
clusterrole.rbac.authorization.k8s.io/pgo-deployer-cr created
configmap/pgo-deployer-cm created
clusterrolebinding.rbac.authorization.k8s.io/pgo-deployer-crb created
job.batch/pgo-deploy created
```

This creates a *Job*, which is a one-off program to perform the installation steps. It is implemented by a Pod that terminates when it is *completed* (and won't be restarted/recreated)

The manual specifies that the name for this pod is `pgo-deployer`.

Hint: In more restrictive hosts (such as when your k8s is provided inside a docker container), the filesystem you have access to may not support enough operations for the default config to work. The most common scenario in previous version is that the default storage class is set to `ReadWriteMany` when only `ReadWriteOnce` is supported. (Read the whole of the "Storage Settings" section in the [config reference](https://access.crunchydata.com/documentation/postgres-operator/4.6.1/installation/configuration/) )

## Install the pgo client

Download and run the client installation script:

```bash
curl https://raw.githubusercontent.com/CrunchyData/postgres-operator/v4.6.1/installers/kubectl/client-setup.sh > client-setup.sh && chmod +x client-setup.sh && ./client-setup.sh
```

```
Operating System found is Linux...
Downloading pgo version: v4.6.1...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   609  100   609    0     0   1534      0 --:--:-- --:--:-- --:--:--  1530
100 38.3M  100 38.3M    0     0  14.3M      0  0:00:02  0:00:02 --:--:-- 20.2M
pgo client files have been generated, please add the following to your bashrc
export PATH=/home/xxx/.pgo/pgo:$PATH
export PGOUSER=/home/xxx/.pgo/pgo/pgouser
export PGO_CA_CERT=/home/xxx/.pgo/pgo/client.crt
export PGO_CLIENT_CERT=/home/xxx/.pgo/pgo/client.crt
export PGO_CLIENT_KEY=/home/xxx/.pgo/pgo/client.key
```

Follow the instruction in the output and add the configuration enviornmental variables.

To actually use the client, we will need to expose the underlying server: in a new terminal, type:

```bash
kubectl -n pgo port-forward svc/postgres-operator 8443:8443
```

Then check we can connect:

```bash
pgo version
```

```
pgo client version 4.6.1
pgo-apiserver version 4.6.1
```


## Initial Setup using the pgo client

First, create a cluster:

```bash
pgo create cluster hippo
```

```
created cluster: hippo
workflow id: d904d8f0-2ec2-41ea-987c-ea439e2a7ef3
database name: hippo
users:
        username: testuser password: xxtzxlsD}Lb_,y_{Jg)Rh3Km
```

It is actually provisioning the cluster behind the scene. Wait until the following check passes:

```bash
pgo test -n pgo hippo
```

```
cluster : hippo
        Services
                primary (10.99.25.17:5432): UP
        Instances
                primary (hippo-5c8cb8bdb5-qxdmt): UP
```

Check the initial users created for us:

```bash
pgo show user -n pgo hippo --show-system-accounts
```

```
CLUSTER USERNAME    PASSWORD                 EXPIRES STATUS ERROR
------- ----------- ------------------------ ------- ------ -----
hippo   postgres    4x>CAB(aaXUrdL+2Z{g{6fkX never   ok
hippo   primaryuser |-+xn,1H<]ClhhnZtopAo8l( never   ok
hippo   testuser    xxtzxlsD}Lb_,y_{Jg)Rh3Km never   ok
```

## Connection via psql

Similar to previous labs, we will do a port forward first:

```bash
kubectl -n pgo get svc
```

```bash
kubectl -n pgo port-forward svc/hippo 5432:5432
```

Then connect by the normal PostgreSQL client `psql`:
```bash
psql -h localhost -p 5432 -U testuser hippo
```

```
Password for user testuser:
psql (13.2 (Debian 13.2-1.pgdg100+1))
Type "help" for help.

hippo=>
```


## Installing and using pgAdmin

Simply run the following:

```bash
pgo create pgadmin -n pgo hippo
```

Again, you will need to wait until provisioning is completed.

Then do another port-forward:

```bash
kubectl -n pgo port-forward svc/hippo-pgadmin 5050:5050
```

View port 5050 using Google Cloud Shell's web preview.

## Congratulations!

TODO

### What to do next

For more on operator, see for example the [operator framework](https://operatorframework.io/) for a collection.
