---
description: >-
  Learn how to create a production-grade Kubernetes cluster to deploy your
  application.
---

# Kubernetes Cluster

This section requires basic understanding of Docker and container images - make sure you do the tutorial in sequence.

{% page-ref page="../docker/container-image.md" %}

## Enable API

```text
# To use Compute Engine
gcloud services enable compute.googleapis.com

# To use Kubernetes Engine
gcloud services enable container.googleapis.com

# To store Container Images in Container Registry
gcloud services enable containerregistry.googleapis.com
```

## Create Cluster

While it's easy to create a Kubernetes Engine cluster, but it takes a bit more to provision a production-grade cluster.   This cluster will enable many features for production use:

| Feature | Description |
| :--- | :--- |
| [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity) | Workload Identity is the recommended way to access Google Cloud services from within GKE, so you can securely associate specific service account to a workload. |
| [VPC Native Cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/alias-ips) | Allow Kubernetes Pod IP addresses to be natively routable on a VPC. Most importantly, it allows one-hop from Google Cloud Load Balancer to the Kubernetes Pod without unnecessary intermediary routing. |
| [Network Policy](https://cloud.google.com/kubernetes-engine/docs/how-to/network-policy) | Network policy enforcement to control the communication between your cluster's Pods and Services. |
| [Cloud Operations](https://cloud.google.com/stackdriver/docs/solutions/gke/installing) | Allows you to monitor your running Google Kubenetes Engine clusters, manage your system and debug logs, and analyze your system's performance using advanced profiling and tracing capabilities. |
| [Binary Authorization](https://cloud.google.com/binary-authorization/docs) | Binary Authorization is a deploy-time security control that ensures only trusted container images are deployed on Google Kubernetes Engine. |
| [Shielded Nodes](https://cloud.google.com/kubernetes-engine/docs/how-to/shielded-gke-nodes) | Shielded GKE Nodes provide strong, verifiable node identity and integrity to increase the security of GKE nodes. |
| [Secure Boot](https://cloud.google.com/security/shielded-cloud/shielded-vm#secure-boot) | Secure Boot helps ensure that the system only runs authentic software by verifying the digital signature of all boot components, and halting the boot process if signature verification fails. |
| [Auto Repair](https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-repair) | Automatically repair a Google Kubernetes Engine node if it becomes unhealthy. |
| [Auto Upgrade](https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-upgrades) | Automatically upgrade Google Kubernetes Engine nodes version to keep up to date with the cluster control plane version. |

```text
PROJECT_ID=$(gcloud config get-value project)
gcloud container clusters create demo-cluster \
  --num-nodes 4 \
  --machine-type n1-standard-4 \
  --network=default \
  --workload-pool=${PROJECT_ID}.svc.id.goog \
  --enable-ip-alias \
  --enable-network-policy \
  --enable-stackdriver-kubernetes \
  --enable-binauthz \
  --enable-shielded-nodes \
  --shielded-secure-boot \
  --enable-autorepair \
  --enable-autoupgrade \
  --scopes=cloud-platform
```

## Credentials

Kubernetes credential is automatically retrieved and stored in your `$HOME/.kube/config` file. If you need to re-retrieve the credential:

```bash
gcloud container clusters get-credentials demo-cluster
```

## Node Pool and Nodes

The Kubernetes cluster is composed of multiple Nodes - each node is a Compute Engine Virtual Machine.  When you deploy a container image into Kubernetes, a container instance is ultimately scheduled and ran on one of the Nodes.

In Kubernetes Engine, theses nodes are managed by a Node Pool, which is a set of homogenous Compute Engine Virtual Machines \(i.e., they have exactly the same configuration, such as machine type, disk, operation system, etc\).

{% hint style="info" %}
You can add different machine types to your Kubernetes Engine cluster, by creating a new Node Pool with the configuration you want.
{% endhint %}

You can see a list of Virtual Machines using `gcloud`:

```bash
gcloud compute instances list
```

You can also use `kubectl` to list the nodes that belongs to the current cluster:

```bash
kubectl get nodes
```

You can also SSH into the node directly if needed, by specifying the name of the node:

```bash
gcloud compute ssh gke-demo-cluster-default-pool-...
```

Once you are in the Compute Engine Virtual Machine, you can also see the containers that are running inside of the Node:

```bash
docker ps
exit
```

