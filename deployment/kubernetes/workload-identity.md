# Workload Identity

Workload Identity allows you to assign a specific Google Cloud Service Account to a specific application, so that each application can get its own service account identity/permissions using Machine Credential.

## Create Service Accounts

### Create a Kubernetes Service Account \(KSA\)

```bash
kubectl create serviceaccount helloworld \
  --dry-run -oyaml > k8s/helloworld-sa.yaml

kubectl apply -f k8s/helloworld-sa.yaml
```

### Create a Google Cloud Service Account \(GSA\)

```bash
gcloud iam service-accounts create helloworld

PROJECT_ID=$(gcloud config get-value project)
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
  --member serviceAccount:helloworld@${PROJECT_ID}.iam.gserviceaccount.com \
  --role roles/pubsub.publisher
```

## Bind Service Accounts

Bind the Kubernetes Service Account \(KSA\) to Google Cloud Service Account \(GSA\)

### Binding from Google Cloud

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud iam service-accounts add-iam-policy-binding \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:${PROJECT_ID}.svc.id.goog[default/helloworld]" \
  helloworld@${PROJECT_ID}.iam.gserviceaccount.com
```

### Binding from Kubernetes

```bash
PROJECT_ID=$(gcloud config get-value project)
kubectl annotate -f k8s/helloworld-sa.yaml \
  iam.gke.io/gcp-service-account=helloworld@${PROJECT_ID}.iam.gserviceaccount.com
  
kubectl apply -f k8s/helloworld-sa.yaml
```

## Use the Kubernetes Service Account

{% code title="k8s/nginx-sa-deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-sa
  labels:
    app: nginx-sa
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-sa
  template:
    metadata:
      labels:
        app: nginx-sa
    spec:
      # Specify the KSA to use
      serviceAccountName: helloworld
      containers:
      - image: nginx
        name: nginx
```
{% endcode %}

To try it out, first `exec` into the Pod:

```bash
POD_NAME=$(kubectl get pods -lapp=nginx-sa -o jsonpath='{.items[0].metadata.name}')

kubectl exec -ti ${POD_NAME} -- /bin/bash
```

Inside the Pod, see metadata server:

```bash
curl -H"Metadata-Flavor: Google" \
  http://metadata/computeMetadata/v1/instance/service-accounts/default/email
```

