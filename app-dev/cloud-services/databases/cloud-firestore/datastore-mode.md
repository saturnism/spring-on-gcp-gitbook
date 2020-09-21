# Datastore Mode

## Cloud Firestore Datastore Instance

There can only be one Cloud Firestore instance associated with a single project. The Datastore instance is automatically created when you enable the API:

There can only be one Datastore instance associated with a single project. The Cloud Firestore in Datastore instance is automatically created when you enable the API:

### Enable API

```bash
gcloud services enable datastore.googleapis.com
```

### Data Schema

Because Cloud Firestore is a NoSQL database, you do not need to explicitly create tables, define data schema, etc. Simply use the API to store new documents, and perform CRUD operations.

## Spring Data Datastore

The easiest way to access Datastore is using Spring Cloud GCP's [Spring Data Datastore starter](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/current/reference/html/#spring-data-cloud-datastore). This starter provides full Spring Data support for Datastore while implementing idiomatic access patterns.

| Spring Data Feature | Supported |
| :--- | :--- |
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

Add the Spring Data Datastore starter:

{% tabs %}
{% tab title="Maven" %}
```markup
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-data-datastore</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```groovy
compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-starter-data-datastore'
```
{% endtab %}
{% endtabs %}

### Configuration

There is no explicit configuration required if you use the automatic authentication and project ID detection. I.e., if you already logged in locally with `gcloud` command line, then it'll automatically use Datastore from the project you configured in `gcloud`.

{% hint style="info" %}
Notice that there is no explicit configuration for username/password. Cloud Firestore authentication uses the GCP credential \(either your user credential, or Service Account credential\), and authorization is configured via Identity Access Management \(IAM\).
{% endhint %}

### ORM

Spring Data Cloud Datastore allows you to map domain POJOs to Datastore documents via annotations. Read the [Spring Data Datastore reference documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/current/reference/html/#object-mapping-2) for details

```java
@Entity
class Order {
    @Id
    private Long id;
    private String description;
    private LocalDateTime timestamp;
    private List<OrderItem> items;

    // Getters and setters ...
}

@Entity
class OrderItem {
    private String description;
    private Long quantity;

    // Getters and setters ...
}
```

Because Datastore is a document-oriented NoSQL database, you can have nested structure, you can establish parent-children relationships without complicated foreign keys.

{% hint style="info" %}
Read the [Spring Data Datastore reference documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/current/reference/html/#object-mapping-2) for more details.
{% endhint %}

### Repository

Use Spring Data repository to quickly get CRUD access to the Datastore.

```java
@Repository
interface OrderRepository extends DatastoreRepository<Order, Long> {
}
```

In a business logic service you can utilize the repositories:

```java
@Service
class OrderService {
    private final OrderRepository orderRepository;

    OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Transactional
    Order createOrder(Order order) {
        // Set the creation time
        order.setTimestamp(LocalDateTime.now());

        // Children are saved in cascade.
        return orderRepository.save(order);
    }
}
```

### Rest Repository

[Spring Data Rest](https://spring.io/projects/spring-data-rest) can expose a Spring Data repository directly on a RESTful endpoint, and rendering the payload as JSON with [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) format. It supports common access patterns like pagination.

Add Spring Data Rest starter:

{% tabs %}
{% tab title="Maven" %}
```markup
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```groovy
compile group: 'org.springframework.boot', name: 'spring-boot-starter-data-rest'
```
{% endtab %}
{% endtabs %}

```java
@RepositoryRestResource
interface OrderRepository extends DatastoreRepository<Order, String> {
}
```

To access the endpoint for Order:

```java
curl http://localhost:8080/orders
```

### Samples

* [Spring Boot with Datastore sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-samples/spring-cloud-gcp-data-datastore-sample)

