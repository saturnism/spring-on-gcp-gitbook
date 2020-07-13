# Kubernetes Engine

## Getting Started \(Cloud\)

### Enable API

```text
gcloud services enable compute.googleapis.com container.googleapis.com
```

### Create Cluster

```text
gcloud container clusters create helloworld-cluster \
  --num-nodes 2 \
  --machine-type n1-standard-1 \
  --zone us-central1-c
```

### Clone

```text
git clone https://github.com/jamesward/hello-springboot-mvn
cd hello-springboot-mvn
```

### Build

```text
./mvnw package
```

### Containerize

{% tabs %}
{% tab title="Jib" %}
[Jib](https://github.com/GoogleContainerTools/jib) can containerize any Java application easily, without a `Dockerfile` nor `docker` installed.

```bash
PROJECT_ID=$(gcloud config get-value project)
./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
You can configure Jib plugin in Maven or Gradle build file to run the Jib easier, such as `./mvnw jib:build`.
{% endhint %}
{% endtab %}

{% tab title="Buildpack" %}
[Cloud Native Buildpacks](https://buildpacks.io) can containerize applications written in different language without a `Dockerfile`. It does require `docker` installed.

1. Install Docker locally - see [Get Docker documentation](https://docs.docker.com/get-docker/).
2. Install `pack` CLI - see [Installing `pack` documentation](https://buildpacks.io/docs/install-pack/)
3. Build container with `pack`:

```bash
PROJECT_ID=$(gcloud config get-value project)
pack build --builder gcr.io/buildpacks/builder:v1 \
  --publish \
  gcr.io/${PROJECT_ID}/helloworld
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

```text
kubectl create service loadbalancer helloworld --tcp=8080:8080
```

Find the public load balancer IP address:

```text
kubectl get services
```

