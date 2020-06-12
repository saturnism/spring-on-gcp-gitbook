# Native Mode

## Cloud Firestore Native Instance

There can only be one Datastore instance associated with a single project. The Cloud Firestore in Datastore instance is automatically created when you enable the API:

### Enable API

```bash
gcloud services enable firestore.googleapis.com
```

### Data Schema

Because Cloud Firestore is a NoSQL database, you do not need to explicitly create tables, define data schema, etc. Simply use the API to store new documents, and perform CRUD operations.

## Spring Data Firestore

The easiest way to access Cloud Firestore is using Spring Cloud GCP's [Spring Data Firestore starter](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/1.2.2.RELEASE/reference/html/#spring-data-reactive-repositories-for-cloud-firestore). This starter provides full Spring Data support for Cloud Firestore while implementing idiomatic access patterns.

| Spring Data Feature | Supported |
| :--- | :--- |
| Reactive Repository | ✅ |
| ORM | ✅ |
| Declarative Transaction | ✅ |
| Repository | ✅ |
| REST Repository | ✅ |
| Query methods | ✅ |
| Query annotation | ✅ |
| Pagination | ✅ |
| Events | ✅ |
| Auditing | ✅ |

### Dependency

Add the Spring Data Firestore starter:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-data-firestore</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash

compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-data-spanner'
```
{% endtab %}
{% endtabs %}

### Configuration

There is no explicit configuration required if you use the automatic authentication and project ID detection. I.e., if you already logged in locally with `gcloud` command line, then it'll automatically use Datastore from the project you configured in `gcloud`.

{% hint style="info" %}
Notice that there is no explicit configuration for username/password. Cloud Firestore authentication uses the GCP credential \(either your user credential, or Service Account credential\), and authorization is configured via Identity Access Management \(IAM\).
{% endhint %}

### ORM

Spring Data Cloud Firestore allows you to map domain POJOs to Datastore documents via annotations.

```java
@Document
class Order {
	@DocumentId
	private String id;
	private String description;
	private LocalDateTime timestamp;
	private List<OrderItem> items;
	
	// Getters and setters ...
}

class OrderItem {
	private String description;
	private Long quantity;
	
	// Getters and setters ...
}
```

Because Firestore is a document-oriented NoSQL database, you can have nested structure, you can establish parent-children relationships without complicated foreign keys.

{% hint style="info" %}
Read the [Spring Data Firestore reference documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/1.2.2.RELEASE/reference/html/#object-mapping-3) for more details.
{% endhint %}

### Repository

Use Spring Data Reactive repository to quickly get CRUD access to the Cloud Firestore.

```java
@Repository
interface OrderRepository extends FirestoreReactiveRepository<Order> {
}
```

In a business logic service, you can utilize the repositories:

```java
@Service
class OrderService {
  private final OrderRepository orderRepository;

  OrderService(OrderRepository orderRepository) {
    this.orderRepository = orderRepository;
  }

  @Transactional
  Mono<Order> createOrder(Order order) {
    // Set the creation time
    order.setTimestamp(Timestamp.of(new Date()));

    // Children are saved in cascade.
    return orderRepository.save(order);
  }
}
```

### Samples

* [Spring Boot with Cloud Firestore sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-data-firestore)

