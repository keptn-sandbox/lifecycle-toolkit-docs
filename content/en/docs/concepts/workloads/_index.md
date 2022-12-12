---
title: Workloads
description: Learn what Keptn Workloads are and how to use them
icon: concepts
layout: quickstart
weight: 10
hidechildren: true # this flag hides all sub-pages in the sidebar-multicard.html
---

## What are workloads for Keptn?
Workloads are representing Kubernetes primitives as [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset) and [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/). Workload resources are used to track the state of your deployments and to trigger pre- and post-deployment tasks and evaluations.

## Definition
A workload is created by annotating the Kubernetes resource with the following annotations/labels:
* `app.kubernetes.io/name`
* `keptn.sh/workload`

If one of these annotations exist, the Keptn Lifecycle Toolkit will create a workload resource for it.

Additionally, the following annotations can be used to add functionality to these workloads:
* `app.kubernetes.io/part-of` or `keptn.sh/app`: The name of the [App]({{< ref "/docs/concepts/apps" >}}) the Workload belongs to.
* `app.kubernetes.io/version` or `keptn.sh/version`: The version of the Workload
* `keptn.sh/pre-deployment-tasks`: A comma-separated list of pre-deployment [tasks]({{< ref "/docs/concepts/tasks" >}}) to be executed before the Workload is deployed
* `keptn.sh/post-deployment-tasks`: A comma-separated list of post-deployment [tasks]({{< ref "/docs/concepts/tasks" >}}) to be executed after the Workload is deployed
* `keptn.sh/pre-deployment-evaluations`: A comma-separated list of pre-deployment [evaluations]({{< ref "/docs/concepts/evaluations" >}}) to be executed before the Workload is deployed
* `keptn.sh/post-deployment-evaluations`: A comma-separated list of post-deployment [evaluations]({{< ref "/docs/concepts/evaluations" >}}) to be executed after the Workload is deployed

## Resources
When a workload is detected, the Keptn Lifecycle Toolkit will create a KeptnWorkload resource fort it. This resource is watched by the Keptn Lifecycle Toolkit and if changes are detected, the Keptn Lifecycle Toolkit will create a new KeptnWorkload Instance resource.

### Keptn Workload Instance

A Workload Instance is responsible for executing the pre- and post-deployment [tasks]({{< ref "/docs/concepts/tasks" >}}) of a workload. In its state, it keeps track of the current status of all tasks, as well as the overall state of
the Pre Deployment phase, which is used by the scheduler to tell that a pod can be allowed to be placed on a node.
Workload Instances have a reference to the respective Deployment/StatefulSet/ReplicaSet, to check if it has reached the desired state. If it detects that the referenced object has reached
its desired state (e.g. all pods of a deployment are up and running), it will be able to tell that the `PostDeploymentTask` can be triggered.
