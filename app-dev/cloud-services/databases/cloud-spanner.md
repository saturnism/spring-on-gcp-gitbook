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

### ORM

### Repository

### Rest Repository

### Samples



## JDBC

## Hibernate

## R2DBC

