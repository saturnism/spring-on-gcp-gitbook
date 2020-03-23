---
description: >-
  Install gcloud command to interact with Google Cloud Platform from the command
  line.
---

# gcloud CLI

## Install

Follow the [official gcloud installation guide](https://cloud.google.com/sdk/docs/downloads-interactive#linux) for your platform.

## Authenticate

Authenticate gcloud so that it can interact with Google Cloud Platform using your account.

```text
gcloud auth login
```

## Project ID

Configure the default project ID to your project.

```text
gcloud config set project YOUR_PROJECT_ID
```

## Application Default Credentials

In addition to authenticating gcloud, also authenticate Application Default Credentials \(ADC\). ADC is used by your application/microservices during local development to authenticate with cloud services.

```text
gcloud auth application-default login
```

