# Cloud Run

Build Your Application

```text
./mvnw package
```

Enable APIs

Containerize

```text
./mvnw package jib:build -Dto.image=gcr.io/...
```

Deploy to Cloud Run!

```text
gcloud run deploy helloworld --image=...
```

