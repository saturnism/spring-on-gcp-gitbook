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

### Storage Client

The starter automatically creates a pre-configured `Storage` bean that provides raw-access to Google Cloud Storage.

```java
@Bean
ApplicationRunner storageRunner(Storage storage, GcpProjectIdProvider projectIdProvider) {
  return (args) -> {
    Page<Blob> list = storage.list(projectIdProvider.getProjectId());
    list.iterateAll().forEach(blob -> System.out.println(blob.getName()));
  };
}
```

### Resource URI

You can address a Cloud Storage file by using the resource URI prefixed with `gs://`. The fully qualified URI is of the form: `gs://project-id/path/to/file`.

### Read a file

You can open the `Resource` using `ApplicationContext`. Then read the content from `InputStream`. You must `close` the stream when you are done \(or wrap with try-with-resource since it's auto-closeable\).

```java
@Bean
ApplicationRunner runner(ApplicationContext ctx, GcpProjectIdProvider projectIdProvider) {
  return (args) -> {
    WritableResource resource = (WritableResource) ctx
        .getResource(String.format("gs://%s/hello.txt", projectIdProvider.getProjectId()));
    try (PrintWriter writer = new PrintWriter(resource.getOutputStream())) {
      writer.println("Hello World!");
    }
  };
}
```

### Write a file

You can open the `Resource` using `ApplicationContext`. Then cast the `Resource` to a `WritableResource`, and then use the `OutputStream` to write the content. Lastly, you must `close` the stream in order for the file to write \(or wrap with try-with-resource since it's auto-closeable\).

```java
@Bean
ApplicationRunner readRunner(ApplicationContext ctx, GcpProjectIdProvider projectIdProvider) {
  return (args) -> {
    Resource resource = ctx
        .getResource(String.format("gs://%s/hello.txt", projectIdProvider.getProjectId()));
    try (InputStreamReader reader = new InputStreamReader(resource.getInputStream())) {
      BufferedReader bufferedReader = new BufferedReader(reader);
      bufferedReader.lines().forEach(System.out::println);
    }
  };
}
```

## Spring Integration

You can channel adapters for Google Cloud Storage to read and write files to Google Cloud Storage through `MessageChannels`.

### Dependency

Add both the Spring Cloud GCP Storage starter, and Spring Integration File component.

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-storage</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-file</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash

compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-starter-storage'
compile group: 'org.springframework.integration', name: 'spring-integration-file'
```
{% endtab %}
{% endtabs %}

### Inbound Channel Adapter

#### Inbound File Synchronizer

File-based integration with files typically requires polling a directory that contains the new files.  In Spring Integration, this is configured through an `InboundFileSynchronizer`. Use `GcsInboundFileSyncronizer` to create a `MessageSource` and adapt it to a `MessageChannel`.

{% hint style="warning" %}
The files are temporarily stored in a directory in the local file system.
{% endhint %}

```java
@Bean
public MessageChannel gcsInputChannel() {
  return MessageChannels.direct().get();
}

@Bean
@InboundChannelAdapter(channel = "gcsInputChannel", poller = @Poller(fixedDelay = "5000"))
public MessageSource<File> (Storage gcs, GcpProjectIdProvider projectIdProvider)
    throws IOException {
  GcsInboundFileSynchronizer synchronizer = new GcsInboundFileSynchronizer(gcs);
  synchronizer.setRemoteDirectory(projectIdProvider.getProjectId());

  GcsInboundFileSynchronizingMessageSource messageSource =
          new GcsInboundFileSynchronizingMessageSource(synchronizer);
  File localDirectory = Files.createTempDirectory("gcs");
  messageSource.setLocalDirectory(localDirectory);

  return messageSource;
}
```

#### Streaming Message Source

For most use cases, you should use the streaming message source, which does not require files to be stored in the file system.

```java
@Bean
public MessageChannel gcsInputChannel() {
  return MessageChannels.direct().get();
}

@Bean
@InboundChannelAdapter(channel = "gcsInputChannel", poller = @Poller(fixedDelay = "5000"))
public MessageSource<InputStream> streamingAdapter(Storage gcs, GcpProjectIdProvider projectIdProvider) {
  GcsStreamingMessageSource adapter =
          new GcsStreamingMessageSource(new GcsRemoteFileTemplate(new GcsSessionFactory(gcs)));
  adapter.setRemoteDirectory(projectIdProvider.getProjectId());
  return adapter;
}
```





  




