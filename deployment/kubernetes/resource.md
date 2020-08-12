# Resource Allocation

This section continues from the previous section - make sure you do the tutorial in sequence.

{% page-ref page="deployment.md" %}

## Default Configuration

You can specify the computing resource needs for each of the containers. By default, each container is given 10% of a CPU and no memory use restrictions.

{% hint style="danger" %}
The defaults can cause issues:

* If a Node has 1 full CPU, then Kubernetes may schedule up to 10 instances of the same container, which may overload the system.
* If a Node has 16GB of RAM, and without memory restriction, then each container instance \(JVM\) may think they each can use up to 16GB, causing memory overuse \(and thus, virtual memory swapping, etc\)
{% endhint %}

You can see the current resource by describing a Pod instance, look for the Requests/Limits lines.

```bash
POD_NAME=$(kubectl get pods -lapp=helloworld -o jsonpath='{.items[0].metadata.name}')

kubectl describe pod $POD_NAME
```

The details should have a `Requests` section with `cpu` value set to `100m`:

```text
Name:           helloworld-...
Namespace:      default...
Containers:
  helloworld:
    ...
    Requests:
      cpu:  100m
...
```

{% hint style="info" %}
The default value is `100m`, which means `100 milli` = `100/1000` = `10%`of a vCPU core.
{% endhint %}

The default is configured per Namespace. The application was deployed into the `default` Namespace. Look at the default resource configuration for this Namespace:

```bash
kubectl describe ns default
```

See the output:

```bash
Name:         default
Labels:       <none>
Annotations:  <none>
Status:       Active

Resource Quotas
 Name:                       gke-resource-quotas
 Resource                    Used  Hard
 --------                    ---   ---
 count/ingresses.extensions  1     100
 count/jobs.batch            0     5k
 pods                        3     1500
 services                    2     500

Resource Limits
 Type       Resource  Min  Max  Default Request  Default Limit  Max Limit/Request Ratio
 ----       --------  ---  ---  ---------------  -------------  -----------------------
 Container  cpu       -    -    100m             -              -
```

However, the configuration is actually stored in a `LimitRange` Kubernetes resource:

```bash
kubectl get limitrange limits -oyaml
```

{% hint style="info" %}
The default can be updated. [See Configure Default CPU Requests and Limits for a Namespace](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/) documentation.
{% endhint %}

## Resource Request

In Kubernetes, you can reserve capacity by setting the Resource Requests to reserve more CPU and memory. Configure the deployment to reserve at least `20%` of a CPU, and `128Mi` of RAM.

{% code title="k8s/deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: helloworld
  name: helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - image: gcr.io/.../helloworld
        name: helloworld
        # Add the resources requests block
        resources:
          requests:
            cpu: 200m
            memory: 128Mi
```
{% endcode %}

{% hint style="info" %}
In this example, CPU request is`200m` means `200 milli`=`200/1000` = `20%` of 1 vCPU core.

Memory is `128Mi`, which is `128 Mebibytes` = `~134 Megabytes`.
{% endhint %}

{% hint style="info" %}
See [Kubernetes Resource Units](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes) documentation for the units descriptions such as `m`, `M`, and `Mi`.
{% endhint %}

{% hint style="danger" %}
When specifying the Memory resource allocation, do not accidentally use `m` as the unit. `128m` means `0.128 bytes`.
{% endhint %}

## Resource Limit

The application can consume more CPU and memory than requested - they can burst up to the limit, but cannot exceed the limit. Configure the deployment to set the limit:

{% code title="k8s/service.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: helloworld
  name: helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - image: gcr.io/.../helloworld
        name: helloworld
        # Add the resources requests block
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 256Mi
```
{% endcode %}

{% hint style="info" %}
CPU limit is a _compressible_ resource. If the application exceeds the CPU limit, it'll simply be throttled, and thus capping the latency and  throughput.
{% endhint %}

{% hint style="danger" %}
Memory is bot a compressible resource. If the application exceeds the Memory limit, then the container will be killed \(`OOMKilled`\) and restarted.
{% endhint %}

{% hint style="info" %}
For Java applications, read the [Container Awareness](../docker/container-awareness.md) section to make sure you are using a Container-Aware OpenJDK version to avoid unnecessary `OOMKilled` errors.
{% endhint %}

