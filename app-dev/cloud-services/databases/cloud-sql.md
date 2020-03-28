# Cloud SQL

Cloud SQL is managed MySQL, PostgreSQL, and SQL Server.  Cloud SQL automates backups, replication, and failover to ensure your database is reliable, highly available.

Cloud SQL has automatic data encryption at rest and in transit. Private connectivity with Virtual Private Cloud \(VPC\) and user-controlled network access that includes firewall protection. Compliant with SSAE 16, ISO 27001, PCI DSS v3.0, and HIPAA

## Cloud SQL Instance

### Enable API

```bash
gcloud services enable sqladmin.googleapis.com
```

### Create an Instance

{% tabs %}
{% tab title="MySQL" %}
Create a new Cloud SQL - MySQL Instance with 

```bash
gcloud sql instances create mysql-instance \
  --database-version=MYSQL_5_7 \
  --region=us-central1 \
  --cpu=2 \
  --memory=4G \
  --root-password=[CHOOSE A PASSWORD]
```
{% endtab %}

{% tab title="PostgreSQL" %}
Create a new Cloud SQL - PostgreSQL instance.

```bash
gcloud sql instances create postgresql-instance \
  --database-version=POSTGRES_11 \
  --region=us-central1 \
  --cpu=2 \
  --memory=4G \
  --root-password=[CHOOSE A PASSWORD]
```
{% endtab %}

{% tab title="SQL Server" %}
Create a new Cloud SQL - SQL Server instance.

```bash
gcloud beta sql instances create sqlserver-instance \
  --database-version=SQLSERVER_2017_STANDARD \
  --region=us-central1 \
  --cpu=2 \
  --memory=4G \
  --root-password=[CHOOSE A PASSWORD]
```
{% endtab %}
{% endtabs %}

### Create a Database

{% tabs %}
{% tab title="MySQL" %}
Create a new database inside of the MySQL database instance.

```bash
gcloud sql databases create orders --instance=mysql-instance
```
{% endtab %}

{% tab title="PostgreSQL" %}
Create a new database inside of the PostgreSQL database instance.

```bash
gcloud sql databases create orders --instance=postgresql-instance
```
{% endtab %}

{% tab title="SQL Server" %}
Create a new database inside of the SQL Server database instance.

```bash
gcloud sql databases create orders --instance=sqlserver-instance
```
{% endtab %}
{% endtabs %}

### Connect to Database instance

By default, every database instance has a public IP address. However, the instance is not publicly accessible because it's protected by the firewall. 

To easily connect to the database instance from command line:

