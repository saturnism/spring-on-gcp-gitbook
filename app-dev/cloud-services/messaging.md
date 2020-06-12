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
gcloud pubsub topics publish orders \
  --message='{"id":"1", "description": "My Order"}'
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
    <artifactId>spring-cloud-gcp-starter-pubsub</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash

compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-starter-pubsub'
```
{% endtab %}
{% endtabs %}

### Configuration

There is no explicit configuration required if you use the automatic authentication and project ID detection. I.e., if you already logged in locally with `gcloud` command line, then it'll automatically use Datastore from the project you configured in `gcloud`.

{% hint style="info" %}
Notice that there is no explicit configuration for username/password. Cloud Pub/Sub authentication uses the GCP credential \(either your user credential, or Service Account credential\), and authorization is configured via Identity Access Management \(IAM\).
{% endhint %}

### Pub/Sub Template

#### JSON Serialization

You need to produce a  `PubSubMessageConverter` bean in order for Spring Cloud GCP Pub/Sub to automatically serialize a POJO into JSON payload, 

```java
@Bean
public PubSubMessageConverter pubSubMessageConverter() {
    return new JacksonPubSubMessageConverter(new ObjectMapper());
}
```

#### Publish a Message

You can use `PubSubPublisherTemplate` to easily publish a message.

```java
@RestController
class OrderController {
  private final PubSubPublisherTemplate publisherTemplate;

  OrderController(
      PubSubPublisherTemplate publisherTemplate) {
    this.publisherTemplate = publisherTemplate;
  }

  @PostMapping("/order/submit")
  void submitOrder(@RequestBody Order order) {
    publisherTemplate.publish("orders", order);
  }
}
```

#### Pull a Message

You can pull N number of messages by using `PubSubSubscriberTemplate`.

```java
@Bean
ApplicationRunner runner(PubSubSubscriberTemplate subscriberTemplate) {
  return (args) -> {
    var msgs = subscriberTemplate
        .pullAndConvert("orders-subscription", 1, true, Order.class);
    msgs.forEach(msg -> {
      logger.info(m.getPayload().getId());
      msg.ack();
    });
  };
}
```

#### Subscribe to a Subscription

You can also just subscribe to a subscription using Streaming Pull, so that it maintains a persistent connection, and can process messages whenever they arrive:

```java
@Bean
ApplicationRunner subscribeRunner(PubSubSubscriberTemplate subscriberTemplate) {
  return (args) -> {
    subscriberTemplate.subscribeAndConvert("orders-subscription", msg -> {
      System.out.println(msg.getPayload().getId());
      msg.ack();
    }, Order.class);
  };
}
```

{% hint style="warning" %}
Streaming pull currently does not support back-pressure  well. If you have many small messages, but each message takes a long time to process.
{% endhint %}

### Reactive Stream

If you are using Project Reactor \(or Webflux that uses Project Reactor\), you can also subscribe to a Pub/Sub Subscription using `PubSubReactiveFactory`.

```java
@Bean
ApplicationRunner reactiveSubscriber(PubSubReactiveFactory reactiveFactory, PubSubMessageConverter converter) {
  return (args) -> {
    reactiveFactory.poll("orders-subscription", 250L)
      .map(msg -> converter.fromPubSubMessage(msg.getPubsubMessage(), Order.class))
      .doOnNext(order -> System.out.println(order.getId()))
      .subscribe();
  };
}
```

### Spring Integration

If you use Spring Integration, you can easily use Pub/Sub to send and consume messages, using an `InboundChannelAdapter` and `MessageHandler`.

#### Inbound Channel

In Spring Integration, you can configure bind an input channel to a Pub/Sub Subscription using the `PubSubInboundChannelAdapter`.

```java
@Bean
public MessageChannel orderRequestInputChannel() {
  return MessageChannels.direct().get();
}

@Bean
public PubSubInboundChannelAdapter orderRequestChannelAdapter(
    @Qualifier("orderRequestInputChannel") MessageChannel inputChannel,
    PubSubTemplate pubSubTemplate) {
  PubSubInboundChannelAdapter adapter =
      new PubSubInboundChannelAdapter(
          pubSubTemplate, "orders-subscription");
  adapter.setOutputChannel(inputChannel);
  adapter.setPayloadType(Order.class);
  adapter.setAckMode(AckMode.AUTO);

  return adapter;
}
```

#### Message Handler and Message Gateway

To send the message to a topic, you can use `PubSubMessageHandler` to bind it to a channel.

```java
@Bean
@ServiceActivator(inputChannel = "ordersRequestOutputChannel")
public MessageHandler ordersOutputMessageHandler() {
  return new PubSubMessageHandler(pubSubTemplate, "orders");
}
```

With Spring Integration Message Gateway, you can also bind a gateway method to a channel that's handled by the `PubSubMessageHandler`.

```java
@MessagingGateway
public interface OrdersGateway {
  @Gateway(requestChannel = "ordersRequestOutputChannel")
  void sendOrder(Order order);
}
```

### Spring Cloud Stream

