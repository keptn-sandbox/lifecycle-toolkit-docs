---
title: Deploy the Demo Application using ArgoCD
description: Deploy the Demo Application using ArgoCD
icon: concepts
layout: quickstart
weight: 20
hidechildren: true # this flag hides all sub-pages in the sidebar-multicard.html
---
The Keptn Lifecycle Toolkit brings application-awareness to your Kubernetes cluster and helps you reliably delivering your application with:
## Objectives
* Install the Demo Application using Kubectl
* Watch the progress of the deployment
* Add applicaton awareness to the deployment
* Add post-deployment tasks to the deployments

## Check out the Getting Started Repository
```
git clone https://github.com/keptn/lifecycle-toolkit-getting-started.git
```

## Install Demo Application
For this demonstration, we use a slightly modified version of [the PodTatoHead](https://github.com/podtato-head/podtato-head).

To install it, simply apply the manifest:
```shell
kubectl apply -f demo-application/base/manifest.yml
```

You can watch the progress of the deployment as follows:
<details>
<summary>Watch workload state</summary>
When the Lifecycle Toolkit detects workload labels ("app.kubernetes.io/name" and "keptn.sh/workload") on a resource, a KeptnWorkloadInstance (kwi) resource will be created. Using this resource you can watch the progress of the deployment.

```shell
kubectl get keptnworkloadinstances -n podtato-kubectl
```

This will show the current status of the Workloads and in which phase they are at the moment. You can get more detailed information about the workloads by describing one of the resources:

```shell
kubectl describe keptnworkloadinstances entry -n podtato-kubectl
```

Note that there are more detailed information in the event stream of the object.
</details>

<details>
<summary>Watch application state</summary>
Although you didn't specify an application in your manifest, the Lifecycle Toolkit assumed that this is a single-service application and created an ApplicationVersion (kav) resource for you.

Using `kubectl get keptnappversions -n podtato-kubectl` you can see state of these resources.
</details>

<details>
<summary>Watch pods</summary>
Obviously, you should see that the pods are starting normally. You can watch the state of the pods using:

```shell
kubectl get pods -n podtato-kubectl
```
</details>

Finally, all of these pods should be running and the workload as well as the application deployment should be in a succeeded state.

In this state the lifecycle toolkit created Kubernetes Events for your deployment process. Furthermore, it exports metrics with the deployment stats and also would export traces if an open-telemetry collector was available. To jump directly into this topic, please check out the [observability](./install-observability.md) section. 

## Add Application Awareness
In the previous step, we installed the demo application without any application awareness. This means that the Lifecycle Toolkit assumed that every workload is a single-service application at the moment and created the Application resources for you.

To get the overall state of an application, we need a grouping of workloads, called KeptnApp in the Lifecycle Toolkit. To get this working, we need to modify our application manifest with two things:
* Add an "app.kubernetes.io/part-of" or "keptn.sh/app" label to the deployment
* Create an application resource

### Preparing the Manifest and create an App resource

---
**TL;DR**

You can also used the prepared manifest and apply it directly using: `kubectl apply -k demo-application/with-app-awareness/` and proceed [here](#watch-application-behavior).

---
**Otherwise**

Create a temporary directory and copy the base manifest there:
```shell
mkdir ./my-deployment
cp demo-application/base/manifest.yml ./my-deployment
```

Now, open the manifest in your favorite editor and add the following label to the deployments, e.g.:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: podtato-head-right-leg
  namespace: podtato-kubectl
  labels:
    app: podtato-head
spec:
  selector:
    matchLabels:
      component: podtato-head-right-leg
  template:
    metadata:
      labels:
        component: podtato-head-right-leg
      annotations:
        keptn.sh/workload: "right-leg"
        keptn.sh/version: "0.1.0"
        keptn.sh/app: "podtato-head"
    spec:
      terminationGracePeriodSeconds: 5
      containers:
        - name: server
          image:  ghcr.io/podtato-head/right-leg:0.2.7
          imagePullPolicy: Always
          ports:
            - containerPort: 9000
          env:
            - name: PODTATO_PORT
              value: "9000"
```

Now, update the version of the workloads in the manifest to `0.2.0`.

Finally, create an application resource (app.yaml) and save it in the directory as well:
```yaml
apiVersion: lifecycle.keptn.sh/v1alpha1
kind: KeptnApp
metadata:
  name: podtato-head
  namespace: podtato-kubectl
spec:
  version: "0.1.0"
  workloads:
    - name: left-arm
      version: "0.1.1"
    - name: left-leg
      version: "0.1.1"
    - name: entry
      version: "0.1.1"
    - name: right-arm
      version: "0.1.1"
    - name: left-arm
      version: "0.1.1"
    - name: hat
      version: "0.1.1"
```

Now, apply the manifests:
```shell
kubectl apply -f ./my-deployment/.
```

### Watch Application behavior
Now, your application gets deployed in an application aware way. This means that pre-deployment tasks and evaluations would be executed if you would have any. The same would happen for post-deployment tasks and evaluations after the last workload has been deployed successfully.

<details>
<summary>Watch application state</summary>
Now that you defined your application, you could watch the state of the whole application using:

```shell
kubectl get keptnappversions -n podtato-kubectl`
```
</details>

You should see that the application is in a progressing state as long as the workloads (`kubectl get kwi`) are progressing. After the last application has been deployed, and post-deployment tasks and evaluations are finished (there are none at this point), the state should switch to completed.

Now, we have deployed an application and are able to get the total state of the application state. Metrics and traces get exported and now we're ready to dive deeper in the world of Pre- and Post-Deployment Tasks.

## Add Pre-Deployment Task

---

**TL;DR**

You can also used the prepared manifest and apply it directly using: `kubectl apply -k demo-application/with-pre-deployment/` and proceed [here](#watch-workload-behavior-with-pre-deployment-task).

---
In this step, we will use a pre-defined Keptn Function to check if a service is available before we deploy it. 

Let's assume that the other services should not start before the entry service is available.

We could achieve this in two different ways:

<details>
<summary>Use a hosted Keptn Function</summary>
In this case, published this function in our repository and you can simply reference to it in your KeptnTaskDefinition Manifest as this:

```yaml
apiVersion: lifecycle.keptn.sh/v1alpha1
kind: KeptnTaskDefinition
metadata:
  name: pre-deployment-check-entry
  namespace: podtato-kubectl
spec:
  function:
    httpRef:
      url: https://raw.githubusercontent.com/keptn/lifecycle-toolbox/main/functions-runtime/samples/ts/http.ts
    parameters:
      map:
        url: http://podtato-head-entry.podtato-kubectl.svc.cluster.local:9000
```

Note, that we referred to the URL function in `.spec.function.httpRef.url`

</details>

<details>
<summary>Create the Task Inline</summary>
Alternatively, you could also create the function directly in the KeptnTaskDefinition manifest. This would look like this:

```yaml
apiVersion: lifecycle.keptn.sh/v1alpha1
kind: KeptnTaskDefinition
metadata:
  name: pre-deployment-check-entry
  namespace: podtato-kubectl
spec:
  function:
    inline:
      code: |
        let text = Deno.env.get("DATA");
        let data;
        data = JSON.parse(text);
  
        try {
          let resp = await fetch(data.url);
        }
        catch (error){
          console.error("Could not fetch url");
          Deno.exit(1);
        }
    parameters:
      map:
        url: http://podtato-head-entry.podtato-kubectl.svc.cluster.local:9000
```

In this case, we added the typescript code directly in the manifest. This is valuable if you want to deploy the function code nearby your application and don't need to share it or don't want to rely on an external service.
</details>

### Change manifests and apply them
Now that we specified our pre-deployment task, we can add this to our workloads. Therefore, you can add the `keptn.sh/pre-deployment-tasks: pre-deployment-check-entry` annotation to all workloads except the entry service in your manifest.yaml.

Before we apply the new manifests, we need to drop the old entry-service to see the effect of the pre-deployment tasks by executing `kubectl delete deploy -n keptn podtato-head-entry`.

Whatever path you have chosen before, create this manifest (`workload-pre-deploy.yaml`) nearby your application manifest, raise the version of your application in `app.yaml`, and the workload versions in the `manifest.yaml` and apply it.

### Watch workload behavior with pre-deployment task
Now you should see that the entry service is deployed first and the other services are not deployed until the entry service is available.

Using `kubectl get pods` you see that the pre-deployment tasks fail as long as the entry workload is not available. Once the entry workload is available, the pre-deployment tasks succeed and the other workloads will be scheduled.

As before, you can also watch the behavior of the application using `kubectl get keptnappversions -n podtato-kubectl` and the workload state using `kubectl get kwi -n podtato-kubectl`.

## Summary
You learned how to enable the Keptn Lifecycle Toolkit to manage your application and how to use pre-deployment tasks to ensure that your application is in a valid state before it gets scheduled.

