# App Engine

App Engine is a fully managed Platform-as-a-Service that can run your application, provision a HTTPS load balancer, and scale out the workload as needed. When no one is using your application, it can scale down to zero.

## Getting Started

### Clone

```text
git clone https://github.com/GoogleCloudPlatform/java-docs-samples
cd java-docs-samples/appengine-java11/springboot-helloworld
```

### Build

```text
./mvnw package
```

### Deploy

```text
gcloud app deploy target/springboot-helloworld-j11-0.0.1-SNAPSHOT.jar
```

Once deployed, the command will output the HTTPS URL, and you can open it up in a browser. Or, simply run:

```text
gcloud app browse
```

{% hint style="info" %}
You can run any Java service in App Engine as long as it's packaged as a JAR file, and can be executed with `java -jar app.jar`.
{% endhint %}

## Additional Configuration

By default, App Engine will deploy with the smallest `F1`instance class. You can specify a larger instance, configure environment variables, and more tuning parameters using an `app.yaml`:  

{% code title="app.yaml" %}
```text
runtime: java11
instance_class: F4
env_variables:
  GREETING: "Hola!"
```
{% endcode %}

Deploy the JAR file with the configuration:

```text
gcloud app deploy target/springboot-helloworld-j11-0.0.1-SNAPSHOT.jar \
  --appyaml app.yaml
```

{% hint style="info" %}
Learn more about the configurations in [app.yaml reference documentation](https://cloud.google.com/appengine/docs/standard/java11/config/appref).
{% endhint %}

## Learn More

* [App Engine Java 11 documentation](https://cloud.google.com/appengine/docs/standard/java11)

