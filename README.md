# Kubernetes the `Kubeadm` Way

This tutorial, inspired by Kelsey Hightower's [`kubernetes-the-hard-way`](https://github.com/kelseyhightower/kubernetes-the-hard-way), walks you through setting up highly available Kubernetes cluster by using `Kubeadm`, therefore `the Kubeadm way`.

`Google Cloud Platform` will be used as it's really handy for me.


## Target Audience

The target audience for this tutorial is someone planning to build highly available OSS Kubernetes clusters by purely using the **official** tooling: `Kubeadm`.


## OS, Tools, Components and Versions

* OS: Ubuntu 18.04 LTS, 64bit
* [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) v1.15.1
* [kubernetes](https://github.com/kubernetes/kubernetes) v1.15.1
* [cri-o](https://github.com/cri-o/cri-o) v1.13.11-dev
* [canal](https://docs.projectcalico.org/v3.8/getting-started/kubernetes/installation/flannel) v3.8


## Labs

This tutorial assumes you have access to the [Google Cloud Platform](https://cloud.google.com). While GCP is used for basic infrastructure requirements, the lessons learned in this tutorial can be applied to other platforms.

* [01-Prerequisites](docs/01-prerequisites.md)
* [02-Provisioning Infrastructure](docs/02-infrastructure.md)
* [03-Preparing Controllers Before Running `kubeadm`](docs/03-prepare-controllers.md)
* [04-Bootstrapping and Joining Controllers](docs/04-init-join-controllers.md)
* [05-Preparing Workers Before Running `kubeadm`](docs/05-prepare-workers.md)
* [06-Joining Workers](docs/06-join-workers.md)
* [07-Configuring `kubectl` for Remote Access](docs/07-configure-kubectl.md)
* [08-Performing Smoke Tests](docs/08-smoke-test.md)
* [09-Cleaning Up](docs/09-cleanup.md)
