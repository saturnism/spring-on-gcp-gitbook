# Artifact Repository

When developing applications in a larger project, you may find a need to share common libraries across multiple teams or applications. If this library is a public OSS library, it's usually hosted on Maven Central. For an internal library, though, you'll need to use a private repository. Typically, in an on-premise datacenter, these Java \(Maven\) artifacts in private repositories such as Sonatype Nexus, or JFrog Artifactory.

On Google Cloud, you can continue setup/configure/use these repositories. JFrog can also run [Artifactory as a hosted service on Google Cloud](https://jfrog.com/partner/google-cloud-platform/)!

Google Cloud also has a fully managed artifact repository service called [Artifact Registry](https://cloud.google.com/artifact-registry) \(beta\), and you can use it to store container images, NPM packages, and Java artifacts.

{% hint style="info" %}
See [Artifact Registry documentation](https://cloud.google.com/artifact-registry) for more information.
{% endhint %}

## Artifact Registry - Maven Repository

Artifact Registry can host Maven repositories to host the Java artifacts. Artifacts are hosted within a region of your choice, and you can apply Identity Access Management to control who can access/update artifacts.

{% embed url="https://www.youtube.com/watch?v=2-P4cSCk1VM" %}

{% hint style="warning" %}
Artifact Registry is currently in beta, and the Maven Repository feature is in Alpha. You'll need to sign up for the Alpha program first.

[Sign up for Artifact Registry Alpha](https://docs.google.com/forms/d/e/1FAIpQLSf5q3CeDna_c27ifadF1KO17W3PrYO91w-UI-jjUdnvGS1cmQ/viewform) feature to try the hands-on instructions.
{% endhint %}

Once you are confirmed to be enrolled in the alpha program, you can give it a try!

### Enable API

```bash
gcloud services enable artifactregistry.googleapis.com
```

### Create a Maven Repository

```bash
gcloud beta artifacts repositories create private-maven-repo \
  --repository-format=maven \
  --location=us-central1
```

### List Artifacts

```bash
gcloud beta artifacts packages list \
  --repository=private-maven-repo \
  --location=us-central1
```

There should be no artifacts at the moment.

### Deploy a Maven Artifact

You need to update the build configuration \(e.g., `pom.xml`\) in order to configure an artifact to Artifact Registry's Maven repository. You can find the full configuration needed through by running the utility command:

#### Generate a New Project

This example will use Maven. First, create a brand new Maven project:

```bash
mvn archetype:generate \
  -DinteractiveMode=false \
  -DgroupId=com.example \
  -DartifactId=common-libs \
  -DarchetypeGroupId=org.apache.maven.archetypes \
  -DarchetypeArtifactId=maven-archetype-quickstart
  
cd common-libs/
```

#### Build Configuration

{% tabs %}
{% tab title="Maven" %}
```bash
gcloud beta artifacts print-settings mvn \
  --repository=private-maven-repo \
  --location=us-central1
```

Note that an Artifact Registry Wagon extension is needed to publish to Artifact Registry.
{% endtab %}

{% tab title="Gradle" %}
```bash
gcloud beta artifacts print-settings gradle \
  --repository=private-maven-repo \
  --location=us-central1
```

Note that an Artifact Registry Gradle plugin is needed to publish to Artifact Registry.
{% endtab %}
{% endtabs %}

This example uses Maven, so edit the `pom.xml` to add the additional settings:

```markup
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  
  ...
  
  <!-- Add Distributuion Management -->
  <distributionManagement>
    ...
  </distributionManagement>
  
  <!-- Add Repository -->
  <repositories>
    ...
  </repositories>

  <build>
    <!-- Add the Wagon Extension -->
    <extensions>
      <extension>
        <groupId>com.google.cloud.artifactregistry</groupId>
        <artifactId>artifactregistry-maven-wagon</artifactId>
        <version>2.1.0</version>
      </extension>
    </extensions>
    
    <pluginManagement>
      ...
    </pluginManagement>
  </build>
</project>
```

#### Build and Deploy

```bash
mvn clean package deploy
```

Verify that the artifact is published!

```bash
gcloud alpha artifacts packages list \
  --repository=private-maven-repo \
  --location=us-central1
```

In the Cloud Console, you can also browse to **Artifact Registry &gt; private-maven-repo.**

![](../../.gitbook/assets/image%20%2842%29.png)

And see manage the artifacts:

![](../../.gitbook/assets/image%20%2841%29.png)

{% hint style="info" %}
See [Artifact Registry Quickstart for Maven and Gradle](https://cloud.google.com/artifact-registry/docs/java/quickstart) for more information.
{% endhint %}

