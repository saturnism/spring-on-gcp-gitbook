# App Engine

[App Engine](https://cloud.google.com/appengine/docs/standard/java11) is a fully managed Platform-as-a-Service that can run your application, provision a HTTPS load balancer, and scale out the workload as needed. When no one is using your application, it can scale down to zero.

{% embed url="https://www.youtube.com/watch?v=qx\_T6-EKkBE" %}

## Getting Started

### Clone

```text
cd $HOME
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

{% hint style="info" %}
If this is your first time using App Engine on the project, you'll be prompted to choose the region of the App Engine application. Pick the region that's most suitable for your application. This site mostly uses `us-central` as an example.
{% endhint %}

{% hint style="warning" %}
Once you picked the region, you cannot change it for an App Engine application.
{% endhint %}

### Connect

Once deployed, the command will output the HTTPs URL. To open the URL in your browser:

```text
gcloud app browse
```

To find the URL without opening the browser:

```bash
gcloud app browse --no-launch-browser
```

You can `curl` the URL:

```bash
URL=$(gcloud app browse --no-launch-browser)
curl ${URL}
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
  SPRING_PROFILES_ACTIVE: "prod"
```
{% endcode %}

{% hint style="info" %}
See [App Engine Standard Instance Classes documentation](https://cloud.google.com/appengine/docs/standard#instance_classes) for a list of Instance Classes and associated CPU/Memory resources.
{% endhint %}

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

