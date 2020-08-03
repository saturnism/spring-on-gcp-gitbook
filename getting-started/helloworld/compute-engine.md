# Compute Engine

## Getting Started

### Clone

```text
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

### Build

```text
./mvnw package
```

### Enable API

```bash
# To use Compute Engine
gcloud services enable compute.googleapis.com
```

### Create a VM

```bash
gcloud compute instances create helloworld \
  --scopes=cloud-platform
```

{% hint style="info" %}
If you want to use a specific distribution, such as Debian 10, you can add additional parameters:

```bash
gcloud compute instances create helloworld \
  --image-family debian-10 --image-project debian-cloud
```
{% endhint %}

### Copy File to VM

```bash
gcloud compute scp target/helloworld.jar helloworld:
```

### SSH to VM

```bash
gcloud compute ssh helloworld
```

### Install OpenJDK in the VM

```bash
sudo apt-get update && sudo apt-get install -y openjdk-11-jdk
```

### Run in the VM

```text
java -jar helloworld.jar
```

### Expose

#### Firewall

By default, most ports on the Compute Engine are firewalled off.  If you want to expose port `8080` in this case, you can first add a `tag` to the Compute Engine instance, and then add a firewall rule to allow inbound port `8080` traffic for any Compute Engine instance with a certain tag.

Add a tag:

```text
gcloud compute instances add-tags helloworld --tags=webapp
```

Add Firewall rule:

```bash
gcloud compute firewall-rules create webapp-rule \
  --source-ranges=0.0.0.0/0 \
  --target-tags=webapp \
  --allow=tcp:8080
```

### Connect

Find the external IP address of the Compute Engine VM instance:

```bash
gcloud compute instances list
```

You can now connect to the external IP on port `8080` of the application:

```bash
EXTERNAL_IP=$(gcloud compute instances describe helloworld \
  --format='value(networkInterfaces.accessConfigs[0].natIP)')
curl http://${EXTERNAL_IP}:8080
```

{% hint style="info" %}
In production environments, you would most likely want to put a Load Balancer in front, either with a [Network \(L4\) Load Balancer](https://cloud.google.com/load-balancing/docs/network/setting-up-network), or a [HTTP \(L7\) Load Balancer](https://cloud.google.com/load-balancing/docs/https/ext-http-lb-simple).
{% endhint %}

## Getting Started - Container in Compute Engine

### Clone

```text
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

### Build

```text
./mvnw package
```

### Enable API

```bash
# To use Compute Engine
gcloud services enable compute.googleapis.com
```

### Containerize

{% tabs %}
{% tab title="Jib" %}
[Jib](https://github.com/GoogleContainerTools/jib) can containerize any Java application easily, without a `Dockerfile` nor `docker` installed. Jib will push the container image directly to the remote registry.

```bash
PROJECT_ID=$(gcloud config get-value project)
./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
You can configure [Jib Maven plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin) or [Jib Gradle plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-gradle-plugin) directly in the build file to run the Jib easier, such as `./mvnw jib:build`.
{% endhint %}
{% endtab %}

{% tab title="Buildpack" %}
[Cloud Native Buildpacks](https://buildpacks.io) can containerize applications written in different language without a `Dockerfile`. It does require `docker` installed.

1. Install Docker locally - see [Get Docker documentation](https://docs.docker.com/get-docker/).
2. Install `pack` CLI - see [Installing `pack` documentation](https://buildpacks.io/docs/install-pack/)
3. Build container with `pack`, and use `--publish` flag to push directly to the remote registry:

```bash
PROJECT_ID=$(gcloud config get-value project)
pack build --builder gcr.io/buildpacks/builder:v1 \
  --publish \
  gcr.io/${PROJECT_ID}/helloworld
```
{% endtab %}

{% tab title="Spring Boot 2.3" %}
Since Spring Boot 2.3+, you can build container using the Spring Boot plugin.

```bash
PROJECT_ID=$(gcloud config get-value project)

# Maven with Paketo Buildpack
./mvnw spring-boot:build-image \
  -Dspring-boot.build-image.imageName=gcr.io/${PROJECT_ID}/helloworld
  
# Maven with GCP Buildpack
./mvnw spring-boot:build-image \
  -Dspring-boot.build-image.imageName=gcr.io/${PROJECT_ID}/helloworld \
  -Dspring-boot.build-image.builder=gcr.io/buildpacks/builder
  
# Gradle with Paketo Buildpack
./gradlew bootBuildImage --imageName=gcr.io/${PROJECT_ID}/helloworld

# Gradle with GCP Buildpack
./gradlew bootBuildImage --imageName=gcr.io/${PROJECT_ID}/helloworld \
  --builder=gcr.io/buildpacks/builder
```
{% endtab %}
{% endtabs %}

### Create a VM with Container Image

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud compute instances create-with-container \
  helloworld-with-container \
  --container-image=gcr.io/${PROJECT_ID}/helloworld \
  --scopes=cloud-platform
```

{% hint style="info" %}
This will automatically create a Container-Optimized VM, and start the container on VM startup.
{% endhint %}

### Expose

#### Firewall

By default, most ports on the Compute Engine are firewalled off.  If you want to expose port `8080` in this case, you can first add a `tag` to the Compute Engine instance, and then add a firewall rule to allow inbound port `8080` traffic for any Compute Engine instance with a certain tag.

Add a tag:

```bash
gcloud compute instances add-tags \
  helloworld-with-container --tags=webapp
```

Add Firewall rule:

```bash
gcloud compute firewall-rules create webapp-rule \
  --source-ranges=0.0.0.0/0 \
  --target-tags=webapp \
  --allow=tcp:8080
```

### Connect

Find the external IP address of the Compute Engine VM instance:

```bash
gcloud compute instances list
```

You can now connect to the external IP on port `8080` of the application:

```bash
EXTERNAL_IP=$(gcloud compute instances describe helloworld-with-container \
  --format='value(networkInterfaces.accessConfigs[0].natIP)')
curl http://${EXTERNAL_IP}:8080
```

{% hint style="info" %}
In production environments, you would most likely want to put a Load Balancer in front, either with a [Network \(L4\) Load Balancer](https://cloud.google.com/load-balancing/docs/network/setting-up-network), or a [HTTP \(L7\) Load Balancer](https://cloud.google.com/load-balancing/docs/https/ext-http-lb-simple).
{% endhint %}

