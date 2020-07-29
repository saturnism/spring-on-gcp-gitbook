---
description: >-
  In this section, you'll create a container image for a Hello World
  application.
---

# Container Image

This section continues from the previous section - make sure you do the tutorial in sequence.

{% page-ref page="kubernetes-cluster.md" %}

## Clone

```text
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

## Build

```text
./mvnw package
```

## OpenJDK Version

When you containerize a Java application, make sure you use a base JDK image that is container-aware \(CGroup aware\) so that the JDK can allocate memory and CPU counts properly.

Older versions of JDK \(prior to 8u192\) may not have container awareness \(or may have experimental support that requires explict flags to enable\). Older versions of JDK may look at the traditional `/proc/meminfo` and `/proc/cpuinfo` files for available memory and CPUs. The content of these files reflects the amount of resources of the host/node machine that is running the container, but do not reflect the actual limits assigned to the container \(which may be much less\).

Newer versions of JDK \(8u192 and above\) will automatically discover the CGroup resource allocations located in `/sys/fs/cgroup/cpu` and `/sys/fs/cgroup/memory`.

### Heap

Run a Docker container and give it only 256MB of memory,  and see an older version of JDK will assign for the default Max Heap.

```bash
docker run -ti --rm --cpus=1 --memory=256M openjdk:8u141-jre \
  java -XX:+PrintFlagsFinal -version | grep MaxHeapSize
```

Because version `8u141` is not container-aware, it will output the `MaxHeapSize` \(in bytes\) that is calculated from the host machine and can be significantly higher than the 256MB of memory you originally assigned. This means your Java process may allocate heap aggressively and go beyond the original limit, causing the container instance to be killed, usually result in a `OOMKilled`message.

Run the same command, but with a newer version of JDK:

```bash
 docker run -ti --rm --cpus=1 --memory=256M openjdk:8u252-jre \
   java -XX:+PrintFlagsFinal -version | grep MaxHeapSize
```

The output of `MaxHeapSize` is now `132120576` bytes, which is ~126MB, indicating that it's now respecting the 256MB limitation we assigned for the container.

The JVM heap size should never be equal  memory resource you assigned. In this case, even though we assigned 256MB of memory to the container, the Max Heap must be much lower than that \(e.g., 50% of that, or depending on your application\). This is because the JVM also uses native memory in addition to the heap.

JVM native memory usages contains thread stack, code cache, metaspace, and potentially direct memory buffer allocations.

#### Estimate Memory Needs

According to the [Cloud Foundry Java Buildpack Memory calculator documentation](https://docs.google.com/document/d/1vlXBiwRIjwiVcbvUGYMrxx2Aw1RVAtxq3iuZ3UK2vXA/edit), the total native memory needed for a JVM instance is approximately linear to the number of loaded classes.

You can use [Cloud Foundry Java Buildpack Memory calculator](https://github.com/cloudfoundry/java-buildpack-memory-calculator) to the memory needs and configurations.

#### Understand Memory Used

In cases where you are getting `OOMKilled` for your container instance, and have already made sure that you are using a container-aware version of JDK, then you may want to turn on [Native Memory Tracking](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html).

Native Memory Tracking can only be enabled via command line argument, and cannot be enabled using `JAVA_TOOL_OPTIONS`. 

You can run this command to see a sample output of Native Memory Tracking:

```bash
docker run -ti --rm openjdk:8u252-jre \
  java -XX:+UnlockDiagnosticVMOptions \
  -XX:NativeMemoryTracking=summary \
  -XX:+PrintNMTStatistics \
  -version
```

Native Memory Tracking can only print out memory usage details upon a **successful** exit.

```bash
java -XX:+UnlockDiagnosticVMOptions \
  -XX:NativeMemoryTracking=summary \
  -XX:+PrintNMTStatistics \
  -jar ...
```

If your application was `OOMKilled`, then it's an unsuccessful exit, so the memory details may not be printed. In this case, consider first increase the amount of memory allocation, and then trigger a successful exit, to get the native memory usage details.

### CPU

Run a Docker container and giving it only 2 CPUs,  and see what an older version of JDK will assign for the default Parallel GC threads.

```bash
docker run -ti --rm --cpus=2 openjdk:8u141-jre java \
  -XX:+PrintFlagsFinal -XX:+UseParallelGC -version | grep ParallelGCThreads
