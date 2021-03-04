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


## Installing a development version

We will use the helm chart method. As this is for development, we will skip all the elaborate HA setup.

On the other hand, we want to be somewhat realistic in terms of security, so we will override the chart default of standalone mode with file storage backend.

