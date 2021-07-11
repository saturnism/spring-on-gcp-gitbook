# External Load Balancing

This section continues from the previous section - make sure you do the tutorial in sequence.

{% page-ref page="../service.md" %}

There are primarily 2 ways to expose a Kubernetes Service on the public internet:

| Type | Protocol | Locality | When to use? |
| :--- | :--- | :--- | :--- |
| [External Network Load Balancer](https://cloud.google.com/load-balancing/docs/network) | TCP/UDP | Regional | Non-HTTP requests, or no need for a global load balancer. Connection to the Load Balancer is routed by public Internet to region of the load balancer. |
| [External HTTP\(s\) Load Balancer](https://cloud.google.com/load-balancing/docs/https) | HTTP\(s\) | Global | HTTP requests. GCP's L7 Load Balancer is a global load balancer - a single IP address can automatically route traffic to the nearest region within the GCP network. |

## External Network Load Balancer

### Service YAML

To create an external network load balancer, simply change Kubernetes Service's type from `clusterip` to `loadbalancer`. Modify the `k8s/service.yaml`:

{% code title="k8s/service.yaml" %}
```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloworld
  labels:
    app: helloworld
  annotations:
    cloud.google.com/neg: '{"exposed_ports": {"8080":{}}}'
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

### Deploy

Use `kubectl` command line to deploy the YAML file:

```bash
kubectl apply -f k8s/service.yaml
```

To verify the application is deployed, run :

```bash
kubectl get svc helloworld
```

You should see that the Service has a Cluster IP address, but also the External IP address with the initial value of `<pending>`. This is because, behind the scenes, Kubernetes Engine is provisioning a Google Cloud Network Load Balancer.

```bash
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
helloworld   ClusterIP   ...          <pending>     8080/TCP   7s
```

### Connect

Continuously check the External IP address, until an IP address is assigned. Once the IP Address is assigned, you can connect to the External IP address, and it'll be load balanced to the `helloworld` service backend pods.

```bash
EXTERNAL_IP=$(kubectl get svc helloworld -ojsonpath="{.status.loadBalancer.ingress[0].ip}")
curl $EXTERNAL_IP:8080
```

### Static IP Address

You can assign a static IP address to the Network Load Balancer.

Reserve a regional static IP address:

```bash
REGION=$(gcloud config get-value compute/region)
gcloud compute addresses create helloworld-service-ip --region=${REGION}
```

See the reserved IP address:

```bash
REGION=$(gcloud config get-value compute/region)
gcloud compute addresses describe helloworld-service-ip --region=${REGION} --format='value(address)'
```

Update the `k8s/service.yaml` to pin the Load Balancer IP address:

{% code title="k8s/service.yaml" %}
```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloworld
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

## External HTTP Load Balancer

You can configure an external HTTP load balancer using Kubernetes Ingress. In order for the HTTP Load Balancer to find the backends, it's recommended to use [container-native load balancing](https://cloud.google.com/kubernetes-engine/docs/how-to/container-native-load-balancing) on Google Cloud.

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

Create a Kubernetes Ingress configuration that will create the HTTP Load Balancer. Create a `k8s/ingress.yaml`:

{% code title="k8s/ingress.yaml" %}
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: helloworld
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          service:
            name: helloworld
            post:
              number: 8080
```
{% endcode %}

### Deploy

Use `kubectl` command line to deploy the YAML files:

```bash
# Delete the existing service because it may contain a node port:
kubectl delete -f k8s/service.yaml

# Redeploy the service
kubectl apply -f k8s/service.yaml

# Deploy the ingress
kubectl apply -f k8s/ingress.yaml
```

To verify the Ingress is deployed:

```bash
kubectl get ingress helloworld
```

You should see that the Ingress has an IP address provisioned:

```bash
NAME         HOSTS   ADDRESS         PORTS   AGE
helloworld   *       ...             80      81s
```

Many Google Cloud components are being configured behind the scenes to enable global load balancing. It'll take a few minutes before the address is accessible. Use `kubectl describe` to see the current status:

```bash
kubectl describe ingress helloworld
```

Initially, you may see:

```bash
Name:             helloworld
Namespace:        default
Address:          ...
Default backend:  helloworld:8080 (...)
Rules:
  Host  Path  Backends
  ----  ----  --------
  *     *     helloworld:8080 (...)
Annotations:
  ...
  ingress.kubernetes.io/backends:  {"...":"Unknown"}
```

When the annotation value of `ingress.kubernetes.io/backends` is `Unknown`, it means that the backend is not yet accessible.

Re-check the status until the backend becomes `HEALTHY`.

```bash
Name:             helloworld
Namespace:        default
Address:          ...
Default backend:  helloworld:8080 (...)
Rules:
  Host  Path  Backends
  ----  ----  --------
  *     *     helloworld:8080 (...)
Annotations:
  ...
  ingress.kubernetes.io/backends:  {"...":"HEALTHY"}
```

### Connect

You can then use the IP address to connect:

```bash
EXTERNAL_IP=$(kubectl get ingress helloworld -ojsonpath="{.status.loadBalancer.ingress[0].ip}")
curl $EXTERNAL_IP
```

### Static IP Address

By default, the Ingress IP address is ephemeral - it'll change if you ever delete and recreate the Ingress. You can associate the Ingress with a static IP address instead.

#### Global Static IP Address

Reserve a global static IP address:

```bash
gcloud compute addresses create helloworld-ingress-ip --global
```

See the static IP address you reserved:

```bash
gcloud compute addresses describe helloworld-ingress-ip --global \
  --format='value(address)'
```

#### Configurations

In `k8s/ingress.yaml`, use the `kubernetes.io/ingress.global-static-ip-name` annotation to specify the IP name:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: helloworld
  annotations:
    kubernetes.io/ingress.global-static-ip-name: "helloworld-ingress-ip"
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          service:
            name: helloworld
            post:
              number: 8080
```

#### Deploy

Deploy the Ingress:

```bash
kubectl apply -f k8s/ingress.yaml
```

Continuously check the IP address to be updated. It'll take several minutes for the IP address to update:

```bash
kubectl get ingress helloworld
```

### SSL Certificate

In order to use a SSL certificate to serve HTTPs traffic, you must use a real fully qualified domain name and configure it to point to the IP address. If you don't have a real domain, then you can use [xip.io](https://xip.io).

```bash
EXTERNAL_IP=$(kubectl get ingress helloworld -ojsonpath="{.status.loadBalancer.ingress[0].ip}")
DOMAIN="${EXTERNAL_IP}.xip.io"
curl $DOMAIN

echo $DOMAIN
```

You can provision the External HTTP\(s\) Load Balancer using Ingress with a Managed Certificate, or you can provide your own Self-Managed Certificate.

#### Managed Certificate

Google Cloud can automatically provision a certificate for your domain name when using the External HTTP\(s\) Load Balancer.

Create a new `k8s/certificate.yaml`:

{% tabs %}
{% tab title="With xip.io" %}
```bash
EXTERNAL_IP=$(kubectl get ingress helloworld -ojsonpath="{.status.loadBalancer.ingress[0].ip}")
DOMAIN="${EXTERNAL_IP}.xip.io"

cat << EOF > k8s/certificate.yaml
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: helloworld-certificate
spec:
  domains:
  # Replace the value with your domain name
  - ${DOMAIN}
EOF
```
{% endtab %}

{% tab title="With Custom Domain" %}
{% code title="k8s/certificate.yaml" %}
```yaml
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: helloworld-certificate
spec:
  domains:
  # Replace the value with your domain name
  - YOUR_DOMAIN_NAME
```
{% endcode %}
{% endtab %}
{% endtabs %}

In `k8s/ingress.yaml`, use the `networking.gke.io/managed-certificates` annotation to associate the certificate:

{% code title="k8s/ingress.yaml" %}
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: helloworld
  annotations:
    kubernetes.io/ingress.global-static-ip-name: "helloworld-ingress-ip"
    # Associate the ingress with the certificate name
    networking.gke.io/managed-certificates: "helloworld-certificate"
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          service:
            name: helloworld
            post:
              number: 8080
```
{% endcode %}

Deploy both files:

```bash
kubectl apply -f k8s/certificate.yaml
kubectl apply -f k8s/ingress.yaml
```

It may take several minutes to provision the certificate. Check the Managed Certificate status:

```bash
kubectl describe managedcertificate helloworld-certificate
```

Wait until the Certificate Status becomes `ACTIVE`:

```text
Name:         helloworld
Namespace:    default
...
Status:
  Certificate Name:    ...
  Certificate Status:  Active
...
```

You can then use HTTPs to connect:

{% tabs %}
{% tab title="Use xip.io" %}
```bash
EXTERNAL_IP=$(kubectl get ingress helloworld -ojsonpath="{.status.loadBalancer.ingress[0].ip}")
DOMAIN="${EXTERNAL_IP}.xip.io"

curl "https://${DOMAIN}"
```
{% endtab %}

{% tab title="With Custom Domain" %}
```bash
curl https://YOUR_DOMAIN_NAME
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
See [Using Google-managed SSL certificates](https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs) for more details.
{% endhint %}

#### Self-Managed Certificate

You can configure the Ingress to serve with your own SSL certificate. Usually you would already have a certificate/key pair.

If you don't already have one, you can provision a self-signed certificate for non-production use.

```bash
EXTERNAL_IP=$(kubectl get ingress helloworld -ojsonpath="{.status.loadBalancer.ingress[0].ip}")
DOMAIN="${EXTERNAL_IP}.xip.io"

mkdir -p cert/

# Generate a key
openssl genrsa -out cert/helloworld-tls.key 2048

# Generate a certificate signing request
openssl req -new -key cert/helloworld-tls.key \
  -out cert/helloworld-tls.csr \
  -subj "/CN=${DOMAIN}"
  
# 
openssl x509 -req -days 365 -in cert/helloworld-tls.csr \
  -signkey cert/helloworld-tls.key \
  -out cert/helloworld-tls.crt
```

Create a Kubernetes Secret to hold the certificate/key pair:

```bash
kubectl create secret tls helloworld-tls \
  --cert cert/helloworld-tls.crt --key cert/helloworld-tls.key \
  --dry-run -oyaml > k8s/tls-secret.yaml
```

Update the Ingress to refer to the secret for TLS certificate/key pair:

{% code title="k8s/ingress.yaml" %}
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: helloworld
  annotations:
    kubernetes.io/ingress.global-static-ip-name: "helloworld-ingress-ip"
spec:
  # Associate with the TLS certificate/key pair by the secret name
  tls:
  - secretName: helloworld-tls
  rules:
  - http:
      paths:
      - path: /*
        backend:
          service:
            name: helloworld
            post:
              number: 8080
```
{% endcode %}

Deploy the configurations:

```bash
kubectl apply -f k8s/tls-secret.yaml
kubectl apply -f k8s/ingress.yaml
```

It will take several minutes for the new configuration to take effect.

You can then use HTTPs to connect. However, if you used a self-signed certificate, you will need to ignore certificate validation errors:

```bash
EXTERNAL_IP=$(kubectl get ingress helloworld -ojsonpath="{.status.loadBalancer.ingress[0].ip}")
DOMAIN="${EXTERNAL_IP}.xip.io"

curl -k "https://${DOMAIN}"
```

{% hint style="info" %}
See [Using multiple SSL certificates in HTTP\(s\) load balancing with Ingress](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-multi-ssl) for more details.
{% endhint %}

