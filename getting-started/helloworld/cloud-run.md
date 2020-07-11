# Cloud Run

Click to Deploy

[![Deploy a Spring Boot app on Cloud Run](https://deploy.cloud.run/button.svg)](https://deploy.cloud.run/?git_repo=https://github.com/jamesward/hello-springboot-mvn.git)

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

