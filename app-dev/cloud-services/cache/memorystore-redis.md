# Memorystore Redis

## Memorystore Redis Instance

### Enable API

```bash
gcloud services enable redis.googleapis.com
```

{% hint style="warning" %}
Enabling this API may take a few minutes.
{% endhint %}

### Create an Instance

```bash
gcloud redis instances create orders-cache \
  --size=1 --region=us-central1
```

{% hint style="warning" %}
Creating a Redis instance may take a few minutes.
{% endhint %}

### Get Instance IP Address

```text
gcloud redis instances describe orders-cache \
  --region=us-central1 --format="value(host)"
```

{% hint style="warning" %}
The IP address is not a static IP address. If you create the instance, the IP address may be different.
{% endhint %}

### Connect to Instance

The Memorystore instance is only accessible from the VPC. You can connect from different Google Cloud Platform computing resources differently.  In general, VM-based products \(Compute Engine, Kubernetes Engine, and App Engine Flexible\) requires the VM to be on the same VPC, and Serverless products requires [VPC Service Connector](https://cloud.google.com/vpc/docs/configure-serverless-vpc-access).

| Resource | Method | Guide |
| :--- | :--- | :--- |
| Compute Engine | Out-of-the-box | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-gce) |
| Kubernetes Engine | [VPC-Native cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/alias-ips) | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-gke) |
| App Engine Flexible | Out-of-the-box | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-flex#java_1) |
| App Engine Standard | [VPC Service Connector](https://cloud.google.com/appengine/docs/standard/java11/connecting-vpc) | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-standard) |
| Cloud Run | [VPC Service Connector](https://cloud.google.com/run/docs/configuring/connecting-vpc) | [Guide](https://cloud.google.com/run/docs/configuring/connecting-vpc) |
| Cloud Function | [VPC Service Connector](https://cloud.google.com/functions/docs/networking/connecting-vpc) | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-functions) |

## Spring Boot Cache

Spring Boot can [use Redis](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-caching-provider-redis)  to [cache with annotations](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-caching).

## Spring Boot Session

Spring Boot can [use Redis for session data](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-session). 

