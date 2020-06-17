# Secret Management

## Cloud Secret Manager

Secret Manager is a secure and convenient storage system for API keys, passwords, certificates, and other sensitive data. Secret Manager provides a central place and single source of truth to manage, access, and audit secrets across Google Cloud.

### Enable API

```bash
gcloud services enable secretmanager.googleapis.com
```

### Create a Secret

```bash
echo "qwerty" | \
  gcloud secrets create some-api-key --data-file=- --replication-policy=automatic
```

### List Secrets

```bash
gcloud secrets list
```

### Delete a Secret

```bash
gcloud secrets delete some-api-key
```

### Assign IAM Permission

You can finely control CRUD permissions for an account \(user account, service account, a Google Group\) to a secret.  See the [Secret Manager IAM access control](https://cloud.google.com/secret-manager/docs/access-control) for more information. 

```bash
gcloud secrets add-iam-policy-binding --help
```



