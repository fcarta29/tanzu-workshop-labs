# LABS-03 - Tanzu Build Service/Kpack

[Main](../../README.md)

# Introduction

Tanzu Build Service is an enterprise offering of cloud native build pack. its a Kubernetes-native approach of building and managing containers. 

TBS leverages two open-source projects as primary components:-

Cloud Native Buildpacks to transform your application source code into an images. Cloud native buildpacks Automatically Configure Frameworks and App Dependencies as layers. With layered artifact, its really easy to patch individual layer no need to rebuild entire image. 

KPACK to extend the Kubernetes API to automatically build and maintain container images.

This project shows how to use [kpack](https://github.com/pivotal/kpack), an open-source project by [Pivotal](https://pivotal.io), to leverage [Cloud Native Buildpacks](https://buildpacks.io) on any Kubernetes clusters.

Using kpack, you can automatically build secure Docker images from your source code, without having to write a `Dockerfile`. Moreover, kpack can rebase your Docker images when updates are available. Let's say you're deploying a Java app as a container image, embedding a JRE: when a new JRE version is out, kpack can update your image without having you building a new container image.


## Deploy TBS/Kpack
>As part of cf-for-k8s setup in previous lab we already installed kpack(build service) on the cluster.


## Explore/ kpack cli 
This repository describes how to use kpack to create your first Docker image.

Check that kpack is running:
```bash
$ kubectl -n kpack get pods
NAME                                READY   STATUS    RESTARTS   AGE
kpack-controller-6d9c8cd8dc-hcqvx   2/2     Running   0          12h
kpack-webhook-6df4f46998-mp8kb      2/2     Running   0          12h
```

We'll build Docker images using a Cloud Foundry buildpack (don't worry: you can deploy the resulting Docker image anywhere 😋).

Lets explore kubectl and kp cli to access builders, clusterstack, and clusterstore resources on the cluster these resources are deployed as part of TAS setup in "cf-workloads-staging" namespace. 


Show Builders, Out of the box builder configuration
```bash
$ kp builder list -n cf-workloads-staging
or 
$ kubectl get builder -n cf-workloads-staging
```

Show buildpack definitions and ordering
```bash
$ kp builder status cf-default-builder -n cf-workloads-staging
or 
$ kubectl describe builder cf-default-builder -n cf-workloads-staging
```

Show Images that are registered with TBS as part of CF push operation performed in previous LAB
```bash
$ kp images list -n cf-workloads-staging
```

Let's trigger build for the image
```bash
$ kp images trigger ee7bd099-f6b6-4660-b6b4-1e7e76953ed3  -n cf-workloads-staging
```

List build for the image
```bash
$ kp build list <image-name> -n cf-workloads-staging
or
$ kubectl get cnbbuilds -n cf-workloads-staging
```

Check the logs for specific image build by specifying build number. This will list all the phases involved in build process PREPARE, DETECT, ANALYZE, RESTORE, BUILD, EXPORT, and COMPLETION
```bash
$ kp build logs <image-name> -b 3 -n cf-workloads-staging
```

## Creating a Docker image from a Git repository using kpack

Now we will build images, All the components requred for defining and building images are already installed 
Secret for push access to your registry, Secret for read access to your Git repository, and 
Service account for your registry and your Git repository credentials.

Finally, create a configuration file for building a Docker image from your Git repository:

```bash
$ cat spring-petclinic-image.yaml

    apiVersion: kpack.io/v1alpha1
    kind: Image
    metadata:
    annotations:
        sidecar.istio.io/inject: "false"
    name: spring-petclinic
    namespace: cf-workloads-staging
    spec:
    tag: harbor.workshop.frankcarta.com/workshop-01/spring-petclinic
    imageTaggingStrategy: BuildNumber
    failedBuildHistoryLimit: 10
    successBuildHistoryLimit: 10
    cacheSize: 2G
    serviceAccount: cc-kpack-registry-service-account
    builder:
        name: cf-default-builder
        kind: Builder
    source:
        git:
        url: https://github.com/samarsinghal/spring-petclinic.git
        revision: main

$ k create -f spring-petclinic-image.yaml
```

Check the build process logs for spring-petclinic application

```bash
$ kp build logs spring-petclinic -b 1 -n cf-workloads-staging
```

Get application image also Monitor kpack build status Status is `Unknown` while image is being built. 
Wait a couple of minutes, and the status will be updated

```bash
$ kubectl get cnbbuilds -n cf-workloads-staging
```

Now deploy this build image on the cluster using deployment file. Update image in the deployment file and push it on the cluster
```bash
$ cat deployment.yaml
```
Update container image in the deployment file and push it on the cluster

```bash
$ kubectl create -f deployment.yaml
```

Check the deployment 
```bash
$ kubectl get deployment -n cf-workloads
```


