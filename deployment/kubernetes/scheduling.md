# Scheduling

## Anti-Affinity

By default, Kuberntes will schedule a pod onto a random node as long as the node has capacity to execute the pod based on the resource constraints. However, when you scale a deployment to `2`, there is a chance where both of the Pods are scheduled onto the same Kuberntes Node. This can cause issues if the Node goes down, causing both available Pods to shutdown and need to reschedule onto another node.

One solution is to avoid scheduling the pods onto the same node and this is called [ant-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity).

### Required Anti-Affinity

This example will enforce anti-affinity and not schedule any pods if anti-affinity requirement cannot be met. For the Hello World container, add additional configuration to make sure the pods are scheduled onto different nodes.

{% code title="k8s/deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld
  labels:
    app: helloworld
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
      - name: helloworld
        image: gcr.io/.../helloworld
      # Add configuration for anti-affinity
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - helloworld
            # Prefer spreading the Pods across multiple Nodes.
            # There are other keys you can use, e.g., anti-affinity
            # across zones.
            topologyKey: "kubernetes.io/hostname"
```
{% endcode %}

Scale out the deployment to 4 pods:

```bash
kubectl scale deployment helloworld --replicas=4
```

List all the pods and show which node it is running on:

```bash
kubectl get pods -lapp=helloworld -owide
```

Observe in the `NODE` column, that each pod is running on a different node.

But since the demo cluster only has 4 nodes, if you scale out to 5 pods, it can no longer satisfy the anti-affinity requirement, and the 5th pod will remain in the unschedulable \(Pending\) state:

```bash
kubectl scale deployment helloworld --replicas=5
```

Find the pending pod:

```bash
kubectl get pods -lapp=helloworld --field-selector='status.phase=Pending'
```

Describe it's detail:

```bash
POD_NAME=$(kubectl get pods -lapp=helloworld \
  --field-selector='status.phase=Pending' \
  -o jsonpath='{.items[0].metadata.name}')

kubectl describe pod $POD_NAME
```

{% hint style="info" %}
See Kubernetes [Affinity / Anti-Affinity documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity) for more information
{% endhint %}

### Preferred Anti-Affinity

In the Required case, if a pod cannot ensure anti-affinity, it'll simply not run. This may not be desirable for most workload. Instead, tell Kubernetes that anti-affinity is Preferred, but if the condition cannot be met, schedule it onto a host with another pod anyways. Use `preferredDuringSchedulingIgnoredDuringExecution` block instead.

{% code title="k8s/deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld
  labels:
    app: helloworld
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
      - name: helloworld
        image: gcr.io/.../helloworld
      affinity:
        podAntiAffinity:
          # Use Preferred rather than Required
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - helloworld
              topologyKey: "kubernetes.io/hostname"
```
{% endcode %}

## Disruption Budget

