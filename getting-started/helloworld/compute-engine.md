# Compute Engine

## Getting Started

### Clone

```text
git clone https://github.com/jamesward/hello-springboot-mvn
cd hello-springboot-mvn
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
gcloud compute instances create helloworld
```

### Copy File to VM

```bash
gcloud compute scp target/spring-boot-example-0.1.0.jar helloworld:
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
java -jar spring-boot-example-0.1.0.jar
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

#### External IP Address

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





