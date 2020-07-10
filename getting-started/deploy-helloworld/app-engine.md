# App Engine

Build your Spring Boot JAR file.

```text
./mvnw package
```

Deploy the JAR to App Engine:

```text
gcloud app deploy target/helloworld.jar
```

Additional Configuration

{% code title="app.yaml" %}
```text
runtime: java11
instance_class: F4
env_variables:
  GREETING: "Hola!"
```
{% endcode %}

Deploy with the configuration:

```text
gcloud app deploy target/helloworld.jar --appyaml app.yaml
```

