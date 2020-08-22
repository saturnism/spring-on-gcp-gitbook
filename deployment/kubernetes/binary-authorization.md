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



