# Getting Started

This section requires basic understanding of Kubernetes - make sure you do the tutorial in sequence.

{% page-ref page="../kubernetes/" %}

## Install Istio

First, make sure you already have a [Kubernetes cluster](../kubernetes/kubernetes-cluster.md) up and running.

### Install istioctl

You need to get the `istioctl` CLI to install Istio into the cluster.

```bash
cd $HOME

# Specify an Istio version to install
curl -L https://istio.io/downloadIstio | \
  ISTIO_VERSION=1.7.4 sh -
```

Add Istio's `bin` path to shell's `PATH.`

```bash
echo 'export PATH="$PATH:$HOME/istio-1.7.4/bin"' >> ~/.bash_profile

source $HOME/.bash_profile
```

Verify `istioctl` is installed properly and with the correct version:

```bash
istioctl version
```

### Install Istio

Install the `demo` profile of Istio, which comes with the basic settings for most of the things you'll want to learn about. In addition, because the cluster in this guide enabled Network Policy, so we can use Istio with Container Network Interface \(CNI\).

```bash
istioctl install \
  --set profile=demo \
  --set values.cni.cniBinDir=/home/kubernetes/bin \
  --set components.cni.enabled=true \
  --set components.cni.namespace=kube-system
```

Validate that Istio is installed. `istioctl version` should show you the `control plane version`.

```bash
istioctl version
```

In addition, Istio is installed into the `istio-system` namespace.

```bash
kubectl get ns

kubectl -n=istio-system get pods
```

## Install Addons

In addition to core-Istio, you can install addons for observability, for example to see distributed traces and out-of-the-box metrics/dashboard.

```bash
# Zipkin
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/extras/zipkin.yaml

# Prometheus
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/prometheus.yaml

# Grafana
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/grafana.yaml

# Wait for Grafana a bit before Kiali
sleep 20

# Kiali
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/kiali.yaml
```

