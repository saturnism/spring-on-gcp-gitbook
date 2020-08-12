# Health Checks

his section continues from the previous section - make sure you do the tutorial in sequence.

{% page-ref page="service.md" %}

## Spring Boot Actuator

[Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/2.3.0.BUILD-SNAPSHOT/reference/html/production-ready-features.html#production-ready-enabling) can provide some basic health checking mechanisms via the `/actuator/health` endpoint. However, which endpoint you use depends on the Spring Boot version.

Spring Boot 2.3 and above, [Spring Boot Actuator has dedicated support for Liveness Probe](https://docs.spring.io/spring-boot/docs/2.3.0.BUILD-SNAPSHOT/reference/html/production-ready-features.html#production-ready-kubernetes-probes).

Spring Boot &lt; 2.3 and below, it's best to create a simple endpoint that simply returns HTTP `200` response status instead of using the Spring Boot Actuator's `/actuator/health` endpoint. This is because `/actuator/health` by default may fail if an external dependency fails.

| Spring Boot Version | Liveness Probe | Readiness Probe |
| :--- | :--- | :--- |
| &gt;= 2.3 | /actuator/health/liveness | /actuator/health/readiness |
| &lt; 2.3 | Any endpoint that simply returns `200` | /actuator/health |

### Clone

```text
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

### Add Dependencies

```markup
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    
    ...
  </dependencies>
  ...
</project>
```

### Build

```text
./mvnw package
```

### Containerize

Use Jib to containerize the application:

```bash
PROJECT_ID=$(gcloud config get-value project)

./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
Learn different ways to containerize a Java application in the [Container Image](../docker/container-image.md) section.
{% endhint %}

## Liveness Probe

Kubernetes can automatically detect application issues using a Liveness Probe. When the Liveness Probe check fails, Kubernetes will automatically restart the container, hoping that restarting your application will help it to recover. If the container continues to fail the Liveness Probe, Kubernetes will go into a Crash Loop and backs off the restart exponentially.

{% hint style="info" %}
Liveness Probe failure indicates to Kubernetes that the failure can be recovered after a restart.
{% endhint %}

{% hint style="danger" %}
If your Liveness Probe checks an endpoint that fails due to an external dependency, but the external dependency cannot recover simply because your container restarts, then it's not a good check! This type of checks may cause catastropic cascading failures.
{% endhint %}

{% code title="k8s/deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: helloworld
  name: helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - image: gcr.io/.../helloworld
        name: helloworld
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 256Mi
        # Configure the liveness probe
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 10
```
{% endcode %}

## Readiness Probe



