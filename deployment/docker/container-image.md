---
description: Build container images with tools and best practices.
---

# Container Image

## Clone

```bash
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

## Build

```bash
./mvnw package
```

## Enable API

Enable Container Registry API to push your container images to the Container Registry.

```bash
gcloud services enable containerregistry.googleapis.com
```

## Containerize

Typically, tutorials teach you how to write a `Dockerfile` to containerize a Java application. A `Dockerfile` can be error prone and it's hard to implement all the best practices. Rather than writing a `Dockerfile`, use tools such as Jib and Buildpacks to automatically create optimized container images.

### Build and Push

Most tools can build and push directly into a container registry. In case of Jib, this step does not require a Docker daemon at all, and it can push changed layers directly into a remote registry. This is great for automated CI/CD pipelines.

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
1. Install `pack` CLI - see [Installing `pack` documentation](https://buildpacks.io/docs/install-pack/)
1. Build container with `pack`, and use `--publish` flag to push directly to the remote registry:

```bash
# Paketo Buildpack
PROJECT_ID=$(gcloud config get-value project)
pack build \
  --builder gcr.io/paketo-buildpacks/builder:base \
  --publish \
  gcr.io/${PROJECT_ID}/helloworld

# GCP Buildpack
PROJECT_ID=$(gcloud config get-value project)
pack build \
  --builder gcr.io/buildpacks/builder:v1 \
  --publish \
  gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
Learn about [Paketo Buildpack](https://paketo.io/) and [GCP Buildpack](https://github.com/GoogleCloudPlatform/buildpacks).
{% endhint %}

{% hint style="danger" %}
Paketo Buildpack will calculate the minimum memory needed to run the Spring Boot application. For a Hello World example, the minimum is 1GB of RAM.
{% endhint %}
{% endtab %}

{% tab title="Buildpack with Cloud Build" %}
Cloud Build has built-in Buildpack support, so you can build the container image in the remote Cloud Build environment:

```bash
# GCP Buildpack
PROJECT_ID=$(gcloud config get-value project)
gcloud alpha builds submit \
  --pack image=gcr.io/${PROJECT_ID}/helloworld

# Paketo Buildpack
PROJECT_ID=$(gcloud config get-value project)
gcloud alpha builds submit \
  --pack image=gcr.io/${PROJECT_ID}/helloworld,builder=gcr.io/paketo-buildpacks/builder:base
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

After the image is built, push the docker image to Container Registry:

```bash
docker push gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
Learn about [Paketo Buildpack](https://paketo.io/) and [GCP Buildpack](https://github.com/GoogleCloudPlatform/buildpacks).
{% endhint %}

{% hint style="danger" %}
Paketo Buildpack will calculate the minimum memory needed to run the Spring Boot application. For a Hello World example, the minimum is 1GB of RAM.
{% endhint %}
{% endtab %}
{% endtabs %}

### Build Locally

If you are running a local Docker daemon and you do not want to push straight to a remote registry, then you can build container images without pushing:

{% tabs %}
{% tab title="Jib" %}
```bash
PROJECT_ID=$(gcloud config get-value project)
./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:dockerBuild \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```
{% endtab %}

{% tab title="Buildpack" %}
```bash
# Paketo Buildpack
PROJECT_ID=$(gcloud config get-value project)
pack build \
  --builder gcr.io/paketo-buildpacks/builder:base \
  gcr.io/${PROJECT_ID}/helloworld

# GCP Buildpack
PROJECT_ID=$(gcloud config get-value project)
pack build \
  --builder gcr.io/buildpacks/builder:v1 \
  gcr.io/${PROJECT_ID}/helloworld
```
{% endtab %}

{% tab title="Spring Boot 2.3" %}
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

## Run Locally

If you have Docker installed locally, you can run the docker container locally to ensure everything works. This command will run the container locally and forward localhost's port 8080 to the container instance's port 8080.

```bash
PROJECT_ID=$(gcloud config get-value project)

docker pull gcr.io/${PROJECT_ID}/helloworld
docker run -ti --rm -p 8080:8080 gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
The `-ti` flag means allocate a `TTY`, and expect interaction via `STDIN`. The `--rm` flag means delete the container completely upon exit.
{% endhint %}

## Connect Locally

You can connect to the container that's running locally

```bash
curl localhost:8080
```
