---
description: Deploy a simple HTTP function.
---

# Cloud Functions

[Cloud Function](https://cloud.google.com/functions/docs/) is a scalable, pay as you go, Functions-as-a-Service \(FaaS\).

Spring Cloud Functions has pre-GA support for Cloud Functions for Java 11. See [Spring Cloud Functions Reference Documentation](https://docs.spring.io/spring-cloud-function/docs/current/reference/html/gcp.html) for more details.

This guide currently uses a non-Spring example for Cloud Functions.

{% embed url="https://www.youtube.com/watch?v=UsYRKkibLPI" caption="" %}

## Getting Started

### Clone

```bash
cd $HOME
git clone https://github.com/GoogleCloudPlatform/java-docs-samples
cd java-docs-samples/functions/helloworld/helloworld
```

### Build

```bash
mvn package
```

### Run Locally

```bash
mvn function:run

# In a different tab, trigger the function:
curl localhost:8080
```

### Deploy

#### Enable API

```bash
gcloud services enable cloudfunctions.googleapis.com
```

#### Deploy

```bash
gcloud functions deploy helloworld --trigger-http \
  --runtime=java11 \
  --entry-point=functions.HelloWorld \
  --allow-unauthenticated
```

### Connect

Once a HTTP function is deployed, you can connect to it using `curl`. You can also find the URL:

```bash
gcloud functions describe helloworld --format='value(httpsTrigger.url)'
```

Trigger the function with `curl`:

```bash
URL=$(gcloud functions describe helloworld --format='value(httpsTrigger.url)')

curl ${URL}
```

Alternatively, you can also use `gcloud`:

```bash
gcloud functions call helloworld
```

## Additional Configurations

By default, Cloud Functions will deploy to the smallest 256MB instance. You can specify a larger instance and configure environment variables with the `gcloud` CLI:

```bash
gcloud functions deploy helloworld --trigger-http \
  --runtime=java11 \
  --memory=512M
  --entry-point=functions.HelloWorld \
  --allow-unauthenticated
```

## Learn More

* [Cloud Functions Java Runtime documentation](https://cloud.google.com/functions/docs/concepts/java-runtime)
* [Framework support](https://cloud.google.com/functions/docs/concepts/java-frameworks) for [Spring Cloud Functions](https://cloud.spring.io/spring-cloud-static/spring-cloud-function/3.0.7.RELEASE/reference/html/gcp.html), [Micronaut](https://micronaut-projects.github.io/micronaut-gcp/2.0.x/guide/#cloudFunction), [Quarkus](https://quarkus.io/guides/gcp-functions)
