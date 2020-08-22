# Binary Authorization

This section continues from the previous section - make sure you do the tutorial in sequence.

{% page-ref page="../docker/attestation.md" %}

## Enforce Attestation

Binary Authorization allows you to enforce container image attestation, so that only attested container images can run.

Before you can turn this on, you must have [attested a container image](../docker/attestation.md).

### Enable Policy

First, export the existing Binary Authorization policy:

```bash
gcloud container binauthz policy export > $HOME/binauthz-policy.yaml
```

Edit the `binauthz-policy.yaml` and enable attestation policy:

{% code title="binauthz-policy.yaml" %}
```yaml
admissionWhitelistPatterns:
- namePattern: gcr.io/google_containers/*
- namePattern: gcr.io/google-containers/*
- namePattern: k8s.gcr.io/*
- namePattern: gke.gcr.io/*
- namePattern: gcr.io/stackdriver-agents/*
defaultAdmissionRule:
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  # Change evaluationMode to require attestation
  evaluationMode: REQUIRE_ATTESTATION
  # Add the policy, and reference the `default-attestor` created from
  # Attestation section.
  # Replace PROJECT_ID with your Project ID.
  requireAttestationsBy:
  - projects/PROJECT_ID/attestors/default-attestor
globalPolicyEvaluationMode: ENABLE
name: projects/PROJECT_ID/policy
```
{% endcode %}

Import the Policy File:

```bash
gcloud container binauthz policy import $HOME/binauthz-policy.yaml
```

## Unattested Container Image

You can verify that the policy is being enforced by deploying an unattested container image:

```bash
kubectl create deployment unattested-nginx --image=nginx
```

While this should have created a new deployment for `nginx` and running a Pod, you can validate that no Pod is running:

```bash
kubectl get pods -lapp=unattested-nginx
```

In addition, you can verify the Kubernetes events stream:

```bash
kubectl get event
```

Observe the event where the container image was denied by the attestor:

```text
... Error creating: pods "..." is forbidden: image policy webhook backend denied one or more images: Denied by default admission rule. Denied by Attestor. ...
```

Delete the deployment:

```bash
kubectl delete deployment unattested-nginx
```

## Attested Container Image

Deploy a [previously attested container image](../docker/attestation.md#create-an-attestation) from the [Container Image Attestation](../docker/attestation.md) section.

```bash
PROJECT_ID=$(gcloud config get-value project)
IMAGE=$(gcloud container images describe gcr.io/$PROJECT_ID/helloworld \
  --format='value(image_summary.fully_qualified_digest)')

kubectl create deployment attested-helloworld --image=$IMAGE
```

Verify that the Pod is up and running:

```bash
kubectl get pods -lapp=attested-helloworld
```

## Allow List

It may be impossible to attest every single container image you want to run. For example, you may trust certain images from open source projects. You can add these images into an allow list.

For example, to be able to deploy the `nginx` container image from Dockerhub without attestation, you need to add it to the policy.

First, export the existing Binary Authorization policy:

```bash
gcloud container binauthz policy export > $HOME/binauthz-policy.yaml
```

Edit the `binauthz-policy.yaml` and enable attestation policy:

{% code title="binauthz-policy.yaml" %}
```yaml
admissionWhitelistPatterns:
- namePattern: gcr.io/google_containers/*
- namePattern: gcr.io/google-containers/*
- namePattern: k8s.gcr.io/*
- namePattern: gke.gcr.io/*
- namePattern: gcr.io/stackdriver-agents/*
# Add nginx to the allow list
- namePattern: nginx
defaultAdmissionRule:
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  evaluationMode: REQUIRE_ATTESTATION
  requireAttestationsBy:
  - projects/PROJECT_ID/attestors/default-attestor
globalPolicyEvaluationMode: ENABLE
name: projects/PROJECT_ID/policy
```
{% endcode %}

Import the Policy File:

```bash
gcloud container binauthz policy import $HOME/binauthz-policy.yaml
```

Deploy `nginx` again:

```bash
kubectl create deployment unattested-nginx --image=nginx
```

Verify that the Pod is up and running due to the allow list:

```bash
kubectl get pods -lapp=unattested-nginx
```

If you trust every container image from a particular Project:



{% code title="binauthz-policy.yaml" %}
```yaml
admissionWhitelistPatterns:
- namePattern: gcr.io/google_containers/*
- namePattern: gcr.io/google-containers/*
- namePattern: k8s.gcr.io/*
- namePattern: gke.gcr.io/*
- namePattern: gcr.io/stackdriver-agents/*
# Add the container registry from a project to the allow list.
# Replace PROJECT_ID.
- namePattern: gcr.io/PROJECT_ID/*
defaultAdmissionRule:
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  evaluationMode: REQUIRE_ATTESTATION
  requireAttestationsBy:
  - projects/PROJECT_ID/attestors/default-attestor
globalPolicyEvaluationMode: ENABLE
name: projects/PROJECT_ID/policy
```
{% endcode %}

