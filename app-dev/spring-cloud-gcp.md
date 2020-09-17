# Spring Cloud GCP

Spring Cloud GCP contains a set of easy to use starters/autoconfigurations that allow you to easily connect and adopt GCP services.

Spring Cloud GCP is part of the Spring Cloud release train.

{% hint style="info" %}
Even though Spring Cloud GCP is part of the Spring Cloud release train, it doesn't mean that you need to use any Spring Cloud features \(Eureka, Config Server, etc.\). The release train helps manage dependency versions so that you don't need to specify versions. It avoids having incompatible dependency versions, and eliminates dependency conflicts.
{% endhint %}

## Demo

{% embed url="https://www.youtube.com/watch?v=5d\_dy7RVcpE" caption="A short demo video of Spring Cloud GCP" %}

## Configure Spring Boot Project

### New Spring Boot Project

Create a new Spring Boot project use [Spring Initializr](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.2.6.RELEASE&packaging=jar&jvmVersion=1.8&groupId=com.example&artifactId=demo&name=demo&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.demo&dependencies=cloud-gcp,web), and add the [GCP Support dependency](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.2.6.RELEASE&packaging=jar&jvmVersion=1.8&groupId=com.example&artifactId=demo&name=demo&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.demo&dependencies=cloud-gcp,web).

![Add the GCP Support dependency](../.gitbook/assets/image%20%283%29.png)

Or, generate the project using `curl`:

```bash
curl https://start.spring.io/starter.zip \
  -d dependencies=web,cloud-gcp,lombok \
  -d bootVersion=2.3.1.RELEASE \
  -d baseDir=demo
```

Unpack the downloaded zip file:

```bash
unzip demo.zip
```

The generated project will include Spring Cloud release train BOM. 

### Existing Spring Boot Project

If you want to add Spring Cloud GCP to existing project, simply add Spring Cloud release train configuration. Different Spring Boot version is compatible with different Spring Cloud releases.

| Spring Boot Version | Spring Cloud Version |
| :--- | :--- |
| 2.1 | Greenwich |
| 2.2 | Hoxton |
| 2.3 | Hoxton.SR5 |

See [Spring Cloud documentation](https://spring.io/projects/spring-cloud#learn) for the latest versions.

Add the compatible Spring Cloud BOM version to your project:

{% tabs %}
{% tab title="Maven" %}
```markup
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
{% endtab %}

{% tab title="Gradle" %}
```groovy
buildscript {
  dependencies {
    classpath "io.spring.gradle:dependency-management-plugin:1.0.2.RELEASE"
  }
}

apply plugin: "io.spring.dependency-management"

dependencyManagement {
  imports {
    mavenBom 'org.springframework.cloud:spring-cloud-dependencies:Hoxton.RELEASE'
  }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
A BOM is a Bill of Material, when imported, you can specify dependencies managed by the BOM without explicitly specifying a version for that dependency.  Using the Spring Cloud BOM will allow you to use all Google Cloud Client Libraries and Spring Cloud GCP libraries without explicitly specifying a version.
{% endhint %}

## Services

Spring Cloud GCP supports many GCP services using de-facto Spring abstraction layers.

| Concerns | GCP Service | Spring Abstraction |
| :--- | :--- | :--- |
| **Databases** | Cloud SQL | JDBC template |
|  |  | Spring Data JPA |
|  | Cloud Spanner | Spring Data Spanner |
|  |  | Spring Data JPA with Hibernate |
|  | Cloud Datastore | Spring Data Datastore |
|  | Cloud Firestore | Spring Reactive Data Firestore |
| **Messaging** | Cloud Pub/Sub | Pub/Sub Template |
|  |  | Spring Integration |
|  |  | Spring Cloud Stream |
|  |  | Spring Dataflow |
| **Configuration** | Cloud Secret Manager | Spring Cloud Config |
| **Storage** | Cloud Storage | Spring Resource |
| **Cache** | Cloud Memorystore | Spring Data Redis |
| **Distributed Tracing** | Cloud Trace | Spring Cloud Sleuth |
|  |  | Zipkin / Brave |
| **Centralized Logging** | Cloud Logging | SLF4J / Logback |
| **Monitoring Metrics** | Cloud Monitoring | Micrometer / Prometheus |
| **Security** | Cloud Identity Aware Proxy | Spring Security |

