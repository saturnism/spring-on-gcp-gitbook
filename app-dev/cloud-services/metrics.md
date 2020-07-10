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
      <td style="text-align:left"><a href="http://micrometer.io/docs/registry/stackdriver#_spring_boot">Cloud Monitoring API</a>
      </td>
      <td style="text-align:left">Export the metrics directly to Cloud Monitoring using the API.</td>
      <td
      style="text-align:left">This is needed whenever Prometheus scraping is not possible, such as Serverless
        environments like Cloud Run and App Engine.</td>
    </tr>
  </tbody>
</table>

### Prometheus

### Cloud Monitoring API

