# Tutorial 1 - Environment Setup

For the purpose of this tutorial, we will set up a local Kind cluster to work through the exercises. 

## Prerequisites

Please install the following utilities:
- git
- curl
- helm
- kubectl
- kind
- podman
- sed
- openssl
- vault
- envsubst

## Create a Cluster

With the above installed, start a Podman machine with the following commands:

```shell
podman machine init
podman machine start
```

Then set the `DOCKER_HOST` environment variable:

```shell
export DOCKER_HOST=unix://$(podman info --format '{{.Host.RemoteSocket.Path}}')
```
TODO is the above necessary?

Then, we can create a Kind cluster:

```shell
KIND_EXPERIMENTAL_PROVIDER=podman kind create cluster
```

### Interacting With the Environment

The above commands should have created a single node Kind cluster:

```shell
kubectl get nodes
```

```shell
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   84s   v1.27.3
```

Navigate to a work directory and run the following command:

```shell
export TUTORIAL_ROOT=$(pwd)
```

### Ingress

On cloud Kubernetes clusters, there would normally be an Ingress Controller that can be used to access externally-facing applications. As this is a local tutorial, the Kind cluster does not have such an ingress controller. Therefore, we will rely on port-forwarding to localhost. 

```shell
export APP_DOMAIN=example.com
```

With an understanding of the infrastructure that will be used for the tutorial along with setting up the initial configurations to work within this environment, we can get started. Proceed to the next tutorial where the we meet Bob and Kaya and learn more about Bob's application, its architecture, and how it can be deployed within the Kubernetes environment.

[Previous Tutorial - Overview](tutorial0.md)

[Next Tutorial - Application Deployment](tutorial2.md)

[Home](../README.md)
