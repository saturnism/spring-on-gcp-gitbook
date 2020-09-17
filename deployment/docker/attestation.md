# Attestation

To secure your software supply chain, you should consider signing your container images with attestations. Runtime environments like Kubernetes Engine can validate the signature and run only the container images that you have signed/attested with Binary Auth.

## Enable API

```bash
gcloud services enable container.googleapis.com
gcloud services enable containeranalysis.googleapis.com
gcloud services enable binaryauthorization.googleapis.com
```

## Attestor

You need to create an Attestor, which is associated with the metadata of the an asymetric key pair that's used to sign and validate a signature for an image digest.

### Create a Note

A note is a metadata entry in Google Container Analysis and is required when associating with an Attestor. An Attestation ultimately becomes an instance of a Note. 

```bash
PROJECT_ID=$(gcloud config get-value project)
cat > $HOME/attestor-note.json << EOF
{
  "name": "projects/${PROJECT_ID}/notes/default-attestor",
  "attestation": {
    "hint": {
      "human_readable_name": "Default Container Image Attestor"
    }
  }
}
EOF
```

Post the Note to Container Analysis service:

```bash
PROJECT_ID=$(gcloud config get-value project)
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $(gcloud auth print-access-token)"  \
  -H "x-goog-user-project: $PROJECT_ID" \
  --data-binary @$HOME/attestor-note.json\
  https://containeranalysis.googleapis.com/v1/projects/$PROJECT_ID/notes/?noteId=default-attestor
```

### Create an Attestor

```bash
PROJECT_ID=$(gcloud config get-value project)

gcloud beta container binauthz attestors create default-attestor \
    --attestation-authority-note=default-attestor \
    --attestation-authority-note-project=$PROJECT_ID
```

## Asymetric Key Pair

You need to create a key pair so that you can sign an attestation with a private key, and later, verify it with a public key. You can create your own key pair, but this guide will use Cloud KMS.

### Enable API

```bash
gcloud services enable cloudkms.googleapis.com
```

### Create a Keyring

```bash
gcloud kms keyrings create attestor-keyring --location global
```

### Create a Key

```bash
gcloud kms keys create default-attestor-key \
  --location=global \
  --keyring=attestor-keyring  \
  --purpose=asymmetric-signing  \
  --default-algorithm=ec-sign-p256-sha256
```

### Add Key to Attestor

```bash
PROJECT_ID=$(gcloud config get-value project)

gcloud alpha container binauthz attestors public-keys add \
  --attestor=default-attestor \
  --keyversion-project=$PROJECT_ID \
  --keyversion-location=global \
  --keyversion-keyring=attestor-keyring \
  --keyversion-key=default-attestor-key \
  --keyversion=1
```

## Attestation

You can create an attestation for a container image, but you'll need the full SHA256 container image digest. The easiest way to find this is from Container Registry:

```bash
PROJECT_ID=$(gcloud config get-value project)

gcloud container images describe gcr.io/$PROJECT_ID/helloworld
```

### Create an Attestation

```bash
PROJECT_ID=$(gcloud config get-value project)
IMAGE=$(gcloud container images describe gcr.io/$PROJECT_ID/helloworld \
  --format='value(image_summary.fully_qualified_digest)')

gcloud beta container binauthz attestations sign-and-create \
    --artifact-url=$IMAGE \
    --attestor=default-attestor \
    --attestor-project=$PROJECT_ID \
    --keyversion-project=$PROJECT_ID \
    --keyversion-location=global \
    --keyversion-keyring=attestor-keyring \
    --keyversion-key=default-attestor-key \
    --keyversion=1
```

### List Attestations

Once created, you can see the attestation:

```bash
PROJECT_ID=$(gcloud config get-value project)
IMAGE=$(gcloud container images describe gcr.io/$PROJECT_ID/helloworld \
  --format='value(image_summary.fully_qualified_digest)')
  
gcloud beta container binauthz attestations list \
  --artifact-url=$IMAGE \
  --attestor=default-attestor
```

## Binary Authorization

Once the container image has a signed attestation, it can then be used to authorize deployments into a Kubernetes Engine cluster by enabling Binary Authorization.

1. [Create a Kubernetes Engine cluster](../kubernetes/kubernetes-cluster.md#create-cluster) that has Binary Authorization enabled.
2. [Enable Binary Authorization](../kubernetes/binary-authorization.md#enforce-attestation) policy to enforce attestations.

{% hint style="info" %}
See [Binary Authorization](../kubernetes/binary-authorization.md) section for more information.
{% endhint %}
