# Hashicorp Vault on Kubernetes (development version)

## Overview


## Introduction

Hashicorp Vault is a secret management system. In cloud native application, one often need to connect to internal services, such as Database, by providing a credential. This is secret/sensitive material and need to be protected.

### Reference

https://learn.hashicorp.com/tutorials/vault/kubernetes-raft-deployment-guide?in=vault/kubernetes

The guide we follow for install

https://www.vaultproject.io/docs/internals/architecture

High level system design overview

https://www.vaultproject.io/docs/internals/security

The security model

https://www.vaultproject.io/docs/concepts/seal

https://www.vaultproject.io/docs/secrets


## Installing a development version

We will use the helm chart method. As this is for development, we will skip the elaborate HA setup. (Another reason is because we only have one node in reality) The default settings using `standalone` mode is good enough for us - more realistic than the `dev` mode, but still simplified enough for learning.

The initial steps is just the standard helm chart install flow, and can be followed verbatim:

```bash
kubectl create namespace vault
```

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
```

```bash
helm search repo hashicorp/vault
```

The actual install command we will use here is this:

```bash
helm install vault hashicorp/vault --namespace vault
```

The official documentation is very long and may be confusing, so here are some clarifications:

Each server mode provide a different set of preset defaults for the underlying config. The two main configuration relevant to us is the seal method, and the storage backend used. The storage backend default progresses from in memory for dev mode, to file store in standalone mode, to integrated storage using the Raft consensus protocol in HA (High availability) mode. The default for the seal method is Shamir's secret sharing algorithm.

The Vault server itself is just a program and its behavior is configured by supplying a [configuration file](https://www.vaultproject.io/docs/configuration) in HCL or json format. On the other hand, a Helm deployment consists of the Vault server *together with surrounding setups*. Thus there are two layers of config:

1. The [Helm chart config parameters](https://www.vaultproject.io/docs/platform/k8s/helm/configuration) (specified using the standard format/mechanism for a generic Helm Chart config)
2. The vault server config, which resides in the `server` section of the Helm config above (specifically, it is the field `server.<mode>.config`, where `<mode>` is one of `dev`, `standalone`, or `ha`. The field's value should be the full server config file. Helm provide a [special mechanism](https://helm.sh/docs/chart_template_guide/yaml_techniques/) to allow this kind of value by using a trick in yaml to allow multiline string.

As an example, the official guide has the following (Recalling that Helm config can either be specified on command line argument as `--set "<yaml path>=<value>"`, or saved into a yaml file, then supplied via `-f override-values.yaml`):

```
server:
  ...(snipped)...
  ha:
      enabled: true
      replicas: 3

      config: |
        ui = true

        listener "tcp" {
          tls_disable = 1
...(snipped)
```

Notice how the mode has to be enabled first.

(Hint: even the server config can be specified inline using `--set server.standalone.config='{ listener "tcp" { address = "0.0.0.0:8200" }'`, for example)

Back to our task.

The install command should show you this output:

```
NAME: vault
LAST DEPLOYED: Thu Mar  4 13:46:07 2021
NAMESPACE: vault
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault
```

The command shown at the end only works if you have set the namespace to use in the context. We prefer to be more explicit, so please use this instead:

```bash
helm status vault --namespace=vault
```

This will print the above again.

The second command will print the full kubernetes yaml that are actually used. It is very long, so you should pipe it into a file instead.

Notice that the pod will not become ready even after waiting, and **this is normal**. We will explain why, and show what needs to be done, in the next step.

## Manually initialize Vault

The official document has not been upfront about the need to manually initialize Vault (it is mentioned in the guide, but in a section buried towards the later part). See for example this [issue](https://github.com/hashicorp/vault-helm/issues/17). Without this step, the pod will not become ready even after waiting. If you check the events in the kubernetes dashboard, you may find this:

```
Readiness probe failed: Key Value --- ----- Seal Type shamir Initialized false Sealed true Total Shares 0 Threshold 0 Unseal Progress 0/0 Unseal Nonce n/a Version 1.6.2 Storage Type file HA Enabled false
```

We will need to perform the `initialize` operation to generate the first set of shared secrets for the master key, then perform `unseal` for the first time ourself.

For convinience, we will use `kubectl`'s function to drop into a shell inside the pod/container, because the `vault` binary is already installed inside and available for our use.

By default, the pod is named `vault-0`, so type this:

```bash
kubectl exec -ti --namespace=vault vault-0 -- vault operator init
```

The output is:

```
Unseal Key 1: owYBO2Ze1cvDqxbpBalUiU1yPXj4Or4w4GH5wyReGXay
Unseal Key 2: iAuJTL/x7zzjbBLzSauSZhie3Y/3NKJdIpOaIsiRYQvS
Unseal Key 3: 67bAvAhyuB6zVQjupgUgFYJLFbR6SW+0tlTa+pB7wbjN
Unseal Key 4: 5FFhlNbs7DQ5o1uJtoSEH+W+nS9oWigS4QQ5TJD1lTBf
Unseal Key 5: ZKdPIR2Zgt7rMsV312K/vTBnVf4hnEIManmS/CaYbhZD

Initial Root Token: s.WpN0Bi0WPBHJt0LvzSctUd1S

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

The root token is similar to the root password and is used at the user level for the initial login and setup after unseal.

At this point there will be a new event:

```
Readiness probe failed: Key Value --- ----- Seal Type shamir Initialized true Sealed true Total Shares 5 Threshold 3 Unseal Progress 0/3 Unseal Nonce n/a Version 1.6.2 Storage Type file HA Enabled false
```
(The difference is that the total is changed from 0 to 5 etc)

Now perform the unseal:

```bash
kubectl exec -ti --namespace=vault vault-0 -- vault operator 
```

Enter a different unseal key each time when prompted. Repeat this for three times. Below are the printout for the first and third time:

```
Unseal Key (will be hidden):
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    1/3
Unseal Nonce       1fe1685b-7b92-770b-747c-7d5514a96062
Version            1.6.2
Storage Type       file
HA Enabled         false
```

```
Unseal Key (will be hidden):
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.6.2
Storage Type    file
Cluster Name    vault-cluster-ebbd66ca
Cluster ID      10f98705-2cce-2b6c-3f78-a2638b96d1b3
HA Enabled      false
```

At this point it should finally become ready:

```bash
kubectl get pods --namespace=vault
```

```
NAME                                    READY   STATUS    RESTARTS   AGE
vault-0                                 1/1     Running   0          15m
vault-agent-injector-784ccfc788-q44q9   1/1     Running   0          15m
```

For later use, we should also examine the log (in fact the log can be examined during install as an additional diagnostic tool):

```bash
kubectl logs --namespace=vault vault-0
```

```
==> Vault server configuration:

             Api Address: http://172.17.0.4:8200
                     Cgo: disabled
         Cluster Address: https://vault-0.vault-internal:8201
              Go Version: go1.15.7
              Listener 1: tcp (addr: "[::]:8200", cluster address: "[::]:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
               Log Level: info
                   Mlock: supported: true, enabled: false
           Recovery Mode: false
                 Storage: file
                 Version: Vault v1.6.2
             Version Sha: be65a227ef2e80f8588b3b13584b5c0d9238c1d7

==> Vault server started! Log data will stream in below:

2021-03-04T13:46:29.711Z [INFO]  proxy environment: http_proxy= https_proxy= no_proxy=
2021-03-04T13:46:37.571Z [INFO]  core: security barrier not initialized
2021-03-04T13:46:37.571Z [INFO]  core: seal configuration missing, not initialized
...(snipped)...
2021-03-04T13:55:52.570Z [INFO]  core: seal configuration missing, not initialized
2021-03-04T13:55:54.225Z [INFO]  core: security barrier not initialized
2021-03-04T13:55:54.226Z [INFO]  core: security barrier initialized: stored=1 shares=5 threshold=3
2021-03-04T13:55:54.228Z [INFO]  core: post-unseal setup starting
2021-03-04T13:55:54.243Z [INFO]  core: loaded wrapping token key
2021-03-04T13:55:54.245Z [INFO]  core: successfully setup plugin catalog: plugin-directory=
2021-03-04T13:55:54.245Z [INFO]  core: no mounts; adding default mount table
2021-03-04T13:55:54.250Z [INFO]  core: successfully mounted backend: type=cubbyhole path=cubbyhole/
2021-03-04T13:55:54.252Z [INFO]  core: successfully mounted backend: type=system path=sys/
2021-03-04T13:55:54.256Z [INFO]  core: successfully mounted backend: type=identity path=identity/
2021-03-04T13:55:54.266Z [INFO]  core: successfully enabled credential backend: type=token path=token/
2021-03-04T13:55:54.268Z [INFO]  rollback: starting rollback manager
2021-03-04T13:55:54.270Z [INFO]  core: restoring leases
2021-03-04T13:55:54.273Z [INFO]  expiration: lease restore complete
2021-03-04T13:55:54.273Z [INFO]  identity: entities restored
2021-03-04T13:55:54.273Z [INFO]  identity: groups restored
2021-03-04T13:55:54.274Z [INFO]  core: usage gauge collection is disabled
2021-03-04T13:55:54.274Z [INFO]  core: post-unseal setup complete
2021-03-04T13:55:54.275Z [INFO]  core: root token generated
2021-03-04T13:55:54.275Z [INFO]  core: pre-seal teardown starting
2021-03-04T13:55:54.275Z [INFO]  rollback: stopping rollback manager
2021-03-04T13:55:54.275Z [INFO]  core: pre-seal teardown complete
2021-03-04T13:58:12.939Z [INFO]  core.cluster-listener.tcp: starting listener: listener_address=[::]:8201
2021-03-04T13:58:12.939Z [INFO]  core.cluster-listener: serving cluster requests: cluster_listen_address=[::]:8201
2021-03-04T13:58:12.939Z [INFO]  core: post-unseal setup starting
2021-03-04T13:58:12.940Z [INFO]  core: loaded wrapping token key
2021-03-04T13:58:12.941Z [INFO]  core: successfully setup plugin catalog: plugin-directory=
2021-03-04T13:58:12.942Z [INFO]  core: successfully mounted backend: type=system path=sys/
2021-03-04T13:58:12.942Z [INFO]  core: successfully mounted backend: type=identity path=identity/
2021-03-04T13:58:12.942Z [INFO]  core: successfully mounted backend: type=cubbyhole path=cubbyhole/
2021-03-04T13:58:12.945Z [INFO]  core: successfully enabled credential backend: type=token path=token/
2021-03-04T13:58:12.945Z [INFO]  core: restoring leases
2021-03-04T13:58:12.947Z [INFO]  rollback: starting rollback manager
2021-03-04T13:58:12.947Z [INFO]  expiration: lease restore complete
2021-03-04T13:58:12.947Z [INFO]  identity: entities restored
2021-03-04T13:58:12.947Z [INFO]  identity: groups restored
2021-03-04T13:58:12.948Z [INFO]  core: usage gauge collection is disabled
2021-03-04T13:58:12.949Z [INFO]  core: post-unseal setup complete
2021-03-04T13:58:12.949Z [INFO]  core: vault is unsealed
```

## Install client and accessing the Vault server

At this point things become easier. Follow the [tutorial](https://learn.hashicorp.com/tutorials/vault/getting-started-install?in=vault/getting-started) with the standard linux procedure (add trust to a new repo, then download and install packaged software using OS provided tool):

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
```

```bash
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
```

```bash
sudo apt-get update && sudo apt-get install vault
```

Verify that the client tool `vault` is available. We need to do two more things:

1. Expose Vault to outside network on "local machine":

As Vault's purpose is to manage secret, it is usually for internal use only and should not be exposed. We do this for development purpose:

```bash
kubectl port-forward --namespace=vault vault-0 8200:8200
```

2. Set enviornment variable to point to Vault endpoint

From the log output in the last step, we see that the URL is `http://172.17.0.4:8200`. However, as we have done port forwarding, it is now accessible over `127.0.0.1:8200`. So,

```bash
export VAULT_ADDR="http://127.0.0.1:8200"
```

Notice the `http` instead of `https` as the default Helm settings did not include `https` due to complications with cert setup etc.

We can now use the command line client for some sanity check:

```bash
vault status
```

```
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.6.2
Storage Type    file
Cluster Name    vault-cluster-ebbd66ca
Cluster ID      10f98705-2cce-2b6c-3f78-a2638b96d1b3
HA Enabled      false
```

```bash
vault login
```

```
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.WpN0Bi0WPBHJt0LvzSctUd1S
token_accessor       PczToMdSTHJJEcPpHwu9tzwT
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

Alternatively, the default installation expose an UI dashboard at the same URL. Use Web Preview to proxy port 8200, then open it using a browser. Login in the same way as the command line client, and you will be led to a dashboard with an invitation to do a quick start tutorial in it.

