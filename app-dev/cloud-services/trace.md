# Trace

## Cloud Trace

Cloud Trace is a managed distributed tracing system that collects latency data from your applications and displays it in the Google Cloud Console. You can track how requests propagate through your application and receive detailed near real-time performance insights.

### Enable API

```bash
gcloud services enable cloudtrace.googleapis.com
```

## Spring Cloud Sleuth

[Spring Cloud GCP's Trace integration](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/1.2.3.RELEASE/reference/html/#stackdriver-trace) uses [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) behind the scenes to instrument and trace your application. In addition, it'll also enhance the log messages to include the current trace context \(Trace ID, Span ID\) for trace to log correlation.

### Dependency

Add the Spring Cloud GCP Trace starter:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-trace</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash

compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-starter-trace'
```
{% endtab %}
{% endtabs %}

### Configuration

By default, Spring Cloud Sleuth samples only 10% of the requests.  I.e., 1 in 10 requests may have traces propagated to the trace server \(Cloud Trace\). In a non-production environment, you may want to see all of the trace. You can adjust the sampling rate using Spring Cloud Sleuth's properties:

{% code title="application.properties" %}
```text
# Set sampler probability to 100%
spring.sleuth.sampler.probability=1.0
```
{% endcode %}

{% hint style="info" %}
Notice that there is no explicit configuration for username/password. Cloud Trace authentication uses the GCP credential \(either your user credential, or Service Account credential\), and authorization is configured via Identity Access Management \(IAM\).
{% endhint %}

### Instrumentation

Spring Cloud Sleuth automatically adds trace instrumentation to commonly used components, such as [incoming HTTP requests](https://docs.spring.io/spring-cloud-sleuth/docs/2.2.x-SNAPSHOT/reference/html/#http-integration), and incoming [messages from Spring Integration](https://docs.spring.io/spring-cloud-sleuth/docs/2.2.x-SNAPSHOT/reference/html/#messaging-2). See [Spring Cloud Sleuth Integrations documentation](https://docs.spring.io/spring-cloud-sleuth/docs/2.2.x-SNAPSHOT/reference/html/#integrations) for more details.

#### Web

Spring Cloud Sleuth will automatically trace incoming requests from WebMVC, or WebFlux as-is.

```java
@RestController
class OrderController {
  private final OrderRepository orderRepository;
  
  OrderController(OrderRepository orderService) {
    this.orderRepository = orderRepository;
  }
  
  @GetMapping("/order/{orderId}")
  public Order getOrder(@PathParam String orderId) {
    return orderRepository.findById(orderId);
  }
}
```

{% hint style="info" %}
In this example, an incoming request to `/order/{orderId}` endpoint will be automatically traced, and the traces will be propagated to Cloud Trace based on the sampler probability.
{% endhint %}

#### Messaging

Spring Cloud Sleuth will automatically trace incoming messages and handlers when using Spring Integration

#### Custom Spans

If there is a piece of code/method that you want to break out into it's own span, you can use Spring Cloud Sleuth's `@NewSpan` annotation. See [Spring Cloud Sleuth's Creating New Span documentation](https://docs.spring.io/spring-cloud-sleuth/docs/2.2.x-SNAPSHOT/reference/html/#creating-new-spans).



```java
@Service
class OrderService {
	private final OrderRepository orderRepository;

	...

  @NewSpan
	@Transactional
	Order createOrder(Order order) {
	  ...
		return orderRepository.save(order);
	}
}
```

#### Tagging Spans

You can associate additional data to a Span \(a tag\) via annotation. See [Spring Cloud Sleuth's Continuing Span documentation](https://docs.spring.io/spring-cloud-sleuth/docs/2.2.x-SNAPSHOT/reference/html/#continuing-spans-2).



```java
@Service
class OrderService {
	private final OrderRepository orderRepository;

  ...

  @NewSpan
	@Transactional
	Order getOrder(@SpanTag("orderId") String id) {
		return orderRepository.findById(id);
	}
}
```

### Propagation

Spring Cloud Sleuth automatically propagates the trace context to a remote system \(e.g., via HTTP request, or messaging\) when using `RestTemplate`, `WebClient`, Spring Integration, and more. See [Spring Cloud Sleuth Integrations documentation](https://docs.spring.io/spring-cloud-sleuth/docs/2.2.x-SNAPSHOT/reference/html/#integrations) for more details.

#### Rest Template / WebClient

Simply create a `RestTemplate` or `WebClient` bean and Spring Cloud Sleuth will automatically add filters to propagate the trace context via HTTP headers.

```java
@Bean
RestTemplate restTemplate() {
  return new RestTemplate();
}
```

#### Messaging

When using Spring Integration, Spring Cloud Sleuth will automatically propagate trace context via message headers. For example, send a [Pub/Sub message with Spring Integration's Gateway](messaging.md#spring-integration) will automatically add trace headers to the Pub/Sub message.

#### Additional Headers

Spring Cloud Sleuth uses [OpenZipkin's Brave tracer](https://github.com/openzipkin/brave), and uses [B3 propagation](https://github.com/openzipkin/b3-propagation). Over HTTP, it will automatically propagate B3 headers to HTTP headers.

When running your application in Istio, you may need to propagate [additional trace headers required by Istio](https://istio.io/latest/faq/distributed-tracing/#how-to-support-tracing), such as `x-request-id` and `x-ot-span-context`.

```text
spring.sleuth.propagation-keys=x-request-id,x-ot-span-context
```

### Samples

* [Spring Cloud GCP Trace sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-samples/spring-cloud-gcp-trace-sample)

## Istio

If you use Istio service mesh, Istio can automatically capture service to service traces. You can use Spring Cloud Sleuth to [propagate additional trace headers](trace.md#additional-headers), without any trace senders:

```text
spring.sleuth.propagation-keys=x-request-id,x-ot-span-context
```

For in-application trace, you can use [Spring Cloud GCP Trace starter](trace.md#spring-cloud-sleuth).

