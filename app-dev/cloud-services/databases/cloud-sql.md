# Cloud SQL

Cloud SQL is managed MySQL, PostgreSQL, and SQL Server.  Cloud SQL automates backups, replication, and failover to ensure your database is reliable, highly available.

Cloud SQL has automatic data encryption at rest and in transit. Private connectivity with Virtual Private Cloud \(VPC\) and user-controlled network access that includes firewall protection. Compliant with SSAE 16, ISO 27001, PCI DSS v3.0, and HIPAA

## Create a Instance

### Enable API

```bash
gcloud services enable sqladmin.googleapis.com
```

### Create an Instance

{% tabs %}
{% tab title="MySQL" %}
Create a new Cloud SQL - MySQL Instance with 

```bash
gcloud sql instances create example-db \
  --database-version=MYSQL_5_7 \
  --region=us-central1
```
{% endtab %}

{% tab title="PostgreSQL" %}
Create a new Cloud SQL - PostgreSQL instance.

```bash
gcloud sql instances create example-db \
  --database-version=POSTGRES_11 \
  --region=us-central1
```
{% endtab %}

{% tab title="SQL Server" %}
Create a new Cloud SQL - SQL Server instance.

```bash
gcloud beta sql instances create example-db \
  --database-version=SQLSERVER_2017_STANDARD \
  --region=us-central1
```
{% endtab %}
{% endtabs %}

### Create a Database

{% tabs %}
{% tab title="MySQL" %}

{% endtab %}

{% tab title="PostgreSQL" %}

{% endtab %}

{% tab title="SQL Server" %}

{% endtab %}
{% endtabs %}

## Cloud SQL Starter

### Spring JDBC Template

### Spring Data JPA

### Spring Data Rest Repository

## SQL Proxy

## R2DBC



