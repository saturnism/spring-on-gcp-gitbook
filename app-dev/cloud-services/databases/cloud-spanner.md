# Cloud Spanner

[Cloud Spanner](https://cloud.google.com/spanner) is a scalable, enterprise-grade, globally-distributed, and strongly consistent database service built for the cloud specifically to combine the benefits of relational database structure with non-relational horizontal scale. It delivers high-performance transactions and strong consistency across rows, regions, and continents with an industry-leading 99.999% availability SLA, no planned downtime, and enterprise-grade security. Cloud Spanner revolutionizes database administration and management and makes application development more efficient.

## Cloud Spanner Instance

### Enable API

```bash
gcloud services enable spanner.googleapis.com
```

### Create an Instance

```bash
gcloud spanner instances create spanner-instance \
  --config=regional-us-central1 \
  --nodes=1 --description="A Spanner Instance"
```

{% hint style="info" %}
This example creates a new regional instance \(i.e., spanning across zones within the same region\). Cloud Spanner supports multi-regional configuration to span across multiple regions for highest availability.
{% endhint %}

### Create a Database

A Cloud Spanner instance can host multiple databases, that each contains its own tables.

```bash
gcloud spanner databases create orders \
  --instance=spanner-instance
```

### Connect to Instance

There is no interactive CLI to Cloud Spanner. You can create table and execute SQL statements from the Cloud Console, or from `gcloud` command line. Following are some common operations:

#### List Databases

```bash
gcloud spanner databases list --instance=spanner-instance
```

#### Execute SQL Statements

```bash
gcloud spanner databases execute-sql orders \
  --instance=spanner-instance \
  --sql="SELECT 1"
```

#### Show Query Plan

```bash
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
  --ddl="$(<schema.ddl)"
```

## Spring Data Spanner Starter

The easiest way to access Cloud Spanner is using Spring Cloud GCP's [Spring Data Spanner starter](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/current/reference/html/#spring-data-cloud-spanner). This starter provides full Spring Data support for Cloud Spanner while implementing idiomatic access patterns.

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

Add the Spring Data Spanner starter:

{% tabs %}
{% tab title="Maven" %}
```markup
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-data-spanner</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```groovy
compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-starter-data-spanner'
```
{% endtab %}
{% endtabs %}

### Configuration

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

Spring Data Cloud Spanner allows you to map domain POJOs to Cloud Spanner tables via annotations. Read the [Spring Data Spanner reference documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/current/reference/html/#object-mapping) for details

{% code title="Order.java" %}
```java
import java.time.LocalDateTime;
import java.util.List;
import lombok.Data;
import org.springframework.cloud.gcp.data.spanner.core.mapping.*;

@Table(name="orders")
@Data // Lombok to generate getter/setters
class Order {
  @PrimaryKey
  @Column(name="order_id")
  private String id;

  private String description;

  @Column(name="creation_timestamp")
  private LocalDateTime timestamp;

  @Interleaved
  private List<OrderItem> items;
}  
```
{% endcode %}

{% code title="OrderItem.java" %}
```java
import lombok.Data;
import org.springframework.cloud.gcp.data.spanner.core.mapping.*;

@Table(name="order_items")
@Data // Lombok to generate getter/setters
class OrderItem {
    @PrimaryKey(keyOrder = 1)
    @Column(name="order_id")
    private String orderId;

    @PrimaryKey(keyOrder = 2)
    @Column(name="order_item_id")
    private String orderItemId;

    private String description;
    private Long quantity;
}
```
{% endcode %}

{% hint style="info" %}
Read the [Spring Data Spanner reference documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/current/reference/html/#object-mapping) for more details.
{% endhint %}

### Repository

Use Spring Data repository to quickly get CRUD access to the Cloud Spanner tables.

{% code title="OrderRepository.java" %}
```java
package com.example.demo;

import org.springframework.cloud.gcp.data.spanner.repository.SpannerRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface OrderRepository extends SpannerRepository<Order, String> {
}
```
{% endcode %}

{% code title="OrderItemRepoistory.java" %}
```java
package com.example.demo;

import lombok.Data;
import org.springframework.cloud.gcp.data.spanner.core.mapping.*;

@Table(name="order_items")
@Data
class OrderItem {
  @PrimaryKey(keyOrder = 1)
  @Column(name="order_id")
  private String orderId;

  @PrimaryKey(keyOrder = 2)
  @Column(name="order_item_id")
  private String orderItemId;

  private String description;
  private Long quantity;
}
```
{% endcode %}

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

[Cloud Spanner JDBC Driver](https://cloud.google.com/spanner/docs/use-oss-jdbc) can be used if you need raw JDBC access.

### Dependency

Add Cloud Spanner JDBC Driver:

{% tabs %}
{% tab title="Maven" %}
```markup
<dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-spanner-jdbc</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```groovy
compile group: 'com.google.cloud', name: 'google-cloud-spanner-jdbc'
```
{% endtab %}
{% endtabs %}

Use Spring Boot JDBC Starter to use JDBC Template:

{% tabs %}
{% tab title="Maven" %}
```markup
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```groovy
compile group: 'org.springframework.boot', name: 'spring-boot-starter-jdbc'
```
{% endtab %}
{% endtabs %}

### Configuration

Cloud Spanner JDBC Driver Class:

```java
com.google.cloud.spanner.jdbc.JdbcDriver
```

Cloud Spanner JDBC URL format:

```java
jdbc:cloudspanner:/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_NAME
```

With Spring Data JDBC, you can configure the datasource:

{% code title="application.properties" %}
```java
spring.datasource.driver-class-name=com.google.cloud.spanner.jdbc.JdbcDriver
spring.datasource.url=jdbc:cloudspanner:/projects/PROJECT_ID/instances/spanner-instance/databases/orders
```
{% endcode %}

## Hibernate

[Cloud Spanner Hibernate Dialect](https://cloud.google.com/spanner/docs/use-hibernate) lets you use Cloud Spanner with [Hibernate ORM](https://hibernate.org/orm/). You can use Cloud Spanner with Hibernate from any Java application. When using Spring Boot, you can use the Hibernate Dialect with [Spring Data JPA](https://spring.io/projects/spring-data-jpa).

### Dependency

Add both the Cloud Spanner JDBC driver and Cloud Spanner Hibernate Dialect:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
  <groupId>com.google.cloud</groupId>
  <artifactId>google-cloud-spanner-jdbc</artifactId>
</dependency>
<dependency>
  <groupId>com.google.cloud</groupId>
  <artifactId>google-cloud-spanner-hibernate-dialect</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```text
compile group: 'com.google.cloud', name: 'google-cloud-spanner-jdbc'
compile group: 'com.google.cloud', name: 'google-cloud-spanner-hibernate-dialect'
```
{% endtab %}
{% endtabs %}

Add Spring Data JPA starter:

{% tabs %}
{% tab title="Maven" %}
```markup
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```groovy
compile group: 'org.springframework.boot', name: 'spring-boot-starter-data-jpa'
```
{% endtab %}
{% endtabs %}

### Configuration

Cloud Spanner Hibernate Dialect class:

```java
com.google.cloud.spanner.hibernate.SpannerDialect
```

Configure Spring Data JPA:

{% code title="application.properties" %}
```java
spring.datasource.driver-class-name=com.google.cloud.spanner.jdbc.JdbcDriver
spring.datasource.url=jdbc:cloudspanner:/projects/PROJECT_ID/instances/spanner-instance/databases/orders

spring.jpa.database-platform=com.google.cloud.spanner.hibernate.SpannerDialect
```
{% endcode %}

### ORM

Use JPA annotations to map POJO to Cloud Spanner database tables.

{% hint style="info" %}
In Cloud Spanner, parent-children relationship is modeled as a composite key. With JPA, use `IdClass` or `EmbeddableId` to map a composite key.
{% endhint %}

```java
@Entity
@Table(name="orders")
class Order {
    @Id
    @Column(name="order_id")
  private String id;

    private String description;

    @Column(name="creation_timestamp")
    private LocalDateTime creationTimestamp;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items;

  // Getter and Setter ...
}

@Embeddable
class OrderItemId implements Serializable {
  @Column(name="order_id")
    private String orderId;

    @Column(name="order_item_id")
    private String orderItemId;

    // Getter and Setter ...
    // Hashcode and Equals ...
}

@Entity
@Table(name="order_items")
class OrderItem {
    @EmbeddedId
    private OrderItemId orderItemId;

    private String description;
    private Long quantity;

    @ManyToOne
  @JoinColumn(name="order_id", insertable = false, updatable = false)
    private Order order;

    // Getter and Setter ...
}
```

### Repository

Use Spring Data repository for CRUD access to the tables.

```java
@Repository
interface OrderRepository extends JpaRepository<Order, String> {
}

@Repository
interface OrderItemRepository extends JpaRepository<OrderItem, OrderItemId> {
}
```

### Rest Repository

Use Spring Data Rest to expose the repositories as RESTful services with HATEOS format.

```java
@RepositoryRestResource
interface OrderRepository extends JpaRepository<Order, String> {
}

@RepositoryRestResource
interface OrderItemRepository extends JpaRepository<OrderItem, OrderItemId> {
}
```

### Sample

* [Spring Boot with Spring Data JPA and Cloud Spanner](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/tree/master/google-cloud-spanner-hibernate-samples/spring-data-jpa-sample)
* [Quarkus with Hibernate and Cloud Spanner](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/tree/master/google-cloud-spanner-hibernate-samples/quarkus-jpa-sample)
* [Microprofile with Hibernate and Cloud Spanner](https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate/tree/master/google-cloud-spanner-hibernate-samples/microprofile-jpa-sample)

## R2DBC

[Cloud Spanner R2DBC driver](https://github.com/GoogleCloudPlatform/cloud-spanner-r2dbc) let's you access Cloud Spanner data with reactive API.

[Cloud Spanner R2DBC driver](https://github.com/GoogleCloudPlatform/cloud-spanner-r2dbc) is under active development.

It can be used with [Spring Data R2DBC](https://spring.io/projects/spring-data-r2dbc) using the [Cloud Spanner R2DBC Dialect](https://github.com/GoogleCloudPlatform/cloud-spanner-r2dbc/tree/master/cloud-spanner-spring-data-r2dbc).

### Samples

* [Spring Boot with Spring Data R2DBC and Cloud Spanner](https://github.com/GoogleCloudPlatform/cloud-spanner-r2dbc/tree/master/cloud-spanner-r2dbc-samples/cloud-spanner-spring-data-r2dbc-sample)

