---
description: Learn how to create a Kubernetes service and how service discovery works.
---

# Service

This section continues from the previous section - make sure you do the tutorial in sequence.

{% page-ref page="deployment.md" %}

## Service

Each Pod has a unique IP address - but the address is ephemeral. The Pod IP addresses are not stable and it can change when Pods start and/or restart. Moreover, if you have a Deployment that starts multiple Pods, and you need to consume an API from the Pods, you do not want to connect using an ephemeral IP address. Usually, you'll need to add a load balancer to distribute traffic to each individual instance.

In Kubernetes, a Service is a Network \(L4\) Load Balancer that'll provide you a single stable Service IP \(and hostname\) to load balance the traffic to a set of pods selected by using labels.

### Service YAML

You can create a Service and deploy into Kubernetes using the `kubectl` CLI like in the [Hello World tutorial](../../getting-started/helloworld/kubernetes-engine.md). That's great to get a feel of Kubernetes. However, it's best that you create a YAML file first, and then deploy the YAML file.

```bash
# Under the helloworld-springboot-tomcat directory , create a k8s directory
mkdir k8s/

kubectl create service clusterip helloworld \
  --tcp=8080:8080 \
  --dry-run \
  -o yaml > k8s/service.yaml
```

You can open the `k8s/service.yaml` file to see the content. Below is a version of the YAML file that's slimmed down to the bare minimum.

{% code title="k8s/service.yaml" %}
```yaml
# API Version and Kind are important to indicate the type of resource
apiVersion: v1
kind: Service
metadata:
  # Every Kubernetes resource has a name that's unique within a namespace
  name: helloworld
  # Every Kubernetes can have labels, label key/value pairs can be queried later.
  labels:
    app: helloworld
spec:
  # The type of the service - this one is an internal only service
  type: ClusterIP
  # Any Pods that matches these labels will be load balanced through
  # this Service (L4 load balancer)
  selector:
    app: helloworld
  ports:
    # A port can have a name, it can be renamed to be more descriptive
    # such as "http", "jmx", "metrics", etc.
  - name: 8080-8080 
    # The port to listen on by the Service (L4 Load Balancer)
    port: 8080
    # The port to forward traffic to on the destination Pod
    targetPort: 8080
    # TCP or UDP protocol
    protocol: TCP
```
{% endcode %}

{% hint style="info" %}
You can read more about Deployment in the [Kubernetes Service Guide](http://kubernetes.io/docs/user-guide/deployments/).
{% endhint %}

### Deploy

Use `kubectl` command line to deploy the YAML file:

```bash
kubectl apply -f k8s/service.yaml
```

Verify that the service is configured:

```bash
kubectl get svc helloworld
```

You should see that the Service has a Cluster IP address:

```bash
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
helloworld   ClusterIP   ...          <none>        8080/TCP   7s
```

## Service Discovery

In a traditional Spring Cloud application, service discovery is done using an external service registry like Eureka, and client-side load balancing with Ribbon.

In Kubernetes, the Kubernetes Service itself can act as the service registry:

* You can discover all the endpoints associated with a service
* Kubernetes Service's Cluster IP is a built-in L4 load balancer, and it's automatically associated with a DNS name

### Endpoints

Each Service has a `selector` block that is used to find Pods with matching labels and enlisting them as an Endpoint of the Service \(or, you can think of it as a backend instance of a load balancer\).

In the example above, the selector is `app: helloworld`, you can also find the matching Pods using `kubectl`:

```bash
# Scale out the # of instances so we can see more than one pod
kubectl scale deployment helloworld --replicas=2

# Find Pods using a selector
kubectl get pods -lapp=helloworld
```

Describe the Service to see the Endpoint IP addresses that it's currently enlisted,, and look for the `Endpoints` output:

```bash
kubectl describe svc helloworld
```

It is possible to continue to use Ribbon for client-side load balancing by retrieving these endpoints using the Kubernetes API. This is an advanced usage and not covered in this documentation. However, you can achieve a similar result if you use [Spring Cloud Kubernetes](https://spring.io/projects/spring-cloud-kubernetes) to replace Spring Cloud Eureka.

### DNS Name

The newly created Kubernetes Service is also a L4 Load Balancer, and it can be accessed using the Cluster IP, or the hostname `helloworld`, or a Fully Qualified Name \(FQN\) of `helloworld.default.svc.cluster.local`.

{% hint style="info" %}
See [Kubernetes DNS for Services and Pods documentation](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) for more detail on how a DNS name is associated with a Service.
{% endhint %}

This service is currently only accessible from within the Kubernetes cluster. I.e., there is no public IP address. You can start a new Pod within the cluster that has a shell, and execute commands within the cluster. This accurately simulates a client application sending request to another backend service.

`nginx` container image has a shell that we can use, so deploy one instance of `nginx`.

```bash
kubectl create deployment nginx --image=nginx
```

Attach to the `nginx` Pod:

```bash
POD_NAME=$(kubectl get pods -lapp=nginx -o jsonpath='{.items[0].metadata.name}')
kubectl exec -ti ${POD_NAME} /bin/bash
```

From within the Pod, `curl` the service:

```bash
# From within the nginx Pod
curl http://helloworld:8080

exit
```

{% hint style="info" %}
If you have a client service that needs to reach the `helloworld` service within the same cluster, you can simply use the DNS name. This will resolve to the Cluster IP, and subsequently, L4 load balanced to one of the backend endpoints.
{% endhint %}

