# Cloud Shell

You can run and test a an application directly within Cloud Shell. It's not meant for development only and not meant for long running processes.

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

### Run

{% tabs %}
{% tab title="Plugin" %}
```text
./mvnw spring-boot:run
```
{% endtab %}

{% tab title="JAR" %}
```
java -jar target/helloworld.jar
```
{% endtab %}
{% endtabs %}

### Connect

From Cloud Shell, click **Web Preview**, then click **Preview on port 8080.**

![Web Preview](../../.gitbook/assets/image%20%2837%29.png)

