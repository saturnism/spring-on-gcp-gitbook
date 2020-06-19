# Cache

## Memorystore

Memory Store is a fully managed in-memory data store service having protocol compatibility with Redis and Memcached. See documentations for [Memorystore Redis](https://cloud.google.com/memorystore/docs/redis/) and [Memorystore Memcached](https://cloud.google.com/memorystore/docs/memcached) \(beta\) for more information.

All Memorystore instances are accessible via a private IP that's local to your VPC network. Memorystore is zonal, meaning each Memorystore instance is only available within a zone, or accessible from other zones within the same region.  What that also means is that Memorystore does not have regional availability \(e.g., replication or fail-over across zones\) .



