---
description: >-
  In this section, you'll learn how to create a Kubernetes service and how
  service discovery works.
---

# Service

## Service

Each Pod has a unique IP address - but the address is ephemeral. The Pod IP addresses are not stable and it can change when Pods start and/or restart. Moreover, if you have a Deployment that starts multiple Pods, and you need to consume an API from the Pods, you do not want to connect using an ephemeral IP address either. Usually, you'll need to put a load balancer to distribute traffic to each individual instances.

In Kubernetes, a Service is a Network \(L4\) Load Balancer that'll provide you a single stable Service IP \(and hostname\) to load balance the traffic to a set of pods selected by using labels.

### Service YAML

You can create a Service and deploy into Kubernetes using `kubectl` command line like in the [Hello World tutorial](../../getting-started/helloworld/kubernetes-engine.md). That's great to get a feel of Kubernetes. However, it's best that you create a YAML file first, and then deploy the YAML file.

```bash
# Under the helloworld-springboot-tomcat directory , create a k8s directory
mkdir k8s/

kubectl create service clusterip helloworld \
  --tcp=8080:8080 \
  --dry-run \
  -o yaml > k8s/service.yaml
```

You can open the `k8s/service.yaml` file to see the content. Following is a version of the YAML file where it's slimmed down to the bare minimum.

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

{% hint style="info" %}
You can read more about Deployment in the [Kubernetes Service Guide](http://kubernetes.io/docs/user-guide/deployments/).
{% endhint %}

### Deploy

Use `kubectl` command line to deploy the YAML file:

```bash
kubectl apply -f k8s/service.yaml
```

To verify the application is deployed, see all the pods that are running:

```bash
kubectl get svc
```

You should see that the Service is configured!

```bash
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
helloworld   ClusterIP   ...          <none>        8080/TCP   7s
```

The newly created Service \(L4 Load Balancer\) can be accessed using the Cluster IP, or the hostname `helloworld`, or a Fully Qualified Name \(FQN\) of `helloworld.default.svc.cluster.local`.

{% hint style="info" %}
See [Kubernetes DNS for Services and Pods documentation](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) for more detail on how a DNS name is associated with a Service.
{% endhint %}

### Endpoints

Each Service has a `selector` block that is used to find Pods with matching labels and enlisting them as an Endpoint of the Service \(or, you can think of it as a backend instance of a load balancer\).

In the example above, the selector is `app: helloworld`, you can also find the matching Pods using `kubectl`:

```bash
# Scale out the # of instances so we can see more than one pod
kubectl scale deployment helloworld --replicas=2

# Find Pods using a selector
kubectl get pods -lapp=helloworld
```

Describe the Service to see the Endpoint IP addresses that it's currently enlisted for the Service, and look for the `Endpoints` output:

```bash
kubectl describe svc helloworld
```

## Service Discovery

In the previous section, you created a Service. A service can also be used for service discovery - by API, or by IP, or by the DNS name.

