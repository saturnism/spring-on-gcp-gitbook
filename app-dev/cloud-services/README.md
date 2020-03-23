# Cloud Services

Spring Cloud GCP supports many GCP services.

| Concerns | GCP Service | Spring Abstraction |
| :--- | :--- | :--- |
| **Databases** | Cloud SQL | JDBC template |
|  |  | Spring Data JPA |
|  | Cloud Spanner | Spring Data Spanner |
|  |  | Spring Data JPA with Hibernate |
|  | Cloud Datastore | Spring Data Datastore |
|  | Cloud Firestore | Spring Reactive Data Firestore |
| **Messaging** | Cloud Pub/Sub | Pub/Sub Template |
|  |  | Spring Integration |
|  |  | Spring Cloud Stream |
|  |  | Spring Dataflow |
| **Configuration** | Cloud Secret Manager | Spring Cloud Config |
| **Storage** | Cloud Storage | Spring Resource |
| **Cache** | Cloud Memorystore | Spring Data Redis |
| **Distributed Tracing** | Cloud Trace | Spring Cloud Sleuth |
|  |  | Zipkin / Brave |
| **Centralized Logging** | Cloud Logging | SLF4J / Logback |
| **Monitoring Metrics** | Cloud Monitoring | Micrometer / Prometheus |
| **Security** | Cloud Identity Aware Proxy | Spring Security |

