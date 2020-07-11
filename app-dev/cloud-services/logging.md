# Logging

## Cloud Logging

Cloud Logging allows you to store, search, analyze, monitor, and alert on logging data and events from Google Cloud runtime environments and also any other on-premises or Cloud environments.

### Enable API

```bash
gcloud services enable logging.googleapis.com
```

{% hint style="info" %}
Logging API is usually enabled by default for your project.
{% endhint %}

## Centralized Logging

There are a couple of ways to send log messages to Google Cloud.

* If you are running in a Kubernetes Engine,  App Engine, Cloud Run, Cloud Functions, then logs to `STDOUT` or `STDERR` are automatically sent to Cloud Logging.
* If you are running in Compute Engine, then you can install a [Logging Agent](https://cloud.google.com/logging/docs/agent/installation).
* If you are running outside of Google Cloud runtime environment, e.g., from on-premise datacenter, or another cloud, you can:
  * Use the Cloud Logging API to send log entries to Cloud Logging
  * Use a [Logging Agent](https://cloud.google.com/logging/docs/agent/installation)
  * Use a [Fluend adapter](https://github.com/GoogleCloudPlatform/google-fluentd)

## Severity Level

Cloud Logging has [9 different log severity levels](https://cloud.google.com/logging/docs/reference/v2/rest/v2/LogEntry#LogSeverity) the log entries can associate with.

However, in all the runtime environments where logs printed `STDOUT` and `STDERR` are sent to Cloud Logging, original log entry's severity level is not retained:

* Log entries printed to `STDOUT` will have a severity level of `INFO` regardless of the original log entry level.
* Log entries printed to `STDERR` will have a severity level of `WARNING` regardless of the original log entry level.

Different runtime environments have different ways of associating the log level properly.

| Environment | Preferred Logging |
| :--- | :--- |
| Cloud Function | [Use Java Logging API \(JUL\)](https://cloud.google.com/functions/docs/concepts/java-logging) |
| App Engine Standard | [Output Structured Logs in JSON format](https://cloud.google.com/logging/docs/structured-logging) |
| Cloud Run | [Output Structured Logs in JSON format](https://cloud.google.com/logging/docs/structured-logging) |
| Compute Engine | [Install Logging Agent](https://cloud.google.com/logging/docs/agent/installation), or use Cloud Logging API |
| Kubernetes Engine | [Output Structured Logs in JSON format](https://cloud.google.com/logging/docs/structured-logging) |

## Logback

Spring Boot uses [Slf4J](http://www.slf4j.org/) logging API and [Logback](http://logback.qos.ch/) logger by default. You can user [Spring Cloud GCP's Logging Starter](https://cloud.spring.io/spring-cloud-static/spring-cloud-gcp/1.2.3.RELEASE/reference/html/#stackdriver-logging) to use pre-configured Logback appenders to produce Structured JSON logs, or send the log via the Cloud Logging API.

### Dependency

Add the Spring Cloud GCP Trace starter:

{% tabs %}
{% tab title="Maven" %}
```bash
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-logging</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Gradle" %}
```bash

compile group: 'org.springframework.cloud', name: 'spring-cloud-gcp-starter-logging'
```
{% endtab %}
{% endtabs %}

### Configuration

Configure Logback to use the additional appenders, by adding a `logback-spring.xml` file, and import the appender configuration:

{% code title="logback-spring.xml" %}
```markup
<configuration>
  <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
  <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>
  <include resource="org/springframework/cloud/gcp/logging/logback-appender.xml"/>

  ...
</configuration>
```
{% endcode %}

### Log with Cloud Logging API

{% code title="logback-spring.xml" %}
```markup
<configuration>
  <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
  <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>
  <include resource="org/springframework/cloud/gcp/logging/logback-appender.xml"/>

  <root level="INFO">
    <appender-ref ref="CONSOLE"/>
    <appender-ref ref="STACKDRIVER"/>
  </root>
</configuration>
```
{% endcode %}

{% hint style="info" %}
Notice that there is no explicit configuration for username/password. Cloud Logging authentication uses the GCP credential \(either your user credential, or Service Account credential\), and authorization is configured via Identity Access Management \(IAM\).
{% endhint %}

### Log with Structured JSON Logging

{% code title="logback-spring.xml" %}
```markup
<configuration>
  <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
  <include resource="org/springframework/cloud/gcp/logging/logback-json-appender.xml"/>

  <root level="INFO">
    <appender-ref ref="CONSOLE_JSON"/>
  </root>
</configuration>
```
{% endcode %}

#### Use Different Appenders with Profile

It's useful to configure different appenders when running in [Spring Boot profiles](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-profiles). For example, in local/dev environments, simply output regular log entries to `STDOUT`, in staging/production environments, use Structured JSON Logging.

{% hint style="info" %}
See [Spring Boot Logging documentation](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#profile-specific-configuration) and Spring Boot profiles for more details.
{% endhint %}

For example, to configure default profile to use regular logging, and higher environments with Structured JSON Logging:

{% code title="logback-spring.xml" %}
```markup
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <include resource="org/springframework/boot/logging/logback/console-appender.xml" />

    <springProfile name="qa | staging | prod">
        <include resource="org/springframework/cloud/gcp/logging/logback-json-appender.xml"/>
        <root level="INFO">
            <appender-ref ref="CONSOLE_JSON"/>
        </root>
    </springProfile>
    <springProfile name="default | dev">
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
</configuration>
```
{% endcode %}

This sample application allows you to:

* If no profile is specified, then the `default` profile is used, then use the default `CONSOLE` appender.
* If you specify `dev` profile, then use the `CONSOLE` appender
* If you specify `qa`, `staging`, `prod` profile, then it'll output to Structured JSON Logging.

Alternatively, you can also mix and match the profiles with more generic profiles:

{% code title="logback-spring.xml" %}
```markup
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <include resource="org/springframework/boot/logging/logback/console-appender.xml" />

    <springProfile name="logging-json">
        <include resource="org/springframework/cloud/gcp/logging/logback-json-appender.xml"/>
        <root level="INFO">
            <appender-ref ref="CONSOLE_JSON"/>
        </root>
    </springProfile>
    <springProfile name="logging-api">
        <include resource="org/springframework/cloud/gcp/logging/logback-appender.xml"/>
        <root level="INFO">
            <appender-ref ref="STACKDRIVER"/>
        </root>
    </springProfile>
    <springProfile name="logging-console | default">
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
</configuration>
```
{% endcode %}

This sample application allows you to:

* If no profile is specified, then the `default` profile is used, then use the default `CONSOLE` appender.
* If you specify `logging-json` profile, it'll output to Structured JSON Logging.
* If you specify `logging-api` profile, it'll send the logs via the API.
* If you speicfy `default` and `logging-api` profiles, then it'll use the default `CONSOLE` appender and send the logs via the API.

### Log / Trace Correlation

When using Structured JSON Logging or logging using the API, then Spring Cloud Sleuth's trace context \(Trace ID, Span ID\) are automatically added to the log metadata. If you explore the log message in the Cloud Logging Console, you can see the `trace` attribute and the `spanId` attribute are both populated with the correct values:

![](https://lh3.googleusercontent.com/4Z0u20hq8WiwuueOU-DqDG58hqbs2m6IG3jCOZpmrNTu8vJjN8sfcjBbbhiDfmQI1MZf_IeJ9x8tnLSUapoiR8kM5M8fqu7avXucQ4JgU3FoWEWu_NbzL8nd1l7kbXdfzqJkiAYfeA)

You can then see the logs alongside the trace itself:

![](https://lh3.googleusercontent.com/O6u214GgMO_GD-xNUkHVj8KTOBH6pf8-_SJP1x17QhdT9Fle3D30gjV-wuTOSSYDHWnjMqFyZmymAIroBTrxNRJGXrT6JqWRQYGVyZE0DMXRDCR4IkNxBCoAwKGnzyctcJMk7-PPBQ)

### Samples

* [Spring Cloud GCP Logging sample](https://github.com/spring-cloud/spring-cloud-gcp/tree/master/spring-cloud-gcp-samples/spring-cloud-gcp-logging-sample)

## Other Loggers

It's highly recommended that you use the default logger \(Logback\) with Spring Boot, to take advantage of Spring Cloud GCP features.  If you do use other Loggers, you may be able to configure logging to API with different appenders/handlers.

#### Java Logging API \(JUL\)

See [Cloud Logging handler for Java Logging API](https://cloud.google.com/logging/docs/setup/java#the_javautillogging_handler).

#### Apache Commons Logging \(JCL\)

There is no ready-to-use appender to Cloud Logging. But you can [bridge it to Slf4J](http://www.slf4j.org/legacy.html), or [bridge it to Java Logging API](http://commons.apache.org/proper/commons-logging/apidocs/org/apache/commons/logging/impl/Jdk14Logger.html). 

#### Log4J 2

There is no ready-to-use appender to Cloud Logging. But you can [bridge it to Slf4J](https://logging.apache.org/log4j/log4j-2.2/log4j-to-slf4j/index.html).

