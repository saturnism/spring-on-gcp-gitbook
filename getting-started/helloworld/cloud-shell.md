---
description: Use Cloud Shell to build and test applications for development purpose.
---

# Cloud Shell

You can run and test a an application directly within Cloud Shell. Cloud Shell has many tools pre-installed, such OpenJDK, Maven, Gradle, and more. Cloud Shell is meant for development and not meant for production or long running processes.

## Getting Started

### Clone

```bash
cd $HOME
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

### Build

```bash
./mvnw package
```

### Run

{% tabs %}
{% tab title="Plugin" %}
```bash
./mvnw spring-boot:run
```
{% endtab %}

{% tab title="JAR" %}
```bash
java -jar target/helloworld.jar
```
{% endtab %}
{% endtabs %}

### Connect

From Cloud Shell, click **Web Preview**, then click **Preview on port 8080.**

![Web Preview](../../.gitbook/assets/image%20%2837%29.png)

