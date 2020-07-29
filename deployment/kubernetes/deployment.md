---
description: >-
  In this section, you'll learn how to create a Kubernetes deployment and
  deploying the Hello World container image.
---

# Deployment

This section continues from the previous section - make sure you do the tutorial in sequence.

{% page-ref page="container-image.md" %}

## Pod

A Kubernetes pod is a group of containers, tied together for the purposes of administration and networking. It can contain one or more containers.  All containers within a single pod will share the same networking interface, IP address, volumes, etc.  All containers within the same pod instance will live and die together.  It’s especially useful when you have, for example, a container that runs the application, and another container that periodically polls logs/metrics from the application container.

You can start a single Pod in Kubernetes by creating a Pod resource. However, a Pod created this way would be known as a Naked Pod. If a Naked Pod dies/exits, it will not be restarted by Kubernetes. A better way to start a pod, is by using a higher-level construct such as a Deployment.

## Deployment

Deployment provides declarative way to manage a set of Pods. You only need to describe the desired state in a Deployment resource, and behind the scenes, a Kubernetes Deployment controller will change the actual state to the desired state for you. It does this using a resource called a ReplicaSet under the covers.

![](../../.gitbook/assets/image%20%2822%29.png)

### Deployment YAML

You can create a Deployment and deploy into Kubernetes using `kubectl` command line like in the [Hello World tutorial](../../getting-started/helloworld/kubernetes-engine.md). That's great to get a feel of Kubernetes. However, it's best that you create a YAML file first, and then deploy the YAML file.

```bash
# Under the helloworld-springboot-tomcat directory , create a k8s directory
mkdir k8s/

PROJECT_ID=$(gcloud config get-value project)
kubectl create deployment helloworld \
  --image=gcr.io/${PROJECT_ID}/helloworld \
  --dry-run \
  -o yaml > k8s/deployment.yaml
```

You can open the `k8s/deployment.yaml` file to see the content. Following is a version of the YAML file where it's slimmed down to the bare minimum.

```yaml
# API Version and Kind are important to indicate the type of resource
apiVersion: apps/v1
kind: Deployment
metadata:
  # Every Kubernetes resource has a name that's unique within a namespace
  name: helloworld
  # Every Kubernetes can have labels, label key/value pairs can be queried later.
  labels:
    app: helloworld
spec:
  # The number of the pods that start
  replicas: 1
  
  # The labels that matches the pods within this deployment
  selector:
    matchLabels:
      app: helloworld
  # Every instance of the pod will be created using the template below
  template:
    metadata:
      # Every new pod created by the deployment will have these labels
      # The name of a newly created pod will be generated
      labels:
        app: helloworld
    spec:
      containers:
      # Every container can have a name, and the container image to run
      - name: helloworld
        image: gcr.io/.../helloworld
```

{% hint style="info" %}
The instructors will explain the descriptor in detail. You can read more about Deployment in the [Kubernetes Deployment Guide](http://kubernetes.io/docs/user-guide/deployments/).  The specification can be found in [Kubernetes API reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10).
{% endhint %}

### Deploy

Use `kubectl` command line to deploy the YAML file:

```bash
kubectl apply -f k8s/deployment.yaml
```

To verify the application is deployed, see all the pods that are running:

```bash
kubectl get pods
```

You should see that there is one pod running!

```bash
NAME                         READY   STATUS    RESTARTS   AGE
helloworld-...               1/1     Running   0          ...
```

## Basic Interactions

### Find Pods with Labels

`kubectl get pods` shows you every pod running in the current namespace. You can limit the output to just the application you are interested in by select only pods matching certain label key/value pairs. 

```bash
kubectl get pods -lapp=helloworld
```

### Scaling a Deployment

Scale the number of instances from 1 to 2.

```bash
kubectl scale deployment helloworld --replicas=2
```

Verify that there are now 2 pods running:

```bash
kubectl get pods
```

### Delete a Pod

Out of the 2 pods, pick one to delete, and then observe that Kubernetes automatically starts another pod instance so that there are always 2 pods running.

```bash
POD_NAME=$(kubectl get pods -lapp=helloworld -o jsonpath='{.items[0].metadata.name}')
kubectl delete pod ${POD_NAME}
```

Verify that there still 2 pods running, but one of them has a lower `age` indicating it's recently started.

```bash
kubectl get pods
```

### Delete All Pods

You can use labels to delete all pods matching certain label key/value pairs.

```bash
kubectl delete pod -lapp=helloworld
```

### Stream Logs

You can see the logs from the pod, and follow the log as new logs are produced:

```bash
POD_NAME=$(kubectl get pods -lapp=helloworld -o jsonpath='{.items[0].metadata.name}')
kubectl logs -f ${POD_NAME}
```

### Executing Commands

You can execute commands directly in the container instance. However, the container image will need to contains the command that you'd like to run.  The Hello World application built with Jib uses a Distroless base image by default - and the Distroless base image does not come with any shell commands for security purposes.

Let's deploy an Nginx container that contains the executables and see how you can shell into the container instance.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/controllers/nginx-deployment.yaml
```

See that Nginx container is running:

```bash
kubectl get pods -lapp=nginx
```

Use a specific Nginx pod, and shell into the container instance:

```bash
POD_NAME=$(kubectl get pods -lapp=nginx -o jsonpath='{.items[0].metadata.name}')
kubectl exec -ti ${POD_NAME} /bin/bash
```

{% hint style="info" %}
The `-ti` flag means to receive output from TTY, and also that the session is interactive \(i.e., you'll be typing commands\).
{% endhint %}

Once you are in the container instance's shell, you can explore the container instance:

```bash
# Within the container instance shell:
ls
ls /sbin
ls /bin
exit
```

In this Nginx container image, you can see that there are actually many command line utilities that's not needed for production deployment of an Nginx server. Exposing more commands like this may increase attack surface area if the container instance is compromised.  For this reason, Distroless base images do not include any commands. On the other hand, lack of commands may increase the difficulty to debug the application instance.

{% hint style="warning" %}
Delete the Nginx deployment before you continue!

`kubectl delete deployment nginx-deployment`
{% endhint %}





