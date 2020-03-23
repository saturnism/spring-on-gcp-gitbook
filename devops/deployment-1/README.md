# Deployment



|  | Compute Engine | App Engine | Cloud Run | Cloud Functions | Kubernetes Engine |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Description** | Virtual machines. | Platform as a service. | Serverless for containers. | Serverless for functions/lambdas. | Managed Kubernetes platform. |
| **Free Tier** | [Yes](https://cloud.google.com/free/docs/gcp-free-tier) | [Yes](https://cloud.google.com/free/docs/gcp-free-tier) | [Yes](https://cloud.google.com/free/docs/gcp-free-tier) | [Yes](https://cloud.google.com/free/docs/gcp-free-tier) | [Yes](https://cloud.google.com/free/docs/gcp-free-tier) |
| **Deploys** | Anything you bring. | Source or JAR | Container Image | Source or JAR | Kubernetes YAML, Container Image |
| **External Load Balancing** | Yes, with HTTP\(s\) or Network Load Balancer | Yes | Yes | Yes | Yes with Kubernetes Service / Ingress |
| **HTTPs** | Manual configuration | Yes, default | Yes, default | Yes, default | Manual configuration |
| **Custom Domains** | Yes | Yes | Yes | No | Yes |
| **Managed Certificates** | [Yes](https://cloud.google.com/load-balancing/docs/ssl-certificates) | [Yes](https://cloud.google.com/appengine/docs/standard/java11/securing-custom-domains-with-ssl) | [Yes](https://cloud.google.com/run/docs/mapping-custom-domains) | No | Yes, with [ManagedCertificate](https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs) |
| **Internal Load Balancing on VPC** | Yes | No | No | No | Yes |
| **Access VPC Resource** | Yes | Yes \(alpha\) | Yes \(alpha\) | Yes \(alpha\) | Yes |
| **Autoscaling** | [Yes with Managed Instance Group and Autoscaler](https://cloud.google.com/compute/docs/autoscaler) | [Yes](https://cloud.google.com/appengine/docs/standard/java/how-instances-are-managed#scaling_types) | [Yes](https://cloud.google.com/run/docs/about-instance-autoscaling) | [Yes](https://cloud.google.com/functions/docs/concepts/exec#auto-scaling_and_concurrency) | Yes with [Horizontal Pod Autoscaling](https://cloud.google.com/kubernetes-engine/docs/concepts/horizontalpodautoscaler) and [Cluster Autoscaler](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler) |
| **Long Running Processes** | Yes | Yes with Manual Scaling | No | No | Yes |
| **Billing Model** | [VM Hours](https://cloud.google.com/compute/all-pricing) | [Instance Hours](https://cloud.google.com/appengine/pricing#standard_instance_pricing) | [Instance execution time](https://cloud.google.com/run/pricing) | [Invocations + Compute time](https://cloud.google.com/functions/pricing) | [Management + VM Hours](https://cloud.google.com/kubernetes-engine/pricing) |



