# Cloud Pub/Sub

## Cloud Pub/Sub

[Cloud Pub/Sub](https://cloud.google.com/pubsub/docs/) is a managed publish/subscribe service, where you can send messages to a topic, and subscribe via push, pull, or streaming pull. A single Cloud Pub/Sub Topic can be associated with one or more Subscriptions. Each Subscription can have one or more subscribers. Cloud Pub/Sub delivers messages with guaranteed at-least-once delivery, and there is no ordering guarantee.

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

### Dead Letter Topic

Normally if you failed to process a message, then you will need to un-acknowledge, and the message will be re-delivered again. However, you may not want to continuously re-deliver the same message indefinitely because your application simply cannot process it. In this case, you'd want to create a subscription with a Dead Letter Topic. You can then configure the max re-delivery attempts - and when all the attempts are exhausted, Pub/Sub will then re-deliver the message to a different topic. In order for the Dead Letter Topic to persist the message, you must also create a subscription for it - otherwise, no message will be persisted for the Dead Letter Topic.

#### Create a Dead Letter Topic

```bash
gcloud pubsub topics create order-dlt
gcloud pubsub subscriptions create order-dlt-subscription \
  --topic=order-dlt \
  --ack-deadline=300 \
  --expiration-period=never
```

#### Create a Subscription with Dead Letter Topic

```bash
gcloud pubsub subscriptions create orders-subscription \
  --topic=orders \
  --dead-letter-topic=order-dlt \
  --max-delivery-attempts=5
```

#### Grant Cloud Pub/Sub Permissions

You need to grant Cloud Pub/Sub additional permissions in order for Cloud Pub/Sub to be able to remove message from the original subscription and then publishing it to the DLT.

```bash
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
PUBSUB_SERVICE_ACCOUNT="service-${PROJECT_NUMBER}@gcp-sa-pubsub.iam.gserviceaccount.com"

# Permission to subscribe to the original subscription
gcloud pubsub subscriptions add-iam-policy-binding orders-subscription \
    --member="serviceAccount:$PUBSUB_SERVICE_ACCOUNT"\
    --role="roles/pubsub.subscriber"
    
# Permission to re-publish the message to the Dead Letter Topic
gcloud pubsub topics add-iam-policy-binding orders-dlt \
    --member="serviceAccount:${PUBSUB_SERVICE_ACCOUNT}"\
    --role="roles/pubsub.publisher"
```

{% hint style="info" %}
See [Cloud Pub/Sub Dead Letter Topic](https://cloud.google.com/pubsub/docs/dead-letter-topics) for more information.
{% endhint %}

### Ordering

You can enable Message Ordering to a Pub/Sub topic, so that messages with the same key value \(e.g., the same Order ID\) can be delivered in order. Ordering can be important for a CQRS system that cannot process events out of order.

{% hint style="info" %}
See [Cloud Pub/Sub Ordering documentation](https://cloud.google.com/pubsub/docs/ordering) for more information.
{% endhint %}

### Filtering

Some architectures may be delivering many different types of messages to a single topic. For example, there may be different event types for an Order event \(e.g., Created, Fulfilled, Returned ...\). If you need to create different workers to process different type of events, then you can create a Subscription for each event type, and select the type of message you want to process with a Filter.

{% hint style="info" %}
See [Cloud Pub/Sub Filtering documentation](https://cloud.google.com/pubsub/docs/filtering) for more information.
{% endhint %}

## Spring Cloud Pub/Sub

The easiest way to use Cloud Pub/Sub is using Spring Cloud GCP's [Spring Pub/Sub starter](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/current/reference/html/#google-cloud-pubsub). This starter provides easy to use `PubSubTemplate` bean to send and receive messages.

### Dependency

Add the Spring Cloud GCP Pub/Sub starter:

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

There is no explicit configuration required if you use the automatic authentication and project ID detection. I.e., if you already logged in locally with `gcloud` command line, then it'll automatically use Pub/Sub topics/subscriptions from the project you configured in `gcloud`.

{% hint style="info" %}
Notice that there is no explicit configuration for username/password. Cloud Pub/Sub authentication uses the GCP credential \(either your user credential, or Service Account credential\), and authorization is configured via Identity Access Management \(IAM\).
{% endhint %}

### Pub/Sub Template

#### JSON Serialization

You need to produce a `PubSubMessageConverter` bean in order for Spring Cloud GCP Pub/Sub to automatically serialize a POJO into JSON payload,

```java
@Bean
public PubSubMessageConverter pubSubMessageConverter() {
    return new JacksonPubSubMessageConverter(new ObjectMapper());
}
```

#### Non-Web Applications

In a web application, the Java process will stay alive until it's explicitly killed. If your Pub/Sub message subscriber does not use a Web starter \(Web or Webflux\), then the application may exit as soon as it initializes. When you need the Pub/Sub subscribers to stay alive without exiting immediately, you must create a bean `ThreadPoolTaskScheduler` named `pubsubSubscriberThreadPool`.

```java
@Bean
ThreadPoolTaskScheduler pubsubSubscriberThreadPool() {
  return new ThreadPoolTaskScheduler();
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
      logger.info(msg.getPayload().getId());
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
Streaming pull currently does not support back-pressure well. If you have many small messages, but each message takes a long time to process, then you may not want to use Streaming Pull.
{% endhint %}

### Reactive Stream

If you are using Project Reactor \(or Webflux that uses Project Reactor\), you can also subscribe to a Pub/Sub Subscription using `PubSubReactiveFactory`.

```java
@Bean
ApplicationRunner reactiveSubscriber(PubSubReactiveFactory reactiveFactory, PubSubMessageConverter converter) {
  return (args) -> {
    reactiveFactory.poll("orders-subscription", 250L)
      // Convert a JSON payload into an object
      .map(msg -> converter.fromPubSubMessage(msg.getPubsubMessage(), Order.class))
      .doOnNext(order -> System.out.println(order.getId()))
      // Mannually acknowledge the message
      .doOnNext(AcknowledgeablePubsubMessage::ack);
      .subscribe();
  };
}
```

### Samples

* [Spring Cloud GCP Pub/Sub Template sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-samples/spring-cloud-gcp-pubsub-sample)
* [Spring Cloud GCP Reactive Pub/Sub sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-samples/spring-cloud-gcp-pubsub-reactive-sample)

## Spring Integration

[Spring Integration](https://spring.io/projects/spring-integration) is allows you to easily create Enterprise Integration pipelines by supporting well known [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/). If you use Spring Integration, you can easily use Pub/Sub to send and consume messages, using an `InboundChannelAdapter` and `MessageHandler`.

### Inbound Channel

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

You can then create a new message processor and binding a method to the input channel, by using the `ServiceActivator` annotation.

```java
public class OrderProcessor {
  private static final Logger logger = LoggerFactory.getLogger(OrderProcessor.class);

  @ServiceActivator(inputChannel = "orderRequestInputChannel")
  void process(@Payload Order order) {
    logger.info(order.getId());
  }
}
```

### Message Handler and Message Gateway

To send the message to a topic, you can use `PubSubMessageHandler` to bind it to a channel by using the `ServiceActivator` annotation.

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

Now you can send a message to the Pub/Sub Topic by using an instance of the Gateway.

```java
@Bean
ApplicationRunner sendOrder(OrderGateway gateway) {
  return (args) -> {
    Order order = new Order();
    order.setId(UUID.randomUUID().toString());
    gateway.sendOrder(order);
  };
}
```

### Integration Flow

Last but not least, you can create an entire message flow, with patterns such as Retry and Rate Limiting, and processing the message by creating a new [Integration Flow](https://docs.spring.io/spring-integration/reference/html/dsl.html).

### Samples

* [Spring Integration with Pub/Sub sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-samples/spring-cloud-gcp-integration-pubsub-sample)
* [Spring Integration with Pub/Sub and JSON payload sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-samples/spring-cloud-gcp-integration-pubsub-json-sample)

## Spring Cloud Stream

Spring Cloud Stream allows you to write event-driven microservices by simply implementing well known Java functional interfaces such as `Function`, `Consumer`, and `Supplier`. Messaging infrastructure \(such as a Pub/Sub Topic or Subscription\) can be bound to these functions at the runtime.

### Dependency

Spring Cloud Stream depends on Spring Integration. In addition, add the Pub/Sub Stream Binder.

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-stream</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-gcp-pubsub-stream-binder</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash
compile group: 'org.springframework.cloud', name: 'spring-cloud-stream'
compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-pubsub-stream-binder'
```
{% endtab %}
{% endtabs %}

### Consume Messages

#### Consumer

A Spring Cloud Stream consumer to consume messages is simply a Java `Consumer`.

```java
@Bean
public Consumer<Order> processOrder() {
  return order -> {
    logger.info(order.getId());
    };
};
```

#### Binding

You can bind the `processOrder` consumer with `applications.properties` configuration. See [Spring Cloud Streams documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/current/reference/html/spring-cloud-stream.html#_binding_and_binding_names) on the binding naming convention, where `processOrder` becomes `processorOrder-in-0`.

```text
spring.cloud.stream.bindings.processOrder-in-0.destination=orders
spring.cloud.stream.bindings.processOrder-in-0.group=orders-processor-group

# For development use, but not recommended for production.
spring.cloud.stream.gcp.pubsub.default.consumer.auto-create-resources=true
```

A Spring Cloud Streams Consumer Group is mapped to a subscription, with the naming convention of `[destination-name].[consumer-group-name]`. So in this example, a subscription named `orders.orders-processor-group` will be automatically created.

### Consume and Produce Message

If your consumer need to also produce a message to another topic, you can implement a `Function`.

#### Function

```java
@Bean
public Function<Order, String> processOrder() {
    return order -> {
      logger.info(order.getId());
      return order.getId();
    };
};
```

#### Binding

The output of the function can be forwarded to the next destination/topic.

```text
spring.cloud.stream.bindings.processOrder-in-0.destination=orders
spring.cloud.stream.bindings.processOrder-in-0.group=orders-processor-group
spring.cloud.stream.bindings.processOrder-out-0.destination=order-processed

# For development use, but not recommended for production.
spring.cloud.stream.gcp.pubsub.default.consumer.auto-create-resources=true
```

### Produce Messages

If you need to continuously produce messages, then you can implement `Supplier`. Supplier can be used in two ways, either supply the object itself, or supply a `Flux` that can then continuously emit new messages. Read Spring Cloud Streams documentation for more information.

#### Supplier

```java
@Bean
Supplier<Flux<Order>> ordersToProcess() {
    return () -> Flux.from(e -> {
        while (true) {
            try {
                Order order = new Order();
                order.setId(UUID.randomUUID().toString());
                e.onNext(order);
                Thread.sleep(1000L);
            } catch (InterruptedException interruptedException) {
            }
      }
    });
}
```

#### Binding

The output of the supplier can be sent to the destination/topic.

```text
spring.cloud.stream.bindings.ordersToProcess-out-0.destination=order-processed

# For development use, but not recommended for production.
spring.cloud.stream.gcp.pubsub.default.consumer.auto-create-resources=true
```

This will send the output of the supplier to the `order-processed` topic.

### Samples

* [Spring Cloud GCP Pub/Sub Stream Binder sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-samples/spring-cloud-gcp-pubsub-binder-sample)
* [Spring Cloud GCP Pub/Sub Stream Binder \(no annotations\) sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-samples/spring-cloud-gcp-pubsub-stream-binder-functional-sample)

