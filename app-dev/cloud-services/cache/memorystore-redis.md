# Memorystore Redis

## Memorystore Redis Instance

### Enable API

```bash
gcloud services enable servicenetworking.googleapis.com
gcloud services enable redis.googleapis.com
```

{% hint style="warning" %}
Enabling this API may take a few minutes.
{% endhint %}

### Create an Instance

Create an instance and attach it to the default VPC.

```bash
gcloud redis instances create orders-cache \
  --size=1 --region=us-central1
```

{% hint style="warning" %}
Creating a Redis instance may take a few minutes.
{% endhint %}

### Get Instance IP Address

```bash
gcloud redis instances describe orders-cache \
  --region=us-central1 --format="value(host)"
```

{% hint style="warning" %}
The IP address is not a static IP address. If you create the instance, the IP address may be different.
{% endhint %}

### Connect to Instance

See [Memorystore connectivity options](./#connectivity) to see how to connect to a Memorystore instance from different computing environments.

| Computing Environment |  |
| :--- | :--- |
| Compute Engine | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-gce) |
| Kubernetes Engine | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-gke) |
| App Engine Flexible | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-flex#java_1) |
| App Engine Standard | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-standard) |
| Cloud Run | [Guide](https://cloud.google.com/run/docs/configuring/connecting-vpc) |
| Cloud Function | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-functions) |

You can test quickly by creating a Compute Engine instance in a zone within the same region:

```bash
gcloud compute instances create test-memorystore-vm --zone=us-central1-c
```

SSH into the machine:

```bash
gcloud compute ssh test-memorystore-vm --zone=us-central1-c
```

Install `redis-cli`:

```bash
sudo apt-get update && sudo apt-get install -y redis-tools
```

Connect to the instance:

```bash
redis-cli -h <MEMORYSTORE_REDIS_IP>
```

You can try different Redis commands, for example:

```text
> PING
PONG
> SET greeting Hello
OK
> GET greeting
"Hello"
```

{% hint style="info" %}
See [redis-cli documentation](https://redis.io/topics/rediscli) for more information.
{% endhint %}

## Spring Boot Cache

Spring Boot can [use Redis](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-caching-provider-redis) to [cache with annotations](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-caching).

### Dependency

Add the Spring Data Redis starter:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash
compile group: 'org.springframework.cloud', name: 'spring-boot-starter-cache'
compile group: 'org.springframework.cloud', name: 'spring-boot-starter-data-redis'
```
{% endtab %}
{% endtabs %}

### Configuration

Configure the Redis instance to connect to:

{% code title="application.properties" %}
```bash
spring.redis.host=<MEMORYSTORE_REDIS_IP>

# Configure default TTL, e.g., 10 minutes
spring.cache.redis.time-to-live=600000
```
{% endcode %}

### Enable Caching

Turn on caching capability explicitly with the `@EnableCaching` annotation:

```java
@SpringBootApplication
@EnableCaching
class DemoApplication {
  ...
}
```

### Cacheable

Once you configured the Spring Boot with Redis and enabled caching, you can use the `@Cacheable` annotation to cache return values.

```java
@Service
class OrderService {
  private final OrderRepository orderRepository;
  
  public OrderService(OrderRepository orderRepository) {
    this.orderRepository = orderRepository;
  }
  
  @Cacheable("order")
  public Order getOrder(Long id) {
    orderRepository.findById(id);
  }
}
```

{% hint style="info" %}
Read Spring Boot documentation on [Cacheable](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-caching) and [Redis](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-caching-provider-redis) for more information.
{% endhint %}

## Spring Boot Session

Spring Boot can [use Redis for session data](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-session) with [Spring Session Data Redis](https://docs.spring.io/spring-session/docs/2.3.0.RELEASE/reference/html5/#httpsession-redis).

### Dependency

Add the Spring Data Spanner starter:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash
compile group: 'org.springframework.cloud', name: 'spring-session-data-redis'
compile group: 'org.springframework.cloud', name: 'spring-boot-starter-data-redis'
```
{% endtab %}
{% endtabs %}

### Configuration

Configure the Redis instance to connect to:

{% code title="application.properties" %}
```bash
spring.redis.host=<MEMORYSTORE_REDIS_IP>

# Configure default TTL, e.g., 10 minutes
spring.cache.redis.time-to-live=600000
```
{% endcode %}

### Enable HTTP Session

Turn on caching capability explicitly with the `@EnableSpringHttpSession` annotation:

```java
@SpringBootApplication
@EnableSpringHttpSession
class DemoApplication {
  ...
}
```

### Samples

* [Spring Boot Session Data Redis sample](https://github.com/spring-projects/spring-session/tree/master/spring-session-samples/spring-session-sample-boot-redis-simple)

