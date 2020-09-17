# Internal Load Balancing

This section continues from the previous section - make sure you do the tutorial in sequence.

{% page-ref page="../service.md" %}

## In-Cluster Load Balancer

A [Kubernetes Service](../service.md#service) acts as an internal L4 load balancer only accessible from within the same Kubernetes Cluster. See the [Service section](../service.md#service) for more information.

## Internal Network Load Balancer

The setup of the Internal Network Load Balancer is similar to the [External Network Load Balancer](external-load-balancing.md#external-network-load-balancer), but with an additional annotation.

### Service YAML

In `k8s/service.yaml`, use the `cloud.google.com/load-balancer-type` annotation to mark the service to use the Internal Network Load Balancer:

{% code title="k8s/service.yaml" %}
```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloworld
  annotations:
    # Indicate this is an Internal Network Load Balancer
    cloud.google.com/load-balancer-type: "Internal"
  labels:
    app: helloworld
spec:
  ports:
  - name: 8080-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: helloworld
  # Use LoadBalancer type instead of ClusterIP
  type: LoadBalancer
```
{% endcode %}

### Internal Static IP

You can assign an internal static IP address to the Network Load Balancer.

Reserve a regional static IP address:

```bash
REGION=$(gcloud config get-value compute/region)

gcloud compute addresses create helloworld-service-internal-ip \
  --subnet=default --region=${REGION}
```

See the reserved IP address:

```bash
REGION=$(gcloud config get-value compute/region)

gcloud compute addresses describe helloworld-service-internal-ip \
  --region=${REGION} --format='value(address)'
```

Update the `k8s/service.yaml` to pin the Load Balancer IP address:

{% code title="k8s/service.yaml" %}
```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloworld
  annotations:
    cloud.google.com/load-balancer-type: "Internal"
  labels:
    app: helloworld
spec:
  ports:
  - name: 8080-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: helloworld
  type: LoadBalancer
  # Replace the value with the IP address you reserved
  loadBalancerIP: RESERVED_IP_ADDRESS
```
{% endcode %}

## Internal HTTP\(s\) Load Balancer

The setup of the Internal Network Load Balancer is similar to the [External HTTP\(s\) Load Balancer](external-load-balancing.md#external-http-load-balancer), but with an additional annotation.

### Service YAML

In the `k8s/service.yaml`, use the `cloud.google.com/neg` annotation to enable Network Endpoint Group \(NEG\) in order to use container-native load balancing:

{% code title="k8s/service.yaml" %}
```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloworld
  labels:
    app: helloworld
  # Add the NEG annotation to enable Network Endpoint Group
  # in order to use container-native load balancing
  annotations:
    cloud.google.com/neg: '{"ingress": true}'
spec:
  ports:
  - name: 8080-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: helloworld
  type: ClusterIP
```
{% endcode %}

### Ingress YAML

Create a Kubernetes Ingress configuration that will create the HTTP Load Balancer. Create a `k8s/ingress.yaml`, but also use `kubernetes.io/ingress.class` annotation to indicate this is an Internal HTTP\(s\) Load Balancer

{% code title="k8s/ingress.yaml" %}
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: helloworld
  annotations:
    # Add the Ingress Class annotation to use Internal HTTP(s) Load Balancer
    kubernetes.io/ingress.class: "gce-internal"
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          serviceName: helloworld
          servicePort: 8080
```
{% endcode %}
