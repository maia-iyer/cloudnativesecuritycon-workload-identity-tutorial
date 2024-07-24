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

## Obtain the code

Navigate to the directory you wish to work in and run the following: 

```shell
export TUTORIAL_ROOT=$(pwd)
git clone -b add_kind https://github.com/maia-iyer/cloudnativesecuritycon-workload-identity-tutorial
```

## Create a Cluster

With the above installed, start a Podman machine with the following commands:

```shell
podman machine init --rootful=true --memory 4096
podman machine start
```

Then, we can create a Kind cluster:

```shell
KIND_EXPERIMENTAL_PROVIDER=podman kind create cluster --config=$TUTORIAL_ROOT/cloudnativesecuritycon-workload-identity-tutorial/KindTutorial/resources/cluster/cluster-config.yaml
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

### Ingress

On Kind we can deploy an Ingress controller to access application services running within the environment. 

```shell
export APP_DOMAIN=$(ipconfig getifaddr en0).nip.io
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout wildcard-tls.key -out wildcard-tls.crt -subj "/CN=*.$APP_DOMAIN/O=Red Hat" -addext "subjectAltName=DNS:*.$APP_DOMAIN"
kubectl create secret tls wildcard-tls-secret --key wildcard-tls.key --cert wildcard-tls.crt
kubectl apply -f $TUTORIAL_ROOT/cloudnativesecuritycon-workload-identity-tutorial/KindTutorial/resources/cluster/ingress-deployment.yaml
```

With an understanding of the infrastructure that will be used for the tutorial along with setting up the initial configurations to work within this environment, we can get started. Proceed to the next tutorial where the we meet Bob and Kaya and learn more about Bob's application, its architecture, and how it can be deployed within the Kubernetes environment.

[Previous Tutorial - Overview](tutorial0.md)

[Next Tutorial - Application Deployment](tutorial2.md)

[Home](../README.md)
