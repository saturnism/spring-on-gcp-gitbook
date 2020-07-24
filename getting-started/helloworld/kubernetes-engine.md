# Kubernetes Engine

[Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs) is a secured and managed Kubernetes service so you can deploy containerized application in an enterprise/production-grade Kubernetes cluster with a click of a button. 

## Getting Started

### Enable API

```text
# To use Compute Engine
gcloud services enable compute.googleapis.com

# To use Kubernetes Engine
gcloud services enable container.googleapis.com

# To store Container Images in Container Registry
gcloud services enable containerregistry.googleapis.com
```

### Create Cluster

Create a [VPC-native Kubernetes Engine cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/alias-ips).

```text
gcloud container clusters create helloworld-cluster \
  --num-nodes 2 \
  --enable-ip-alias \
  --scopes=cloud-platform \
  --network=default \
  --machine-type n1-standard-1
```

Kubernetes credential is automatically retrieved and stored in your `$HOME/.kube/config` file. If you need to re-retrieve the credential:

```bash
gcloud container clusters get-credentials helloworld-cluster
```

### Clone

```text
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

### Build

```text
./mvnw package
```

### Containerize

{% tabs %}
{% tab title="Jib" %}
[Jib](https://github.com/GoogleContainerTools/jib) can containerize any Java application easily, without a `Dockerfile` nor `docker` installed. Jib will push the container image directly to the remote registry.

```bash
PROJECT_ID=$(gcloud config get-value project)
./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
You can configure [Jib Maven plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin) or [Jib Gradle plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-gradle-plugin) directly in the build file to run the Jib easier, such as `./mvnw jib:build`.
{% endhint %}
{% endtab %}

{% tab title="Buildpack" %}
[Cloud Native Buildpacks](https://buildpacks.io) can containerize applications written in different language without a `Dockerfile`. It does require `docker` installed.

1. Install Docker locally - see [Get Docker documentation](https://docs.docker.com/get-docker/).
2. Install `pack` CLI - see [Installing `pack` documentation](https://buildpacks.io/docs/install-pack/)
3. Build container with `pack`, and use `--publish` flag to push directly to the remote registry:

```bash
PROJECT_ID=$(gcloud config get-value project)
pack build --builder gcr.io/buildpacks/builder:v1 \
  --publish \
  gcr.io/${PROJECT_ID}/helloworld
```
{% endtab %}

{% tab title="Spring Boot 2.3" %}
Since Spring Boot 2.3+, you can build container using the Spring Boot plugin.

```bash
PROJECT_ID=$(gcloud config get-value project)

# Maven with Paketo Buildpack
./mvnw spring-boot:build-image \
  -Dspring-boot.build-image.imageName=gcr.io/${PROJECT_ID}/helloworld
  
# Maven with GCP Buildpack
./mvnw spring-boot:build-image \
  -Dspring-boot.build-image.imageName=gcr.io/${PROJECT_ID}/helloworld \
  -Dspring-boot.build-image.builder=gcr.io/buildpacks/builder
  
# Gradle with Paketo Buildpack
./gradlew bootBuildImage --imageName=gcr.io/${PROJECT_ID}/helloworld

# Gradle with GCP Buildpack
./gradlew bootBuildImage --imageName=gcr.io/${PROJECT_ID}/helloworld \
  --builder=gcr.io/buildpacks/builder
```
{% endtab %}
{% endtabs %}

### Deploy

```bash
PROJECT_ID=$(gcloud config get-value project)
kubectl create deployment helloworld \
  --image=gcr.io/${PROJECT_ID}/helloworld
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

Find the public load balancer IP address:

```text
kubectl get services helloworld
```

Then connect with `curl`:

```bash
EXTERNAL_IP=$(kubectl get svc helloworld \
  -ojsonpath='{.status.loadBalancer.ingress[0].ip})
curl http://${EXTERNAL_IP}
```

## Learn More

* [Kubernetes from Basic to Advanced code lab](https://bit.ly/k8s-lab)
* [Spring Boot on GCP code lab](https://bit.ly/spring-gcp-lab)
* [Spring to Kubernetes Faster and Easier](https://saturnism.me/talk/kubernetes-spring-java-best-practices/)

