# Istio Sidecar Proxy

Istio requires to run a sidecar proxy next to every instance of your containers that needs to participate in the service mesh. There are 2 ways of adding the sidecar proxy:

1. Automatic sidecar injection
2. Manual sidecar injection

## Automatic Sidecar Injection

You can inject the Istio sidecar automatically for every pod that's deployed into a specific namespace. You can enable automatic injection by annotating the namespace you want to use the service mesh.

```bash
kubectl label namespace default istio-injection=enabled
```

Deploy a workload, such as the Helloworld application from the [Kubernetes Deployment](../kubernetes/deployment.md#deployment-yaml) section.

```bash
kubectl apply -f k8s/deployment.yaml
```

Verify that the Helloworld pod has 2 containers rather than only 1:

```bash
kubectl get pods
```

Each container within a pod is named. Now that the pod has multiple containers, you can specify a container within the pod using `-c containername` parameter:

```bash
POD_NAME=$(kubectl get pods -lapp=helloworld -o jsonpath='{.items[0].metadata.name}')

kubectl logs ${POD_NAME} -c helloworld
kubectl logs ${POD_NAME} -c istio-proxy
```

If, for some reason, a workload do not want to participate in the mesh, then you can explicitly turn off automatic sidecar injection using annotation:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld
  ...
spec:
  template:
    metadata:
      labels:
        ...
      annotations:
        # Explicitly turn off automatic sidecar injection
        sidecar.istio.io/inject: "false"
    spec:
      ...

```

## Manual Sidecar Injection

You can use `istioctl` to filter your existing Kubernetes deployment file and it'll produce the enhanced deployment manifest.

```bash
istioctl kube-inject -f k8s/deployment.yaml
```

In addition to your original manifest, the enhanced manifest now has an additional `istio-proxy`container.

You can save the enhanced manifest into a file for future deployments. Or, you can filter and apply in one command:

```bash
istioctl kube-inject -f k8s/deployment.yaml| kubectl apply -f
```

In most cases, Automatic Sidecar Injection is what you need.

