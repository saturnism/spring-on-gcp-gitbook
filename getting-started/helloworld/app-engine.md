# App Engine

[App Engine](https://cloud.google.com/appengine/docs/standard/java11) is a fully managed Platform-as-a-Service that can run your application, provision a HTTPS load balancer, and scale out the workload as needed. When no one is using your application, it can scale down to zero.

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

### Deploy

```text
gcloud app deploy target/helloworld.jar
```

### Connect

Once deployed, the command will output the HTTPs URL, and you can open it up in a browser. Or, simply run:

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
  SPRING_ACTIVE_PROFILE: "prod"
```
{% endcode %}

Deploy the JAR file with the configuration:

```text
gcloud app deploy target/helloworld.jar \
  --appyaml app.yaml
```

{% hint style="info" %}
Learn more about the configurations in [app.yaml reference documentation](https://cloud.google.com/appengine/docs/standard/java11/config/appref).
{% endhint %}

## Learn More

* [App Engine Java 11 documentation](https://cloud.google.com/appengine/docs/standard/java11)
* [Deploy with App Engine Maven plugin](https://cloud.google.com/appengine/docs/standard/java11/using-maven#setting_up_maven)
* [Deploy with App Engine Gradle plugin](https://cloud.google.com/appengine/docs/standard/java11/using-gradle)

