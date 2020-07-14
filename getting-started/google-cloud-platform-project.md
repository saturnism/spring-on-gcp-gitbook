---
description: >-
  Get started on Google Cloud Platform by signing up a free account and creating
  a new project to use.
---

# Google Cloud Platform

## Sign Up

If you don't already use [Google Cloud Platform](http://cloud.google.com/), sign up for an account and [get started for free](http://cloud.google.com/freetrial), and receive $300 credit for the first 12 months.

## Project

All cloud services and resources \(such as virtual machines, network, load balancer, etc\) are to be created under a Google Cloud Platform project.

A project is a billing unit. Any services / resources you create under the project will be charged to the Billing Account associated with the project.

A project is  a security boundary. You can assign additional users to access different services / resources within the project.

Project is usually referred by Project ID. A Project ID is globally unique.

### New Account

If this is your first time signing up for Google Cloud Platform, it will automatically create a Google Cloud Platform Project.

![Google Cloud Platform console with a default project](../.gitbook/assets/image%20%281%29.png)

Every project has a Project ID and a Project Number. Project ID is most used. Find the Project ID in **Home**, under **Project info**.

![Project info panel showing the Project ID](../.gitbook/assets/image%20%282%29.png)

### Existing Account

If you already have an account, use an existing project, or create a new one.

## Identity Access Management

IAM may be one of the hardest concepts to grasp about Google Cloud Platform - but once you understand it, everything else becomes clear.

### Principal

All authentication Principals \(i.e., a user\) are identified as an e-mail address:

| Type | Uses | Identified By |
| :--- | :--- | :--- |
| User Account | User interaction with `gcloud` CLI, or the web console. | User's E-Mail address |
| Service Account | Service to Service authentication | Service account's E-Mail address |
| G Suite Group | A collection of user accounts or service accounts. | G Suite Group E-Mail |

### Roles and Permissions

Each Principal can be associated with [different Roles](https://cloud.google.com/iam/docs/understanding-roles), and each Role is associated with specific Permissions.  For example, a `viewer` role may have fewer permissions than an `editor` role, which may have fewer permissions than an `admin` role.  See [Understanding roles documentation](https://cloud.google.com/iam/docs/understanding-roles) for all the available roles and the associated permissions

{% hint style="info" %}
You can create [Custom Roles](https://cloud.google.com/iam/docs/understanding-custom-roles) to associate with specific permissions too.
{% endhint %}

### Credential

| Type | Credential |
| :--- | :--- |
| User Account | OAuth credentials - an Access Token, or a Refresh Token |
| Service Account | A Service Account Key file **or** from [Metadata Server](https://cloud.google.com/compute/docs/storing-retrieving-metadata) |

All Google Cloud runtime environments \(App Engine, Cloud Functions, Cloud Run, Kubernetes Engine, Compute Engine, ...\) have access to the [Metadata Server](https://cloud.google.com/compute/docs/storing-retrieving-metadata). From the runtime environment, you can retrieve the current access token associated with the Service Account:

```text
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token  
```

