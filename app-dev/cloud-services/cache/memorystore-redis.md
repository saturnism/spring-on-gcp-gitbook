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

## Spring Boot Cache

Spring Boot can [use Redis](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-caching-provider-redis)  to [cache with annotations](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-caching).

## Spring Boot Session

Spring Boot can [use Redis for session data](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-session). 

