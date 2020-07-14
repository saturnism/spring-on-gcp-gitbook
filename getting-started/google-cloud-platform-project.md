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

### Principal / Member

All Principals \(i.e., a user\) are identified by an e-mail address:

| Type | Uses | Identified By |
| :--- | :--- | :--- |
| User Account | User interaction with `gcloud` CLI, or the web console. | User's e-mail address |
| Service Account | Service to Service authentication | Service account's e-mail address |
| G Suite Group | A collection of user accounts or service accounts. | G Suite Group e-mail |
| G Suite Domain | All users and groups of a G Suite domain. | G Suite domain name |

Sometimes, when referring to different type of Principals, you may need to add a prefix:

| Type | Prefix | Example |
| :--- | :--- | :--- |
| User Account | user | user:jane@example.com |
| Service Account | serviceAccount | serviceAccount:my-service@appspot.gserviceaccount.com |
| G Suite Group | group | group:webmaster@example.com |
| G Suite Domain | domain | domain:example.com |

See [Identity Access Management Overview documentation](https://cloud.google.com/iam/docs/overview) for more details.

### Roles and Permissions

Each Principal can be associated with [different Roles](https://cloud.google.com/iam/docs/understanding-roles), and each Role is associated with specific Permissions.  For example, a `viewer` role may have fewer permissions than an `editor` role, which may have fewer permissions than an `admin` role.  See [Understanding roles documentation](https://cloud.google.com/iam/docs/understanding-roles) for all the available roles and the associated permissions

{% hint style="info" %}
You can create [Custom Roles](https://cloud.google.com/iam/docs/understanding-custom-roles) to associate with specific permissions too.
{% endhint %}

### Credential

| Type | Credential |
| :--- | :--- |
| User Account | OAuth credentials - an Access Token, or a Refresh Token, or [Application Default Credentials](google-cloud-platform-project.md#application-default-credentials). |
| Service Account | A [Service Account Key file](https://cloud.google.com/iam/docs/creating-managing-service-account-keys) **or** from [Machine Credentials from Metadata Server](https://cloud.google.com/compute/docs/storing-retrieving-metadata) |

{% hint style="info" %}
User Account is great for local development when using `gcloud`.  Service Account is great for your application/microservice.
{% endhint %}

#### Application Default Credentials

This is the default credential that a Google Cloud client library will discover.  Application Default Credential can be:

* Created by `gcloud auth application-default login` when running locally,
* **or** a `GOOGLE_APPLICATION_CREDENTIALS` environmental variable that points to the path of a Service[ Account key file](google-cloud-platform-project.md#service-account-key),
* **or** automatically discovered using the [Metadata Server](google-cloud-platform-project.md#machine-credentials). 

When using a Google Cloud client library to access a Cloud service, the client library would automatically discover the credential to use based on precedence. See [Google Auth Library README for more information](https://github.com/googleapis/google-auth-library-java/blob/master/README.md#application-default-credentials).

#### Service Account Key

Service Account Key file is a JSON file that contains a private key, and the private key is used to retrieve OAuth access token. The Service Account file is like a password and must be stored securely!

{% hint style="danger" %}
Never expose the service account key file in the public.

Never check-in your service account key file.

Never put your service account key file in a container image, or deployable artifact like a JAR file.
{% endhint %}

{% hint style="success" %}
Always store your service account securely.
{% endhint %}

{% hint style="success" %}
In most cases, your application is associated with a service account, but will **not** need the Service Account key file. See [Machine Credentials](google-cloud-platform-project.md#machine-credentials-from-metadata-server).
{% endhint %}

#### Machine Credentials from Metadata Server

All Google Cloud runtime environments \(App Engine, Cloud Functions, Cloud Run, Kubernetes Engine, Compute Engine, ...\) have access to the [Metadata Server](https://cloud.google.com/compute/docs/storing-retrieving-metadata). From the runtime environment, you can retrieve the current access token associated with the Service Account:

```text
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token  
```

{% hint style="info" %}
Each runtime / service may be associated with a specific Service Account. For example, VM1 uses Service Account A, and VM2 uses Service Account B. Depending on which VM is used to access the Metadata Server, the Metadata Server will return the token for the associated Service Account.
{% endhint %}

