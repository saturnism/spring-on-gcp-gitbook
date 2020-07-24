# Cloud Run

[Cloud Run](https://cloud.google.com/run/docs) is a fully managed container runtime environment, where you can deploy any HTTP serving containers, and Cloud Run will automatically scale out the number of instances as needed, and scale down to zero when no one is using it.

## Getting Started - Click to Deploy

You can deploy a [Hello World Application](https://github.com/saturnism/jvm-helloworld-by-example/tree/master/helloworld-springboot-tomcat) simply by click on the **Run on Google Cloud** button below!

[![Deploy a Spring Boot app on Cloud Run](https://deploy.cloud.run/button.svg)](https://deploy.cloud.run/?git_repo=https://github.com/saturnism/jvm-helloworld-by-example.git&dir=helloworld-springboot-tomcat)

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
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

### Build

```bash
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

After the image is built, push the docker image to Container Registry:

```bash
docker push gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
Learn about [Paketo Buildpack](https://paketo.io/) and [GCP Buildpack](https://github.com/GoogleCloudPlatform/buildpacks).
{% endhint %}
{% endtab %}
{% endtabs %}

### Deploy

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud run deploy helloworld \
  --region=us-central1 \
  --platform=managed \
  --allow-unauthenticated \
  --image=gcr.io/${PROJECT_ID}/helloworld
```

### Connect

Once deployed, Cloud Run will display the HTTPs URL. You can also find the URL with the command line:

```text
gcloud run services describe helloworld \
  --region=us-central1 \
  --platform=managed 
```

You can `curl` the URL:

```bash
URL=$(gcloud run services describe helloworld \
  --region=us-central1 \
  --platform=managed \
  --format='value(status.address.url)')
curl ${URL}
```

## Additional Configurations

By default, Cloud Run will deploy with the smallest 1CPU 256MB instance. You can specify a larger instance, and configure environment variables with the `gcloud` CLI:

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud run deploy helloworld --platform=managed --allow-unauthenticated \
  --cpu=2 --memory=512M --set-env-vars="SPRING_PROFILES_ACTIVE=prod"
  --image=gcr.io/${PROJECT_ID}/helloworld
```

## Learn More

* [Optimizing Java Applications on Cloud Run](https://cloud.google.com/run/docs/tips/java)

