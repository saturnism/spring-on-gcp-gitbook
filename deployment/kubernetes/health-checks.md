---
description: >-
  Learn how Kubernets checks the application health, and how to use liveness
  probes and readiness probes.
---

# Health Checks

This section continues from the previous section - make sure you do the tutorial in sequence.

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

{% hint style="info" %}
In addition to `httpGet`, you can also configure different type of probes such as `exec` to execute a command to perform a non-HTTP check, or use `tcpSocket` to simply check if a port is listening. See Kubernetes [Configure Liveness, Readiness, and Startup Probes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup) for more details.
{% endhint %}

{% hint style="info" %}
Notice the additional `initialDelaySeconds` configuration. If your application starts slowly \(e.g., 1 minute to start\), and the `livenessProbe` starts the check early \(e.g., 10 seconds\), then the `livenessProbe` might never succeed - causing the application to always getting restarted.
{% endhint %}

{% hint style="success" %}
When configuring a `livenessProbe`, always consider the initial delay needed for your application.
{% endhint %}

## Readiness Probe

When your application is alive doesn't mean it's ready to receive traffic. For example, during the startup, the application is alive, but it needs to pre-load data, or warmup caches, before it's ready to accept traffic. A Readiness Probe will let Kubernetes know when your application is ready to receive traffic, and only then will the instance be enlisted into the load balancer as a backend to serve requests \(i.e., a Service's Endpoint\).

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
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
        # Configure the readiness probe
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
```
{% endcode %}

{% hint style="success" %}
You should always configure a `readinessProbe`. Even if you don't use Spring Boot Actuator, you can point the probe to `/` or some endpoint that indicates the traffic is ready serve.
{% endhint %}



