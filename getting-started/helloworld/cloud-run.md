# Cloud Run

Cloud Run is a fully managed container runtime environment, where you can deploy any HTTP serving containers, and Cloud Run will automatically scale out the number of instances as needed, and scale down to zero when no one is using it.

## Getting Started

### Click to Deploy

You can deploy a [Hello World Application](https://github.com/jamesward/hello-springboot-mvn.git) simply by click on the **Run on Google Cloud** button below!

[![Deploy a Spring Boot app on Cloud Run](https://deploy.cloud.run/button.svg)](https://deploy.cloud.run/?git_repo=https://github.com/jamesward/hello-springboot-mvn.git)

### Manual Deployment

#### Clone

```bash
git clone https://github.com/jamesward/hello-springboot-mvn
cd hello-springboot-mvn
```

#### Build

```bash
./mvnw package
```

#### Containerize

{% tabs %}
{% tab title="Jib" %}
```bash
PROJECT_ID=$(gcloud config get-value project)
./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}>/helloworld
```
{% endtab %}

{% tab title="Buildpack" %}
1. Install Docker locally - see [Get Docker documentation](https://docs.docker.com/get-docker/).
2. Install `pack` CLI - see [Installing `pack` documentation](https://buildpacks.io/docs/install-pack/)
3. Build container with `pack`:
4. ```bash
   pack build --builder gcr.io/buildpacks/builder:v1 \
     gcr.io/${PROJECT_ID}/helloworld 
   ```
{% endtab %}
{% endtabs %}

Containerize it with [Jib](https://github.com/GoogleContainerTools/jib):

{% hint style="info" %}
Alternatively, you can declare Jib as a plugin inside the Maven or Gradle build file, and execute the plugin easily.  You can also build the container using [Cloud Native Buildpack](https://buildpacks.io/).
{% endhint %}

#### Deploy

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud run deploy helloworld --platform=managed --allow-unauthenticated \
  --image=gcr.io/${PROJECT_ID}/helloworld
```

## Additional Configurations

By default, Cloud Run will deploy with the smallest 1CPU 256MB instance. You can specify a larger instance, and configure environment variables with the `gcloud` CLI:

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud run deploy helloworld --platform=managed --allow-unauthenticated \
  --cpu=2 --memory=512M --set-env-vars="GREETING=Hola!"
  --image=gcr.io/${PROJECT_ID}/helloworld
```

