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

## Containerize

Typically, tutorials teach you how to write a `Dockerfile` to containerize a Java application. A `Dockerfile` however can be error prone and it's hard to implement all the best practices. Rather than writing a `Dockerfile`, use tools such as Jib and Buildpacks to automatically create optimized container images.

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

After the image is built, push the docker image to Container Registry:

```bash
docker push gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
Learn about [Paketo Buildpack](https://paketo.io/) and [GCP Buildpack](https://github.com/GoogleCloudPlatform/buildpacks).
{% endhint %}

{% hint style="danger" %}
Paketo Buildpack will calculate the minimum memory needed to run the Spring Boot application. For a this Hello World example, the minimum is 1GB of RAM.
{% endhint %}
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

## 

