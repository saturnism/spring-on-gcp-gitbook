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

## Default Zone and Region

A cloud resource can be Zonal, Regional, or Multi-Regional. For example, a VM is Zonal, because it can only live in a single availability zone. App Engine service is regional, because it's automatically distributed across multiple zones within a single Region.  Cloud Storage can store your data in a Regional bucket, or a Multi-Regional bucket.

You can always specify the `zone` or `region` with each of the `gcloud` command. If you primarily operate within a single zone or region, set the default `zone` and default `region`.

```bash
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-c
```

{% hint style="info" %}
See the complete list in [Regions and Zones documentation](https://cloud.google.com/compute/docs/regions-zones).
{% endhint %}

## Application Default Credentials

In addition to authenticating gcloud, also authenticate Application Default Credentials \(ADC\). ADC is used by your application/microservices during local development to authenticate with cloud services.

```text
gcloud auth application-default login
```

