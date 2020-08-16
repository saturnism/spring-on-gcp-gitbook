---
description: Create a Kubernetes cluster and deploy a container.
---

# Kubernetes Engine

[Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs) is a secured and managed Kubernetes service so you can deploy containerized application in an enterprise/production-grade Kubernetes cluster with a click of a button. 

## Getting Started

### Clone

```text
cd $HOME
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

### Build

```text
./mvnw package
```

### Containerize

#### Enable API

Enable Container Registry API to be able to push container images to the Container Registry.

```bash
gcloud services enable containerregistry.googleapis.com
```

#### Jib

Use Jib to containerize the application:

```bash
PROJECT_ID=$(gcloud config get-value project)

./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
Learn different ways to containerize a Java application in the [Container Image](../../deployment/docker/container-image.md) section.
{% endhint %}

### Create Cluster

#### Enable API

```text
gcloud services enable compute.googleapis.com
gcloud services enable container.googleapis.com
```

#### Create Cluster

Create a [VPC-native Kubernetes Engine cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/alias-ips).

```text
gcloud container clusters create helloworld-cluster \
  --num-nodes 2 \
  --enable-ip-alias \
  --scopes=cloud-platform \
  --network=default \
  --machine-type n1-standard-1
```

{% hint style="info" %}
See [Compute Engine Machine Types documentation](https://cloud.google.com/compute/docs/machine-types) for a list of Machine Types and the associated CPU/Memory resources.
{% endhint %}

#### Cluster Credentials

Kubernetes credential is automatically retrieved and stored in your `$HOME/.kube/config` file. If you need to re-retrieve the credential:

```bash
gcloud container clusters get-credentials helloworld-cluster
```

### Deploy

```bash
PROJECT_ID=$(gcloud config get-value project)

kubectl create deployment helloworld \
  --image=gcr.io/${PROJECT_ID}/helloworld
```

Check that the container is deployed:

```bash
kubectl get pods
```

### Expose

You can expose this one service using a single [Network \(L4\) Load Balancer](https://cloud.google.com/load-balancing/docs/network):

```text
kubectl create service loadbalancer helloworld --tcp=8080:8080
```

{% hint style="info" %}
A Network \(L4\) Load Balancer is the easiest way to expose a single service for a demo. For production environment, you likely will need to [use a HTTP Load Balancer](https://cloud.google.com/kubernetes-engine/docs/how-to/container-native-load-balancing) instead.
{% endhint %}

### Connect

Find the Load Balancer's External IP address:

```text
kubectl get services helloworld
```

Initially, it may display that the External IP is `<pending>`.

```bash
NAME         TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
helloworld   LoadBalancer   ...          <pending>     8080:32414/TCP   ...
```

Re-check until the External IP is assigned.

Then connect with `curl`:

```bash
EXTERNAL_IP=$(kubectl get svc helloworld \
  -ojsonpath='{.status.loadBalancer.ingress[0].ip}')

curl http://${EXTERNAL_IP}:8080
```

## Learn More

* [Kubernetes from Basic to Advanced code lab](https://bit.ly/k8s-lab)
* [Spring Boot on GCP code lab](https://bit.ly/spring-gcp-lab)
* [Spring to Kubernetes Faster and Easier](https://saturnism.me/talk/kubernetes-spring-java-best-practices/)

