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

{% hint style="info" %}
This authenticated `gcloud` so that you can run all the commands.
{% endhint %}

## Project ID

Configure the default project ID to your project.

```text
gcloud config set project YOUR_PROJECT_ID
```

{% hint style="info" %}
If you already have a project can list using the following command: `gcloud projects list`. Find one and then setting it as a default project.
{% endhint %}

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

It will also configure a Quota Project to be the same as the default project you previously configured. If needed, you configure a different Quota Project:

```bash
gcloud auth application-default set-quota-project YOUR_PROJECT_ID
```

{% hint style="info" %}
Application Default Credentials authentication is for client libraries to make calls to Google Cloud. This is different from the [first gcloud Authenticate](install-gcloud-cli.md#authenticate), which is for `gcloud` to make calls to Google Cloud.
{% endhint %}

This will store the credential \(OAuth refresh token\) in a well-known location, such as `~/.config/gcloud/application_default_credentials.json`. Google Cloud client libraries can automatically detect this file and use this credential.

