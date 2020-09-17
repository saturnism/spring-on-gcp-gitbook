---
description: >-
  Install gcloud command to interact with Google Cloud Platform from the command
  line.
---

# gcloud CLI

## Cloud Shell

Cloud Shell already has the `gcloud` CLI pre-installed, so you can skip ahead to [configure default zone and region](gcloud-cli.md#default-zone-and-region).

## Local Installation

To install the `gcloud` CLI on your local machine, follow the [official installation guide](https://cloud.google.com/sdk/docs/downloads-interactive) for your platform and then follow the below steps to finish configuration.

### Authenticate

Authenticate gcloud so that it can interact with Google Cloud Platform using your account.

```bash
gcloud auth login
```

### Project ID

Set the default Project ID to your project.

```bash
gcloud config set project YOUR_PROJECT_ID
```

{% hint style="info" %}
If you already have a project, run `gcloud projects list` to list available projects. Find one and then set it as a default project.
{% endhint %}

### Application Default Credentials

In addition to authenticating gcloud, also authenticate Application Default Credentials \(ADC\). ADC is used by your application/microservices during local development to authenticate with cloud services.

```bash
gcloud auth application-default login
```

### Quota Project

API calls to Google Cloud may be rate limited and subject to quotas. The quotas are typically tied to a Google Cloud Project. When you are running the application locally, and using the Application Default Credentials, the requests need to be associated with a project to account for usage quota. Typically, the Quota Project should be the same as the project that you are currently working with. In a larger organization, it can be a Project that's used for development purposes and not the production project.

When configuring the Application Default Credentials the first time, It will also configure a Quota Project to be the same as the default project you previously configured.

If needed, you can configure a different Quota Project:

```bash
gcloud auth application-default set-quota-project YOUR_PROJECT_ID
```

{% hint style="info" %}
Application Default Credentials are used by client libraries when making calls to Google Cloud. This is different from the [first gcloud Authenticate](gcloud-cli.md#authenticate), which is for `gcloud` to make calls to Google Cloud.
{% endhint %}

This will store the credential \(OAuth refresh token\) in a well-known location, such as `~/.config/gcloud/application_default_credentials.json`. Google Cloud client libraries can automatically detect this file and use this credential.

### Default Zone and Region

A cloud resource can be Zonal, Regional, or Multi-Regional. For example, a VM is Zonal, because it can only live in a single availability zone. App Engine service is regional, because it's automatically distributed across multiple zones within a single Region. Cloud Storage can store your data in a Regional bucket, or a Multi-Regional bucket.

You can always specify the `zone` or `region` with each of the `gcloud` command. If you primarily operate within a single zone or region, set the default `zone` and default `region`.

```bash
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-c
gcloud config set run/region us-central1
```

{% hint style="info" %}
See the complete list in [Regions and Zones documentation](https://cloud.google.com/compute/docs/regions-zones).
{% endhint %}
