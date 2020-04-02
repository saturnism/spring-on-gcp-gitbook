# Cloud Spanner

[Cloud Spanner](https://cloud.google.com/spanner) is a scalable, enterprise-grade, globally-distributed, and strongly consistent database service built for the cloud specifically to combine the benefits of relational database structure with non-relational horizontal scale. It delivers high-performance transactions and strong consistency across rows, regions, and continents with an industry-leading 99.999% availability SLA, no planned downtime, and enterprise-grade security. Cloud Spanner revolutionizes database administration and management and makes application development more efficient.

## Cloud Spanner Instance

### Enable API

```text
gcloud services enable spanner.googleapis.com
```

### Create an Instance

```text
gcloud spanner instances create spanner-instance \
  --config=regional-us-central1 \
  --nodes=1 --description="A Spanner Instance"
```

{% hint style="info" %}
This example creates a new regional instance \(i.e., spanning across zones within the same region\). Cloud Spanner supports multi-regional configuration to span across multiple regions for highest availability.
{% endhint %}

### Create a Database

A Cloud Spanner instance can host multiple databases, that each contains its own tables.

```text
gcloud spanner databases create orders \
  --instance=spanner-instance
```

### Connect to Instance

There is no interactive CLI to Cloud Spanner. You can create table and execute SQL statements from the Cloud Console, or from `gcloud` command line. Following are some common operations:

#### List Databases

```text
gcloud spanner databases list --instance=spanner-instance
```

#### Execute SQL Statements

```text
gcloud spanner databases execute-sql orders \
  --instance=spanner-instance \
  --sql="SELECT 1"
```

#### Show Query Plan

```text
gcloud spanner databases execute-sql orders \
  --instance=spanner-instance \
  --sql="SELECT 1" \
  --query-mode=PLAN
```

### Create a Table

Create a DDL file, `schema.ddl`:

```sql
CREATE TABLE orders (
  order_id STRING(36) NOT NULL,
  description STRING(255),
  creation_timestamp TIMESTAMP,
) PRIMARY KEY (order_id);

CREATE TABLE order_items (
  order_id STRING(36) NOT NULL,
  order_item_id STRING(36) NOT NULL,
  description STRING(255),
  quantity INT64,
) PRIMARY KEY (order_id, order_item_id),
  INTERLEAVE IN PARENT orders ON DELETE CASCADE;
```

{% hint style="warning" %}
Cloud Spanner differs from traditional RDBMS in a several ways:

* No server-side automatic ID generation.
* Avoid monodically increasing IDs - I.e., no auto incremented ID, because it may create hot spots in partitions.
* No foreign key constraints.
{% endhint %}

{% hint style="success" %}
Cloud Spanner, being horizontally scalable and shards data with your keys, prefers the following:

* UUIDv4 as IDs - It's random and can be partitioned easily.
* Parent-Children relationship encoded using a compose primary key.
* Colocate Parent-Children data using Interleave tables - If accessing parent means children data is also likely to be read, interleaving allows children data to be colocated with parent row in the same parition.
{% endhint %}

Use gcloud to execute the DDL:

```bash
gcloud spanner databases ddl update orders \
  --instance=spanner-instance \
  --ddl=$(<schema.ddl)
```

## Spring Data Spanner Starter

The easiest way to access Cloud Spanner is using Spring Cloud GCP's [Spring Data Spanner starter](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/1.2.2.RELEASE/reference/html/#spring-data-cloud-spanner). This starter provides full Spring Data support for Cloud Spanner while implementing idiomatic access patterns.

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

#### Dependency

Add the Spring Data Spanner starter:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-data-spanner</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash
dependencies {
    compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-data-spanner'
}
```
{% endtab %}
{% endtabs %}

#### Configuration

Configure Cloud Spanner instance and database to connect to.

{% code title="application.properties" %}
```bash
spring.cloud.gcp.spanner.instance-id=spanner-instance
spring.cloud.gcp.spanner.database=orders
```
{% endcode %}

{% hint style="info" %}
Notice that there is no explicit configuration for username/password. Cloud Spanner authentication uses the GCP credential \(either your user credential, or Service Account credential\), and authorization is configured via Identity Access Management \(IAM\).
{% endhint %}

### ORM

Spring Data Cloud Spanner allows you to map domain POJOs to Cloud Spanner tables via annotations. Read the [Spring Data Spanner reference documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/1.2.2.RELEASE/reference/html/#object-mapping) for details  

```java
Table(name="orders")
class Order {
	@PrimaryKey
	@Column(name="order_id")
  private String id;

	private String description;

	private LocalDateTime timestamp;

	@Interleaved
	private List<OrderItem> items;

  // Getter and Setters...
}  

@Table(name="order_items")
class OrderItem {
	@PrimaryKey(keyOrder = 1)
	private String orderId;

	@PrimaryKey(keyOrder = 2)
	private String orderItemId;

	private String description;
	private Long quantity;

  // Getter and Setter...
}
```

{% hint style="info" %}
Read the [Spring Data Spanner reference documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/1.2.2.RELEASE/reference/html/#object-mapping) for more details.
{% endhint %}

### Repository

Use Spring Data repository to quickly get CRUD access to the Cloud Spanner tables.

```java
@Repository
interface OrderRepository extends SpannerRepository<Order, String> {
}

@Repository
interface OrderItemRepository extends SpannerRepository<OrderItem, Key> {
  List<OrderItem> findAllByOrderId(String orderId);
}
```

{% hint style="info" %}
`Order` is the parent and has only a primary key. Spring Data repository's ID parameter type can be the type for the single key. `OrderItem`, however, has a composite key. Spring Data repository's ID parameter type must be Cloud Spanner's `Key` type for a composite key.
{% endhint %}

In a business logic service, you can utilize the repositories:

```java
@Service
class OrderService {
	private final OrderRepository orderRepository;

	OrderService(OrderRepository orderRepository,
			OrderItemRepository orderItemRepository) {
		this.orderRepository = orderRepository;
	}

	@Transactional
	Order createOrder(Order order) {
	  // Use UUID String representation for the ID
		order.setId(UUID.randomUUID().toString());
		
		// Set the creation time
		order.setCreationTimestamp(LocalDateTime.now());

    // Set the parent Order ID and children ID for each item.
		if (order.getItems() != null) {
			order.getItems().stream().forEach(orderItem -> {
				orderItem.setOrderId(order.getId());
				orderItem.setOrderItemId(UUID.randomUUID().toString());
			});
		}
		
		// Children are saved in cascade.
		return orderRepository.save(order);
	}
}
```

### Rest Repository

[Spring Data Rest](https://spring.io/projects/spring-data-rest) can expose a Spring Data repository directly on a RESTful endpoint, and rendering the payload as JSON with [HATEOS](https://en.wikipedia.org/wiki/HATEOAS) format. It supports common access patterns like pagination.

```java
@RepositoryRestResource
interface OrderRepository extends SpannerRepository<Order, String> {
}

@RepositoryRestResource
interface OrderItemRepository extends SpannerRepository<OrderItem, Key> {
  List<OrderItem> findAllByOrderId(String orderId);
}
```

To access the endpoint for Order:

```java
curl http://localhost:8080/orders
```

### Samples

* [Spring Boot with Cloud Spanner sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-samples/spring-cloud-gcp-data-spanner-sample)

## JDBC

## Hibernate

## R2DBC

