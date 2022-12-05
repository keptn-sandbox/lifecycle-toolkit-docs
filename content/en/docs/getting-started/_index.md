---
title: Getting Started
description: Learn how to use the Keptn Lifecycle Toolkit and explore basic features.
icon: concepts
layout: quickstart
weight: 15
hidechildren: true # this flag hides all sub-pages in the sidebar-multicard.html
---

The Keptn Lifecycle Toolkit brings application-awareness to your Kubernetes cluster and helps you reliably delivering your application with:

* Pre-Deployment Tasks: e.g. checking for dependant services, checking if the cluster is ready for the deployment, etc.
* Pre-Deployment Evaluations: e.g. evaluate metrics before your application gets deployed (e.g. layout of the cluster
* Post-Deployment Tasks: e.g. trigger a test, trigger a deployment to another cluster, etc.
* Post-Deployment Evaluations: e.g. evaluate the deployment, evaluate the test results, etc.

All of these things can be executed on a workload or on an application level, whereby an application is a collection of multiple workloads.

## What you will learn here
* Use the Keptn Lifecycle Toolkit to control the deployment of your application
* Connect the lifecycle-toolkit to Prometheus
* Use pre-deployment tasks to check if a dependency is met before deploying a workload
* Use post-deployment tasks on an application level to send a notification

## Prerequisites
* A Kubernetes cluster >= Kubernetes 1.24
    * If you don't have one, we recommend [Kubernetes-in-Docker(KinD)](https://kind.sigs.k8s.io/docs/user/quick-start/) to set up your local development environment
* kubectl installed on your system
    * See (https://kubernetes.io/docs/tasks/tools/) for more information

## Install the Keptn Lifecycle Toolkit
At the moment, the lifecycle controller needs *cert-manager* to be installed. Therefore, you can install cert-manager using:

<!-- 
[cert-manager](https://github.com/cert-manager/cert-manager/releases/download/v1.10.0/cert-manager.yaml)
-->
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.0/cert-manager.yaml
kubectl wait --for=condition=Available deployment/cert-manager-webhook -n cert-manager --timeout=60s
```

After that, you can install the lifecycle toolkit using the current release manifest:
<!---x-release-please-start-version-->
```
kubectl apply -f https://github.com/keptn/lifecycle-toolkit/releases/download/v0.4.0/manifest.yaml
kubectl wait --for=condition=Available deployment/klc-controller-manager -n keptn --timeout=120s
```
<!---x-release-please-end-->

Now, the Lifecycle Toolkit and its dependency is installed and ready to use.

## The Demo Application
For this demonstration, we use a slightly modified version of [the PodTatoHead](https://github.com/podtato-head/podtato-head).

![img.png](assets/podtatohead.png)

Over time, we will evolve this application from a simple manifest to a Keptn-managed application. We will install it first with kubectl and add pre- as well as post-deployment tasks. For this, we will check if the entry service is available before the other ones get scheduled. Afterwards, we will add evaluations to ensure that our infrastructure is in a good shape before we deploy the application. Finally, we will evolve to a GitOps driven deployment and will notify an external webhook service when the deployment has finished.

## I want to ...
* Deploy the Demo Application
    * [Using kubectl](./deploy-kubectl)
    * [Using ArgoCD](./deploy-argocd)
* [Observe my deployment using Prometheus and Grafana](./observability.md)
* [Write a Keptn Task](./writing-a-task.md)

* [Modify the Demo Application](./modify-app.md)
