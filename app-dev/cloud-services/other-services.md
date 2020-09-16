# Other Services

Spring Cloud GCP has idiomatic integrations and starters for a number of Google Cloud services, but not all services. There may be cases where you need to use a Google Cloud client library directly. In this case, you can re-use basic bootstrapping provided by the Spring Cloud GCP, so you can have a consistent way of specifying credentials for your application.

## Dependency

Spring Cloud GCP already imports the [Google Cloud Java BOM](https://github.com/googleapis/java-cloud-bom), and it already has encoded the client library versions. So you can specify any Google Cloud client library without explicitly specifying a version. This is great to ensure that you are using a compatible version of a Google Cloud client library.

For example, to add Container Analysis client library:

{% tabs %}
{% tab title="Maven" %}
```markup
<dependency>
  <groupId>com.google.cloud</groupId>
  <artifactId>google-cloud-containeranalysis</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```groovy
compile group: 'com.google.cloud', name: 'google-cloud-containeranalysis'
```
{% endtab %}
{% endtabs %}

## Credentials

You need Google Cloud credentials to access any services. [Spring Cloud GCP Core](https://docs.spring.io/spring-cloud-gcp/docs/1.2.5.RELEASE/reference/html/#credentials) produces a `CredentailsProvider` bean so you can re-use the same credentials.

Usually you only need a singleton instance of the client library, so it makes sense to configuring it as a Spring Bean. Most client libraries needs to be shutdown gracefully, so you should specify the `destroyMethod` as well:

```java
@Bean(destroyMethod = "shutdownNow")
ContainerAnalysisClient containerAnalysisClient(CredentialsProvider credentialsProvider) throws IOException {
  return ContainerAnalysisClient.create(
    ContainerAnalysisSettings.newBuilder()
      .setCredentialsProvider(credentialsProvider).build());
}
```

## Project ID

In rare cases, you may want to know which Project ID you are currently configured to use by default. You can find out from the [`GcpProjectIdProvider` bean](https://docs.spring.io/spring-cloud-gcp/docs/1.2.5.RELEASE/reference/html/#project-id).

