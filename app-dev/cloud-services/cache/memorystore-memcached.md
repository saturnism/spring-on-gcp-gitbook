# Memorystore Memcached \(beta\)

## Memorystore Memcached Instance

### Enable API

```bash
gcloud services enable servicenetworking.googleapis.com
gcloud services enable memcache.googleapis.com
```

{% hint style="warning" %}
Enabling this API may take a few minutes.
{% endhint %}

### Enable Private Service Access

Memorystore Memcached requires Private Services Access to be enabled. See [Establishing a private services access connection](https://cloud.google.com/memorystore/docs/memcached/establishing-connection) documentation for more information.

Reserve an IP address range to be used in a VPC, so that the Memcached instance's IP address can be allocated within this range:

```text
gcloud beta compute addresses create reserved-range \
  --global --prefix-length=24 \
  --description=description --network=default \
  --purpose=vpc_peering
```

{% hint style="info" %}
This is a simplified range creation on the `default` VPC network. In a production environment, you should verify what the range should be and which VPC network to allocate in.
{% endhint %}

Establish peering so that Memorystore can allocate the IP address in the reserved range in the VPC.

```text
gcloud services vpc-peerings connect \
  --service=servicenetworking.googleapis.com \
  --ranges=reserved-range --network=default
```

### Create an Instance

Create an instance and attach it to the default VPC.

```bash
gcloud beta memcache instances create orders-cache \
  --node-count=1 --node-cpu=1 --node-memory=1G --region=us-central1
```

{% hint style="warning" %}
Creating a Memcached instance may take a few minutes.
{% endhint %}

### Get Instance IP Address

```text
gcloud beta memcache instances describe orders-cache \
  --region=us-central1 --format="value(memcacheNodes.host)"
```

{% hint style="warning" %}
The IP address is not a static IP address. If you create the instance, the IP address may be different.
{% endhint %}

### Connect to Instance

See [Memorystore connectivity options](./#connectivity) to see how to connect to a Memorystore instance from different computing environments.

| Computing Environment |  |
| :--- | :--- |
| Compute Engine | [Guide](https://cloud.google.com/memorystore/docs/memcached/connecting-memcached-instance#connecting-compute-engine) |
| Kubernetes Engine | [Guide](https://cloud.google.com/memorystore/docs/memcached/connecting-memcached-instance#connecting_to_a_memcached_instance_from_a_cluster) |
| App Engine Flexible | [Additional Configuration](https://cloud.google.com/appengine/docs/flexible/java/using-shared-vpc) |
| App Engine Standard | [VPC Service Connector](https://cloud.google.com/appengine/docs/standard/java11/connecting-vpc) |
| Cloud Run | [VPC Service Connector](https://cloud.google.com/run/docs/configuring/connecting-vpc) |
| Cloud Function | [VPC Service Connector](https://cloud.google.com/functions/docs/networking/connecting-vpc) |

You can test quickly by creating a Compute Engine instance in a zone within the same region:

```text
gcloud compute instances create test-memorystore-vm --zone=us-central1-c
```

SSH into the machine:

```text
gcloud compute ssh test-memorystore-vm --zone=us-central1-c
```

Install `redis-cli`:

```text
sudo apt-get update && sudo apt-get install -y telnet
```

Connect to the instance:

```text
telnet <MEMORYSTORE_MEMCACHED_IP> 11211
```

You can try different Memcached commands, for example, `stats`:

```text
Trying ...
Connected to 10.111.98.4.
Escape character is '^]'.
stats
STAT pid 1
STAT uptime 1020
STAT time 1594348128
...
END
quit
Connection closed by foreign host.
```

{% hint style="info" %}
See [Memcached commands](https://github.com/memcached/memcached/wiki/Commands) for more information.
{% endhint %}

## Spring Boot Cache

Spring Boot does not have a built-in Memcached support. However you can use a 3rd party Memcached starter to provide Spring Boot cache support, e.g.:

* [https://github.com/sixhours-team/memcached-spring-boot](https://github.com/sixhours-team/memcached-spring-boot)

### Dependency

Add the 3rd party Memcached Spring Boot starter:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>io.sixhours</groupId>
    <artifactId>memcached-spring-boot-starter</artifactId>
    <version>2.1.2</version>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash
compile group: 'io.sixhours', name: 'memcached-spring-boot-starter:2.1.2'
```
{% endtab %}
{% endtabs %}

### Configuration

Configure the Memcached instance to connect to:

{% code title="application.properties" %}
```bash
memcached.cache.servers=<MEMORYSTORE_MEMCACHED_IP>:11211
memcached.cache.provider=static
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

