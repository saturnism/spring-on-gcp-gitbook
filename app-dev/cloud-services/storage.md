# Storage

## Cloud Storage

Cloud Storage provides globally unified, scalable, and highly durable object storage. You can store files in Cloud Storage without having to worry about running out of space, or managing your own filesystems.

Cloud Storage buckets can be configured to store files in a single region, dual-region, or multi-region.

| Location | Use Case |
| :--- | :--- |
| Region | Use a region to help optimize latency and network bandwidth for data consumers, such as analytics pipelines, that are grouped in the same region. |
| Dual-Region | Use a dual-region when you want similar performance advantages as regions, but also want the higher availability that comes with being [geo-redundant](https://cloud.google.com/storage/docs/key-terms#geo-redundant). |
| Multi-Region | Use a multi-region when you want to serve content to data consumers that are outside of the Google network and distributed across large geographic areas, or when you want the higher availability that comes with being [geo-redundant](https://cloud.google.com/storage/docs/key-terms#geo-redundant). |

{% hint style="info" %}
See [Cloud Storage Bucket locations](https://cloud.google.com/storage/docs/locations) for more information.
{% endhint %}

Cloud Storage buckets can also be configured with a different Storage Class, for different access patterns to optimize cost.

| Storage Class | Use Case |
| :--- | :--- |
| Standard | Standard Storage is best for data that is frequently accessed \("hot" data\) and/or stored for only brief periods of time. |
| Nearline | Nearline Storage is a low-cost, highly durable storage service for storing infrequently accessed data. Nearline Storage is a better choice than Standard Storage in scenarios where slightly lower availability, a 30-day minimum storage duration. |
| Coldline | Coldline Storage is a very-low-cost, highly durable storage service for storing infrequently accessed data. Coldline Storage is a better choice than Standard Storage or Nearline Storage in scenarios where slightly lower availability, a 90-day minimum storage duration. |
| Archive | Archive Storage is the lowest-cost, highly durable storage service for data archiving, online backup, and disaster recovery. Unlike the "coldest" storage services offered by other Cloud providers, your data is available within milliseconds, not hours or days. |

{% hint style="info" %}
See [Cloud Storage Classes](https://cloud.google.com/storage/docs/storage-classes) for more information.
{% endhint %}

### Enable API

```bash
gcloud services enable storage-component.googleapis.com
```

### Create a Bucket

A bucket is the top-level directory that you can add additional files and sub-directories into. A bucket name must be globally unique. For example, create a bucket with the same name as your Google Cloud project.

```bash
PROJECT_ID=$(gcloud config get-value project)
gsutil mb gs://$PROJECT_ID
```

### Copy a file

```bash
echo "Hello World" > hello.txt
gsutil cp hello.txt gs://$PROJECT_ID
```

### Delete a file

```bash
gsutil rm gs://$PROJECT_ID/hello.txt
```

## Spring Resource

With Spring Cloud GCP, you can use [Spring Resource](https://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/resources.html) to perform store and retrieve file from Cloud Storage.

### Dependency

Add the Spring Cloud GCP Storage starter:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-storage</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash

compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-starter-storage'
```
{% endtab %}
{% endtabs %}

### Configuration

There is no explicit configuration required if you use the automatic authentication and project ID detection. I.e., if you already logged in locally with `gcloud` command line, then it'll automatically access buckets that you have access to.

{% hint style="info" %}
Notice that there is no explicit configuration for username/password. Cloud Storage authentication uses the GCP credential \(either your user credential, or Service Account credential\), and authorization is configured via Identity Access Management \(IAM\).
{% endhint %}



