# Cloud Run

Cloud Run is a fully managed container runtime environment, where you can deploy any HTTP serving containers, and Cloud Run will automatically scale out the number of instances as needed, and scale down to zero when no one is using it.

## Getting Started - Click to Deploy

You can deploy a [Hello World Application](https://github.com/jamesward/hello-springboot-mvn.git) simply by click on the **Run on Google Cloud** button below!

[![Deploy a Spring Boot app on Cloud Run](https://deploy.cloud.run/button.svg)](https://deploy.cloud.run/?git_repo=https://github.com/jamesward/hello-springboot-mvn.git)

## Getting Started - Manual Deployment

### Enable API

```text
# To use Cloud Run
gcloud services enable run.googleapis.com

# To store Container Images in Container Registry
gcloud services enable containerregistry.googleapis.com
```

### Clone

```bash
git clone https://github.com/jamesward/hello-springboot-mvn
cd hello-springboot-mvn
```

### Build

```bash
./mvnw package
```

### Containerize

{% tabs %}
{% tab title="Jib" %}
[Jib](https://github.com/GoogleContainerTools/jib) can containerize any Java application easily, without a `Dockerfile` nor `docker` installed.

```bash
PROJECT_ID=$(gcloud config get-value project)
./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}>/helloworld
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
gcloud run deploy helloworld --platform=managed --allow-unauthenticated \
  --image=gcr.io/${PROJECT_ID}/helloworld
```

## Additional Configurations

By default, Cloud Run will deploy with the smallest 1CPU 256MB instance. You can specify a larger instance, and configure environment variables with the `gcloud` CLI:

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud run deploy helloworld --platform=managed --allow-unauthenticated \
  --cpu=2 --memory=512M --set-env-vars="GREETING=Hola!"
  --image=gcr.io/${PROJECT_ID}/helloworld
```

## Learn More

* [Optimizing Java Applications on Cloud Run](https://cloud.google.com/run/docs/tips/java)