```

It will output the `ParallelGCThreads` that is calculated from the number of CPUs of the host machine and can be significantly higher than `2`.

Run the same command, but with a newer version of JDK:

```bash
docker run -ti --rm --cpus=2 --memory=256M openjdk:8u252-jre java \
  -XX:+PrintFlagsFinal -XX:+UseParallelGC -version | grep ParallelGCThreads
```

The output of `ParallelGCThreads` is `2`.

### Runtime API

When using non-container-aware JDK versions, both Memory and CPU can be inaccurately reflected in the [`Runtime`](https://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html) API as well.

```java
// Max heap you can use
Runtime.getRuntime().maxMemory()

// Number of processors
Runtime.getRuntime().availableProcessors()
```

This is important because some libraries and applications may use `availableProcessors` to determine the size of the thread pools. So, if you allocated only `2` CPUs, but the JVM inaccurately sees `32` CPUs from the host, then the libraries may over-allocate the thread pool size, and causing your application to run more than the underlying system allows.

## Containerize

{% tabs %}
{% tab title="Jib" %}
[Jib](https://github.com/GoogleContainerTools/jib) can containerize any Java application easily, without a `Dockerfile` nor `docker` installed. Jib will push the container image directly to the remote registry.

```bash
PROJECT_ID=$(gcloud config get-value project)
./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

{% hint style="info" %}
You can configure [Jib Maven plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin) or [Jib Gradle plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-gradle-plugin) directly in the build file to run the Jib easier, such as `./mvnw jib:build`.
{% endhint %}
{% endtab %}

{% tab title="Buildpack" %}
[Cloud Native Buildpacks](https://buildpacks.io) can containerize applications written in different language without a `Dockerfile`. It does require `docker` installed.

1. Install Docker locally - see [Get Docker documentation](https://docs.docker.com/get-docker/).
2. Install `pack` CLI - see [Installing `pack` documentation](https://buildpacks.io/docs/install-pack/)
3. Build container with `pack`, and use `--publish` flag to push directly to the remote registry:

```bash
PROJECT_ID=$(gcloud config get-value project)
pack build --builder gcr.io/buildpacks/builder:v1 \
  --publish \
  gcr.io/${PROJECT_ID}/helloworld
```
{% endtab %}

{% tab title="Spring Boot 2.3" %}
Since Spring Boot 2.3+, you can build container using the Spring Boot plugin.

```bash
PROJECT_ID=$(gcloud config get-value project)

# Maven with Paketo Buildpack
./mvnw spring-boot:build-image \
  -Dspring-boot.build-image.imageName=gcr.io/${PROJECT_ID}/helloworld
  
# Maven with GCP Buildpack
./mvnw spring-boot:build-image \
  -Dspring-boot.build-image.imageName=gcr.io/${PROJECT_ID}/helloworld \
  -Dspring-boot.build-image.builder=gcr.io/buildpacks/builder
  
# Gradle with Paketo Buildpack
./gradlew bootBuildImage --imageName=gcr.io/${PROJECT_ID}/helloworld

# Gradle with GCP Buildpack
./gradlew bootBuildImage --imageName=gcr.io/${PROJECT_ID}/helloworld \
  --builder=gcr.io/buildpacks/builder
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Luckily, all of these tools automatically try to use the newest version of the JDK that is container-aware.
{% endhint %}

## Run Locally

If you have Docker installed locally, you can run the docker container locally to ensure everything works. This command will run the container locally and forward localhost's port 8080 to the container instance's port 8080.

```bash
PROJECT_ID=$(gcloud config get-value project)
docker pull gcr.io/${PROJECT_ID}/helloworld
docker run -ti --rm -p 8080:8080 gcr.io/${PROJECT_ID}/helloworld
```

## Connect Locally

You can connect to the container that's running locally

```bash
curl localhost:8080
```

