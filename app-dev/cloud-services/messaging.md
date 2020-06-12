# Messaging

## Cloud Pub/Sub

[Cloud Pub/Sub](https://cloud.google.com/pubsub/docs/) is a managed publish/subscribe service, where you can send messages to a topic, and subscribe via push, pull, or streaming pull.  A single Cloud Pub/Sub Topic can be associated with one or more Subscriptions. Each Subscription can have one or more subscribers. Cloud Pub/Sub delivers messages with guaranteed at-least-once delivery, and there is no ordering guarantee.

### Enable API

```bash
gcloud services enable pubsub.googleapis.com
```

### Create a Topic

```bash
gcloud pubsub topics create orders
```

### Create a Pull Subscription

```bash
gcloud pubsub subscriptions create orders-subscription --topic=orders
```

### Publish a Message

```bash
gcloud pubsub topics publish orders --message="my order"
```

### Pull a Message

```bash
gcloud pubsub subscriptions pull orders-subscription --auto-ack
```

## Spring Cloud Pub/Sub

The easiest way to use Cloud Pub/Sub is using Spring Cloud GCP's [Spring Pub/Sub starter](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/1.2.2.RELEASE/reference/html/#google-cloud-pubsub). This starter provides easy to use `PubSubTemplate` bean to send and receive messages.

### Dependency

Add the Spring Data Datastore starter:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-pubsub-starter</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash

compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-pubsub-starter'
```
{% endtab %}
{% endtabs %}

### Configuration

There is no explicit configuration required if you use the automatic authentication and project ID detection. I.e., if you already logged in locally with `gcloud` command line, then it'll automatically use Datastore from the project you configured in `gcloud`.

{% hint style="info" %}
Notice that there is no explicit configuration for username/password. Cloud Firestore authentication uses the GCP credential \(either your user credential, or Service Account credential\), and authorization is configured via Identity Access Management \(IAM\).
{% endhint %}

### 

### Pub/Sub Template

### Spring Integration

### Spring Cloud Stream

