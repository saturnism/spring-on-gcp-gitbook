# Metrics

## Cloud Monitoring

Cloud Monitoring provides visibility into the performance, uptime, and overall health of your applications. It can collect metrics, events, and metadata from Google Cloud, Amazon Web Services, hosted uptime probes, application instrumentation, Metrics data can be used to generate insights via dashboards, charts, and alerts. Cloud Monitoring alerting helps you collaborate by integrating with Slack, PagerDuty, and more.

### Enable API

```bash
gcloud services enable monitoring.googleapis.com
```

## Micrometer

[Micrometer](http://micrometer.io/) is the de-facto metrics collector for Spring Boot applications. Micrometer can export JVM metrics, Spring Boot metrics, and also application metrics with [Counters](http://micrometer.io/docs/concepts#_counters), [Gauges](http://micrometer.io/docs/concepts#_gauges), and [Timers](http://micrometer.io/docs/concepts#_timers). You can export the metrics to Cloud Monitoring with two methods:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Method</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">When to use?</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><a href="https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-metrics-export-prometheus">Prometheus</a>
      </td>
      <td style="text-align:left">
        <p>Export metrics using the Prometheus format from a Spring Boot Actuator
          endpoint (<code>/actuator/prometheus</code>).</p>
        <p></p>
        <p>A Prometheus agent will need to be configured to scrape the metrics.</p>
      </td>
      <td style="text-align:left">Great option when running in Kubernetes, where metrics are usually collected
        using Prometheus operator.</td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-metrics-export-stackdriver">Cloud Monitoring API</a>
      </td>
      <td style="text-align:left">Export the metrics directly to Cloud Monitoring using the API.</td>
      <td
      style="text-align:left">This is needed whenever Prometheus scraping is not possible, such as Serverless
        environments like Cloud Run and App Engine.</td>
    </tr>
  </tbody>
</table>

### Prometheus

#### Dependency

Add Spring Boot Actuator and Micrometer Prometheus dependencies:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash

compile group: 'org.springframework.boot', name: 'spring-boot-starter-actuator'
compile group: 'io.micrometer', name: 'micrometer-registry-prometheus'
```
{% endtab %}
{% endtabs %}

#### Configuration

Configure Spring Boot Actuator to expose the Prometheus endpoint:

{% code title="application.properties" %}
```text
management.endpoints.web.exposure.include=health,info,prometheus
```
{% endcode %}

{% hint style="info" %}
Notice that there is no explicit configuration for username/password. Cloud Trace authentication uses the GCP credential \(either your user credential, or Service Account credential\), and authorization is configured via Identity Access Management \(IAM\).
{% endhint %}

#### Prometheus Endpoint

You should be able to access the metrics in Prometheus format from `/actuator/prometheus`.

```text
$ curl http://localhost:8080/actuator/prometheus

# HELP jvm_memory_committed_bytes The amount of memory in bytes that is committed for  the Java virtual machine to use
# TYPE jvm_memory_committed_bytes gauge
jvm_memory_committed_bytes{area="nonheap",id="Code Cache",} 1.8284544E7
jvm_memory_committed_bytes{area="nonheap",id="Metaspace",} 6.6281472E7
jvm_memory_committed_bytes{area="nonheap",id="Compressed Class Space",} 8609792.0
jvm_memory_committed_bytes{area="heap",id="PS Eden Space",} 6.01358336E8
jvm_memory_committed_bytes{area="heap",id="PS Survivor Space",} 2.2020096E7
jvm_memory_committed_bytes{area="heap",id="PS Old Gen",} 1.1010048E8
# HELP tomcat_global_sent_bytes_total  
...
```

#### Prometheus Scraper

If you are running in Kubernetes Engine, you can use Prometheus Operator to install a Prometheus instance. You also need to configure Prometheus with a sidecar that can propagate Prometheus metrics to Cloud Monitoring.

#### Install Prometheus Operator

```text
kubectl apply -f \
  https://raw.githubusercontent.com/coreos/prometheus-operator/v0.38.1/bundle.yaml
```

#### Provisioning Prometheus server with Sidecar

Create a `prometheus.yaml` for the Prometheus Operator, but replace the variables for `${RPOJECT_ID},` `${LOCATION}`, and `${CLUSTER_NAME}`with your Kubernetes Engine cluster information.

{% code title="prometheus.yaml" %}
```yaml
# This config is cooked based on following resource:
# https://godoc.org/github.com/coreos/prometheus-operator/pkg/apis/monitoring/v1#Prometheus
# https://github.com/istio/installer/blob/master/istio-telemetry/prometheus-operator/templates/prometheus.yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
  labels:
spec:
  image: "docker.io/prom/prometheus:v2.12.0"
  version: v2.12.0
  retention: 720h
  scrapeInterval: 15s
  serviceAccountName: prometheus
  serviceMonitorSelector:
    any: true
  serviceMonitorNamespaceSelector:
    any: true
  podMonitorSelector:
    any: true
  podMonitorNamespaceSelector:
    any: true
  enableAdminAPI: false
  podMetadata:
    labels:
      app: prometheus
  containers:
  - name: sd-sidecar
    image: gcr.io/stackdriver-prometheus/stackdriver-prometheus-sidecar:0.7.3
    args:
    - --stackdriver.project-id=${PROJECT_ID}
    - --stackdriver.kubernetes.location=${LOCATION}
    - --stackdriver.kubernetes.cluster-name=${CLUSTER_NAME}
    - --prometheus.wal-directory=/prometheus/wal
    ports:
    - name: sidecar
      containerPort: 9091
    volumeMounts:
    - name: prometheus-prometheus-db
      mountPath: /prometheus
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
  labels:
    app: prometheus
rules:
  - apiGroups: [""]
    resources:
      - nodes
      - services
      - endpoints
      - pods
      - nodes/proxy
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources:
      - configmaps
    verbs: ["get"]
  - nonResourceURLs: ["/metrics"]
    verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus-default
  labels:
    app: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
  - kind: ServiceAccount
    name: prometheus
    namespace: default
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  annotations:
    prometheus.io/scrape: 'true'
  labels:
    app: prometheus
spec:
  selector:
    app: prometheus
  ports:
    - name: http-prometheus
      protocol: TCP
      port: 9090
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: default
  labels:
    app: prometheus
```
{% endcode %}

Apply `prometheus.yaml`to the Kubernetes cluster to provision a new instance using the Prometheus Operator.

```text
kubectl apply -f prometheus.yaml
```

Configure Prometheus server to scrape the metrics. Create a `pod-monitors.yaml`:

{% code title="pod-monitors.yaml" %}
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      app: order-service
  podMetricsEndpoints:
  - targetPort: 8080 
    path: /actuator/prometheus
    interval: 15s
```
{% endcode %}

### Cloud Monitoring API

To export metrics directly to Cloud Monitoring, you can use the [Micrometer Stackdriver registry](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-metrics-export-stackdriver).

#### Dependency

Add Spring Boot Actuator and Micrometer Stackdriver dependencies:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-stackdriver</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash

compile group: 'org.springframework.boot', name: 'spring-boot-starter-actuator'
compile group: 'io.micrometer', name: 'micrometer-registry-stackdriver'
```
{% endtab %}
{% endtabs %}

#### Configuration

Configure the Project ID.

{% code title="application.properties" %}
```text
management.metrics.export.stackdriver.project-id=<PROJECT_ID>
```
{% endcode %}

{% hint style="warning" %}
Better integration is [coming soon with Spring Cloud GCP](https://cloud.spring.io/spring-cloud-gcp/reference/html/#stackdriver-monitoring). 
{% endhint %}

