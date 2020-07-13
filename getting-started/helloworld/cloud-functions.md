# Cloud Functions

[Cloud Function](https://cloud.google.com/functions/docs/) is a scalable pay as you go Functions-as-a-Service \(FaaS\) to run your code with zero server managements.

## Getting Started

### Enable API

```text
# To use Cloud Functions
gcloud services enable cloudfuncgtions.googleapis.com
```

### Clone

```bash
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

Alternatively, for a quick test, you can also use `gcloud`:

```bash
gcloud functions call helloworld
```

## Additional Configurations

By default, Cloud Functions will deploy with the smallest 256MB instance. You can specify a larger instance, and configure environment variables with the `gcloud` CLI:

```bash
gcloud functions deploy helloworld --trigger-http \
  --runtime=java11 \
  --memory=512M
  --entry-point=functions.HelloWorld \
  --allow-unauthenticated
```







