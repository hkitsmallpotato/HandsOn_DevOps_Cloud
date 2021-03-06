# HandsOn DevOps Cloud

_Getting your hands dirty to learn DevOps and Cloud_

## Motivation

The main online-only option to learn DevOps and Cloud, that are easily accessible are Play-With-Kubernetes and KataCoda. While they do what they do well, their latency and ephemeral nature is a potential drawback.

Google Cloud Shell also offer access to an ephemeral shell (in the form of a docker container inside a per user VM) for free to all accounts (without the need for credit card/signing up for GCP trial) (Note their TOS and 50 hours/week quota though). Their intended use (in my interpretation) is for doing software development, and for admin operations on GCP using their client command line tool `gcloud`.

There have been an integrated tutorial function that is official in this Cloud Shell product (which they use to promote GCP by easing adoption). Taken together, this present an interesting, although perhaps not as they intended, possibility.

Previously, all the factors above taken together do put some limit on Google Cloud Shell's usefulness as a general purpose development and learning platform. However, their recent updates (around Sep-Dec 2020) changed the equation:
- Instead of the g1-small (boostable to n1-standard-1 for 24 hour) flavor, they now give everyone 8GB ram *as the baseline*.
- `sudo` capability is introduced, and both `docker` (already provided in the old version) and `minikube` are provided out of the box.

These significantly extended the scope of what we may achieve with this shell. This repo is an experiment to implement a set of Hands on Lab using the Shell.

## Usage

Click on the "Open in Google Cloud Shell" button.

(Hint: To further assuage any anxiety, one can choose not to trust the repo when opening tutorials in Google Cloud Shell. This will results in Google allocating a completely ephemeral VM to you, where even the home directory is mostly empty (and will be cleared after session ends). Of course your original home directory is actually still stored safely, tucked away from the current VM.)

(Hint: The last hint can be taken even further: as Google git clone the repo, you may do consecutive tutorials within the same completely ephemeral enviornment, simply by changing directory, then entering this command into the console: `teachme tutorial.md`)

## Contents

Note that for Status:

- WIP means that it is still work in prgress
- Beta means it should be usable, but may lack polish
- Ready means it is ready for use.

| Lab                                                 | Status  | Link                       |
|-----------------------------------------------------|---------|----------------------------|
| Introduction to Kubernetes through Minikube         | Beta    | [![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.svg)](https://ssh.cloud.google.com/cloudshell/editor?cloudshell_git_repo=https%3A%2F%2Fgithub.com%2Fhkitsmallpotato%2FHandsOn_DevOps_Cloud.git&cloudshell_tutorial=intro_kubernetes_minikube%2Ftutorial.md&shellonly=true) |
| Install PostgreSQL via CrunchyData Operator         | Beta    | [![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.svg)](https://ssh.cloud.google.com/cloudshell/editor?cloudshell_git_repo=https%3A%2F%2Fgithub.com%2Fhkitsmallpotato%2FHandsOn_DevOps_Cloud.git&cloudshell_tutorial=install_postgresql_crunchydata_operator%2Ftutorial.md&shellonly=true) |
| Hashicorp Vault on Kubernetes (development version) | Beta    | [![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.svg)](https://ssh.cloud.google.com/cloudshell/editor?cloudshell_git_repo=https%3A%2F%2Fgithub.com%2Fhkitsmallpotato%2FHandsOn_DevOps_Cloud.git&cloudshell_tutorial=hashicorp_vault_kubernetes_dev%2Ftutorial.md&shellonly=true) |
| Using Traefik as Ingress                            | Beta    | [![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.svg)](https://ssh.cloud.google.com/cloudshell/editor?cloudshell_git_repo=https%3A%2F%2Fgithub.com%2Fhkitsmallpotato%2FHandsOn_DevOps_Cloud.git&cloudshell_tutorial=traefik_ingress%2Ftutorial.md&shellonly=true) |