{% tabs %}
{% tab title="MySQL" %}
{% hint style="warning" %}
You need the [MySQL client](https://dev.mysql.com/doc/mysql-getting-started/en/) installed locally first, so that you can use `mysql` to connect to any MySQL server.
{% endhint %}

Connect to the MySQL instance using `gcloud` CLI.

```bash
gcloud sql connect mysql-instance
```
{% endtab %}

{% tab title="PostgreSQL" %}
{% hint style="warning" %}
You need the [PostgreSQL client](https://www.postgresql.org/download/) installed locally first, so that you can use `psql` to connect to any PostgreSQL server.
{% endhint %}

Connect to the PostgreSQL instance using `gcloud` CLI.

```bash
gcloud sql connect postgresql-instance
```
{% endtab %}

{% tab title="SQL Server" %}
{% hint style="warning" %}
You need the [MS SQL Server client](https://docs.microsoft.com/en-us/sql/tools/mssql-cli?view=sql-server-ver15) installed locally first, so that you can use `mssql-cli` to connect to any PostgreSQL server.
{% endhint %}

Connect to the PostgreSQL instance using `gcloud` CLI.

```bash
gcloud sql connect postgresql-instance
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
You can configure Cloud SQL instances to only have [private IP addresses](https://cloud.google.com/sql/docs/mysql/private-ip), so that it's only accessible from a Virtual Private Cloud network.
{% endhint %}

### Create a Table

From the command line connection, you can use the client to create a table for the corresponding database. For example:

{% tabs %}
{% tab title="MySQL" %}
```sql
# Change to orders database
USE orders;

CREATE TABLE order_items (
  id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  order_id BIGINT,
  description VARCHAR(255),
  quantity INT DEFAULT 1
);

CREATE TABLE orders (
  id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  description VARCHAR(255),
  creation_timestamp TIMESTAMP
);

ALTER TABLE order_items ADD FOREIGN KEY (order_id) REFERENCES orders (id);
```
{% endtab %}

{% tab title="PostgreSQL" %}
```sql
# Change to orders database
\c orders

CREATE TABLE order_items (
  id BIGSERIAL NOT NULL PRIMARY KEY,
  order_id BIGINT,
  description VARCHAR(255),
  quantity INT DEFAULT 1
);

CREATE TABLE orders (
  id BIGSERIAL NOT NULL PRIMARY KEY,
  description VARCHAR(255),
  creation_timestamp TIMESTAMP
);

ALTER TABLE order_items ADD FOREIGN KEY (order_id) REFERENCES orders (id);
```
{% endtab %}

{% tab title="SQL Server" %}
```sql
USE orders;

CREATE TABLE order_items (
  id BIGINT NOT NULL IDENTITY(1,1) PRIMARY KEY,
  order_id BIGINT,
  description VARCHAR(255),
  quantity INT DEFAULT 1
);

CREATE TABLE orders (
  id BIGINT NOT NULL IDENTITY(1,1) PRIMARY KEY,
  description VARCHAR(255),
  creation_timestamp TIMESTAMP
);

ALTER TABLE order_items ADD FOREIGN KEY (order_id) REFERENCES orders (id);
```
{% endtab %}
{% endtabs %}

### Add a User

You can add a user using `gcloud` command line:

{% tabs %}
{% tab title="MySQL" %}
Use `gcloud` command line to create a new user:

```bash
gcloud sql users create order-user
  --instance=mysql-instance \
  --password=... 
```

{% hint style="danger" %}
The new user has no privileges. Connect to the database server and grant privileges. Refer to [MySQL documentation to use `GRANT`](https://dev.mysql.com/doc/refman/8.0/en/grant.html).
{% endhint %}
{% endtab %}

{% tab title="PostgreSQL" %}
Use `gcloud` command line to create a new user:

```bash
gcloud sql users create order-user \
  --instance=postgresql-instance \
  --password=...
```

{% hint style="danger" %}
The new user has no privileges. Connect to the database server and grant privileges. Refer to [PostgreSQL documentation to use `GRANT`](https://www.postgresql.org/docs/9.0/sql-grant.html).
{% endhint %}
{% endtab %}

{% tab title="SQL Server" %}
Use `gcloud` command line to create a new user:

```bash
gcloud sql users create order-user \
  --instance=sqlserver-instance \
  --password=...
```

{% hint style="danger" %}
The new user has no privileges. Connect to the database server and grant privileges. Refer to [SQL Server documentation to use `GRANT`](https://docs.microsoft.com/en-us/sql/t-sql/statements/grant-object-permissions-transact-sql?view=sql-server-ver15).
{% endhint %}
{% endtab %}
{% endtabs %}

### Instance Connection Name

Every Cloud SQL Instance has a unique instance connection name for the form of `PROJECT_ID:REGION:INSTANCE_NAME`.

Find the Instance Connection Name using `gcloud` command line:

```bash
gcloud sql instances describe INSTANCE_NAME --format='value(connectionName)'
```

{% tabs %}
{% tab title="MySQL" %}
MySQL instance's Instance Connection Name

```bash
gcloud sql instances describe mysql-instance \
  --format='value(connectionName)'
```
{% endtab %}

{% tab title="PostgreSQL" %}
PostgreSQL instance's Instance Connection Name

```bash
gcloud sql instances describe postgresql-instance \
  --format='value(connectionName)'
```
{% endtab %}

{% tab title="SQL Server" %}
SQL Server instance's Instance Connection Name

```bash
gcloud sql instances describe sqlserver-instance \
  --format='value(connectionName)'
```
{% endtab %}
{% endtabs %}

## JDBC

### Cloud SQL Starter

When using Spring Boot, you can use [Spring Cloud GCP's Cloud SQL starter](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/1.2.2.RELEASE/reference/html/#spring-jdbc).

Cloud SQL starter will automatically:

* Add dependency to the corresponding JDBC driver, and the [Cloud SQL socket factory](cloud-sql.md#cloud-sql-socket-factory). You **do not** need to add those dependency separately.
* Configure the JDBC URL for the corresponding database instance.

Add the Cloud SQL Starter dependency:

{% tabs %}
{% tab title="MySQL" %}
Maven:

```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-sql-mysql</artifactId>
</dependency>
```

Gradle:

```bash
compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-starter-sql-mysql'
```
{% endtab %}

{% tab title="PostgreSQL" %}
Maven:

```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-sql-postgresql</artifactId>
</dependency>
```

Gradle:

```bash
compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-starter-sql-postgresql'
```
{% endtab %}

{% tab title="SQL Server" %}
{% hint style="danger" %}
Cloud SQL Starter is not supported for SQL Server. Use Cloud SQL Proxy instead.
{% endhint %}
{% endtab %}
{% endtabs %}

Configure Spring Boot application's`application.properties` with [Instance Connection Name](cloud-sql.md#instance-connection-name) and the database name:

```bash
# Retrieve instance connection name from the previous step
spring.cloud.gcp.sql.instance-connection-name=INSTANCE_CONNECTION_NAME
spring.cloud.gcp.sql.database-name=orders

# Cloud SQL starter automatically configures the JDBC URL

# Configure username/password
spring.datasource.username=...
spring.datasource.password=...

# Configure connection pooling if needed
spring.datasource.hikari.maximum-pool-size=10
```

### Cloud SQL Socket Factory

If you don't use Spring Cloud GCP's Cloud SQL starter, and need to configure JDBC URL directly, you can use [Cloud SQL Socket Factory](https://github.com/GoogleCloudPlatform/cloud-sql-jdbc-socket-factory) with existing JDBC driver.

Add the Cloud SQL Socket Factory dependency:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>com.google.cloud.sql</groupId>
    <artifactId>postgres-socket-factory</artifactId>
    <version>1.0.15</version>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash
compile 'com.google.cloud.sql:postgres-socket-factory:1.0.15'
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="MySQL" %}
MySQL instance's JDBC URL with Cloud SQL Socket Factory follows the format of:

```bash
jdbc:mysql:///DATABASE_NAME?cloudSqlInstance=INSTANCE_CONNECTION_NAME&socketFactory=com.google.cloud.sql.mysql.SocketFactory
```

The JDBC URL for the Cloud SQL instance in this example is:

```bash
jdbc:mysql:///orders?cloudSqlInstance=PROJECT_ID:us-central1:mysql-instance&socketFactory=com.google.cloud.sql.mysql.SocketFactory
```
{% endtab %}

{% tab title="PostgreSQL" %}
PostgreSQL instance's JDBC URL with Cloud SQL Socket Factory follows the format of:

```text
jdbc:postgresql:///DATABASE_NAME?cloudSqlInstance=INSTANCE_CONNECTION_NAME&socketFactory=com.google.cloud.sql.postgres.SocketFactory
```

The JDBC URL for the Cloud SQL instance in this example is:

```text
jdbc:postgresql:///orders?cloudSqlInstance=PROJECT_ID:us-central1:postgresql-instance&socketFactory=com.google.cloud.sql.postgres.SocketFactory
```

 
{% endtab %}

{% tab title="SQL Server" %}
{% hint style="danger" %}
Cloud SQL Socket Factory is not supported for SQL Server. Use Cloud SQL Proxy instead.
{% endhint %}
{% endtab %}
{% endtabs %}

### VPC Private IP

If your Cloud SQL instance is on [VPC and has a private IP](https://cloud.google.com/sql/docs/mysql/private-ip), and your application is running in the Cloud able to access the same VPC, then configure JDBC drivers normally connecting to the private IP address.

## Cloud SQL Proxy

[Cloud SQL Proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy) is the generic way of establishing secured connection to a Cloud SQL instance. Rather than using the Cloud SQL Socket Factory to exchange certificates, Cloud SQL Proxy will authenticate and exchange the certificates.  


![Cloud SQL Proxy diagram](../../../.gitbook/assets/image%20%284%29.png)

Install Cloud SQL Proxy:

```bash
gcloud components install cloud_sql_proxy
```

Start the proxy:

{% tabs %}
{% tab title="MySQL" %}
```bash
# Refer to Instance Connection Name from previous section
cloud_sql_proxy -instances=INSTANCE_CONNECTION_NAME=tcp:3306
```
{% endtab %}

{% tab title="PostgreSQL" %}
```bash
# Refer to Instance Connection Name from previous section
cloud_sql_proxy -instances=INSTANCE_CONNECTION_NAME=tcp:5432
```
{% endtab %}

{% tab title="SQL Server" %}
```bash
# Refer to Instance Connection Name from previous section
cloud_sql_proxy -instances=INSTANCE_CONNECTION_NAME=tcp:1433
```
{% endtab %}
{% endtabs %}

You can then establish connections on `localhost` with the corresponding ports.

{% tabs %}
{% tab title="MySQL" %}
```bash
mysql -u root -p
```
{% endtab %}

{% tab title="PostgreSQL" %}
```
psql -h localhost -U postgres
```
{% endtab %}

{% tab title="SQL Server" %}
```
mssql-cli -U sqlserver
```
{% endtab %}
{% endtabs %}

## R2DBC

You can use R2DBC driver for reactive database access when you connect to Cloud SQL instances using:

* Cloud SQL Proxy
* VPC Private IP

{% hint style="danger" %}
R2DBC driver does not work with Cloud SQL Socket Factory. A [GitHub issue](https://github.com/GoogleCloudPlatform/cloud-sql-jdbc-socket-factory/issues/164) is currently open to track this support.
{% endhint %}

See [R2DBC documentation](https://r2dbc.io/) for corresponding driver usages:

* [r2dbc-mysql](https://github.com/mirromutth/r2dbc-mysql)
* [r2dbc-postgresql](https://github.com/r2dbc/r2dbc-postgresql)
* [r2dbc-mssql](https://github.com/r2dbc/r2dbc-mssql)

