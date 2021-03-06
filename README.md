This repo has all the steps you need to create a modern GitOps style workflow with Kubernetes.

### Pre-reqs
* A vanilla Kubernetes cluster
* A default `StorageClass` resource installed in the cluster
* `helm` to install the Helm operator
* `bash` to run all the install scripts
* `kubectl` to create Namespaces and Secrets
* `mkcert` for all TLS certs

## Installation
1. Starting with a new cluster with a default StorageClass
1. `helm-operator/install.sh`
1. `kapp deploy -a sealed-secrets -f manifests/sealed-secrets`
1. `./setup-secrets.sh`
1. `kapp deploy -a ingress-nginx -f manifests/ingress-nginx`
1. `kapp deploy -a harbor -f manifests/harbor`
1. Install [TBS](https://github.com/techgnosis/tanzu-build-service)
1. `kapp deploy -a concourse -f manifests/concourse`
1. Build and push Concourse Helper with `cd concourse/Helper && ./concourse/Helper/build.sh 1`
1. `setup-pipeline-secrets.sh`
1. `kapp deploy -a concourse-secrets -f manifests/concourse-main`
1. `cd concourse/pipeline`
1. `./fly.sh`
1. `cd ..`
1. Unpause the pipeline

# Stack Overview

### Ingress
Ingress controllers are easier to manage than NodePorts for every app. Use the [Kubernetes in-tree nginx Ingress controller](https://github.com/techgnosis/ingress). It works fine for a lab environment. This implementation uses `hostNetwork: true` to bind port 443 for convenience.

### Harbor
[Harbor](https://github.com/techgnosis/harbor2) is an OCI image registry with lots of great security features. Harbor uses the nginx Ingress controller for convenience.

### Concourse
[Concourse](https://github.com/techgnosis/concourse) is a container-native automation tool commonly used as a "CI/CD" tool. Concourse uses the nginx Ingress controller for convenience.

### Tanzu Build Service
[Tanzu Build Service](https://github.com/techgnosis/tanzu-build-service) (TBS) uses Cloud Native Buildpacks to turn source code into OCI images. TBS has no UI and does not use the Ingress controller.

Use TBS to build [Spring PetClinic](https://github.com/spring-projects/spring-petclinic)


### K8s manifests
Your app is defined entirely in [Kubernetes manifests](https://github.com/techgnosis/deploy-petclinic). `kapp` is used to deploy those manifests as part of a Concourse pipeline.


### PetClinic
PetClinic is a good example of a Spring Boot app. Use Flux to monitor the [PetClinic K8s manifests](https://github.com/techgnosis/petclinic) and deploy them

### Tips to make life easier
1. I used Ubuntu instead of Alpine for the Concourse Helper image. musl behaves strangely sometimes. I was unable to run a particular Golang binary in Alpine.


### TODO
* Need to deploy MySQL for PetClinic
* Write Wavefront Concourse task
* How to install everything at once
* How do you provide a username and password to `pks get-credentials` for use with Concourse? Otherwise I get a password prompt when using OIDC
* Switch from nginx to Contour using the Bitnami chart
* Switch to Bitnami for Harbor