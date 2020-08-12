# Cloud Run

[Cloud Run](https://cloud.google.com/run/docs) is a fully managed container runtime environment, where you can deploy any HTTP serving containers, and Cloud Run will automatically scale out the number of instances as needed, and scale down to zero when no one is using it.

## Getting Started - Click to Deploy

You can deploy a [Hello World Application](https://github.com/saturnism/jvm-helloworld-by-example/tree/master/helloworld-springboot-tomcat) simply by click on the **Run on Google Cloud** button below!

[![Deploy a Spring Boot app on Cloud Run](https://deploy.cloud.run/button.svg)](https://deploy.cloud.run/?git_repo=https://github.com/saturnism/jvm-helloworld-by-example.git&dir=helloworld-springboot-tomcat)

## Getting Started - Manual Deployment

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

### Deploy

#### Enable API

```text
# To use Cloud Run
gcloud services enable run.googleapis.com
```

#### Deploy Container

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
  --cpu=2 --memory=512M --set-env-vars="SPRING_PROFILES_ACTIVE=prod" \
  --image=gcr.io/${PROJECT_ID}/helloworld
```

## Learn More

* [Optimizing Java Applications on Cloud Run](https://cloud.google.com/run/docs/tips/java)

