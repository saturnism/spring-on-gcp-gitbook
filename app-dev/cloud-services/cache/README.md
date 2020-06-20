# Cache

## Memorystore

Memory Store is a fully managed in-memory data store service having protocol compatibility with Redis and Memcached. See documentations for [Memorystore Redis](https://cloud.google.com/memorystore/docs/redis/) and [Memorystore Memcached](https://cloud.google.com/memorystore/docs/memcached) \(beta\) for more information.

#### Zonal Resource

Memorystore is zonal, meaning each Memorystore instance is only available within a zone, or accessible from other zones within the same region.  For high-availability, create a **Standard** tier instance, which includes a failover replica in a separate zone.

#### Connectivity

All Memorystore instances can only be accessed by a private IP on a VPC network. You can connect to a Memorystore instance from different Google Cloud Platform computing resources differently.  In general, VM-based products \(Compute Engine, Kubernetes Engine, and App Engine Flexible\) requires the VM to be on the same VPC, and Serverless products requires [VPC Service Connector](https://cloud.google.com/vpc/docs/configure-serverless-vpc-access).

| Resource | Method | Guide |
| :--- | :--- | :--- |
| Compute Engine | Out-of-the-box | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-gce) |
| Kubernetes Engine | [VPC-Native cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/alias-ips) | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-gke) |
| App Engine Flexible | Out-of-the-box | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-flex#java_1) |
| App Engine Standard | [VPC Service Connector](https://cloud.google.com/appengine/docs/standard/java11/connecting-vpc) | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-standard) |
| Cloud Run | [VPC Service Connector](https://cloud.google.com/run/docs/configuring/connecting-vpc) | [Guide](https://cloud.google.com/run/docs/configuring/connecting-vpc) |
| Cloud Function | [VPC Service Connector](https://cloud.google.com/functions/docs/networking/connecting-vpc) | [Guide](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-functions) |

#### Protocol Compatibility

Because Memorystore is protocol compatible. You can use existing Spring Boot integration with Redis and Memcached as-is.

{% page-ref page="memorystore-redis.md" %}

{% page-ref page="memorystore-memcached.md" %}





