---
description: Create a VM then deploy your application to the VM.
---

# Compute Engine

## Getting Started

### Clone

```bash
cd $HOME
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

### Build

```bash
./mvnw package
```

### Create a VM

#### Enable API

```bash
gcloud services enable compute.googleapis.com
```

#### Create a VM

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

{% hint style="info" %}
See [Compute Engine Machine Types documentation](https://cloud.google.com/compute/docs/machine-types) for a list of Machine Types and the associated CPU/Memory resources.
{% endhint %}

### Copy File to VM

```bash
gcloud compute scp target/helloworld.jar helloworld:
```

{% hint style="info" %}
If this is your first time connecting to the VM, it will automatically prompt you to generate a new SSH key.
{% endhint %}

### SSH to VM

```bash
gcloud compute ssh helloworld
```

### Install OpenJDK in the VM

```bash
sudo apt-get update && sudo apt-get install -y openjdk-11-jdk
```

### Run in the VM

```bash
java -jar helloworld.jar
```

### Expose

#### Firewall

By default, most ports on the Compute Engine are firewalled off. If you want to expose port `8080` in this case, you can first add a `tag` to the Compute Engine instance, and then add a firewall rule to allow inbound port `8080` traffic for any Compute Engine instance with a certain tag.

From outside of the VM \(e.g., your computer, or Cloud Shell\):

#### Add Tag

```bash
gcloud compute instances add-tags helloworld --tags=webapp
```

#### Add Firewall Rule

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

```bash
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

### Build

```bash
./mvnw package
```

### Containerize

#### Enable API

Enable Container Registry API to be able to push container images to the Container Registry.

```bash
gcloud services enable containerregistry.googleapis.com
```

#### Jib

Use Jib to containerize the application:

```bash
PROJECT_ID=$(gcloud config get-value project)

./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
Learn different ways to containerize a Java application in the [Container Image](../../deployment/docker/container-image.md) section.
{% endhint %}

### Create a VM with Container Image

#### Enable API

```bash
gcloud services enable compute.googleapis.com
```

#### Create a VM

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

By default, most ports on a Compute Engine instance are firewalled off. If you want to expose port `8080` in this case, you can first add a `tag` to the Compute Engine instance, and then add a firewall rule to allow inbound port `8080` traffic for any Compute Engine instance with a certain tag.

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

{% hint style="info" %}
To deploy a fleet of VMs, you can use [Managed Instance Group](https://cloud.google.com/compute/docs/containers/deploying-containers#managedinstancegroupcontainer) to deploy a set of VMs running the same container image.
{% endhint %}
