# Profiling

## Cloud Profiler

[Cloud Profiler](https://cloud.google.com/profiler/docs/concepts-profiling) allows you to continuously profile CPU and heap usages to help identify performance bottlenecks and critical paths in your application. It'll be able to produce flame graph on which parts of your application uses the most CPU and/or Heap.

### Enable API

```bash
gcloud services enable cloudprofiler.googleapis.com
```

### CPU Time

The CPU time for a function tells you how long the CPU was busy executing instructions. It doesn't include the time the CPU was waiting or processing instructions for something else.

![](../../.gitbook/assets/image%20%2823%29.png)

### Wall Time

The wall-clock time for a function measures the time elapsed between entering and exiting a function. Wall-clock time includes all wait time, including that for locks and thread synchronization. If the wall-clock time is significantly longer than the CPU time, then that is an indication the code spends a lot of time waiting. This might be an indication of a resource bottleneck.

![](../../.gitbook/assets/image%20%2825%29.png)

### Heap

The heap consumption is the amount of memory allocated in the Java program's heap - this can help you find potential inefficiencies and memory leaks in your application.

![](../../.gitbook/assets/image%20%2828%29.png)



## Java Agent

Cloud Profiler works by adding a Java agent to your JVM startup argument, and the agent can communicate with the Cloud Profiler service in the Cloud. Through the Cloud Console, you can then see the collected profile data.

### Agent Files

A Cloud Profiler agent can work both within Google Cloud environments using the [Machine Credentials](../../getting-started/google-cloud-platform.md#machine-credentials-from-metadata-server), and outside of Google Cloud environments \(e.g., on-premises, and in another cloud\) using a [Service Account key file](../../getting-started/google-cloud-platform.md#service-account-key).

| Latest Version | Versioned URL |
| :--- | :--- |
| [Download](https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent.tar.gz) | https://storage.googleapis.com/cloud-profiler/java/cloud-profiler-java-agent\_${VERSION}.tar.gz |

Unfortunately, the list of versions are not available on GitHub. The only way to see the list of available versions is listing the Google Cloud Storage bucket that contains all the binaries:

```bash
gsutil ls gs://cloud-profiler/java/cloud-profiler-*
```

{% hint style="info" %}
See [Profiling Java Applications](https://cloud.google.com/profiler/docs/profiling-java) for more information.
{% endhint %}

### Agent Configurations

#### Agent Path

To use the agent, you'll need to configure the JVM command line using the standard  `-agentpath` , e.g.:

```bash
java -agentpath:/opt/cprof/profiler_java_agent.so \
  -jar ...
```

Rather than hard coding the startup command line, you can also configure it with the `JAVA_TOOL_OPTIONS` environmental variable:

```bash
JAVA_TOOL_OPTIONS="-agentpath:/opt/cprof/profiler_java_agent.so.so"
java -jar ...
```

#### Agent Configurations

You can specify additional Agent configurations within the same `agentpath` argument, in the form of `java -agentpath:/opt/cprof/profiler_java_agent.so=FLAG1=VALUE1,FLAG2=VALUE2`.

Heap sampling is only available in Java 11 and higher. To turn on both CPU and Heap profiling for a Java 11 application:

```bash
java -agentpath:/opt/cprof/profiler_java_agent.so=-cprof_enable_heap_sampling=true \
  -jar ...
```

{% hint style="info" %}
See [Profiling Java applications Agent Configuration document](https://cloud.google.com/profiler/docs/profiling-java#agent_configuration) for all the possible agent configuration flags.
{% endhint %}

#### Logging

By default the Cloud Profiler does not output any logs. You can turn on logging by using `-logtostderr`flag, and configure the log level using `‑minloglevel`flag.

```text
java -agentpath:/opt/cprof/profiler_java_agent.so=-logtostderr,-minloglevel=2 \
  -jar ...
```

{% hint style="info" %}
See [Profiling Java applications Agent Logging document](https://cloud.google.com/profiler/docs/profiling-java#agent_logging) for all the possible log levels.
{% endhint %}

### Runtime Configuration

{% tabs %}
{% tab title="App Engine" %}
Follow [App Engine Hello World!](../../getting-started/helloworld/app-engine.md) instructions to deploy an application to App Engine.

Cloud Profiler agent is already present in your App Engine application. However, it is not on by default. You can turn it on by using the `JAVA_TOOL_OPTIONS` environmental variable in an `app.yaml` file:

{% code title="app.yaml" %}
```yaml
runtime: java11
env_variables:
  JAVA_TOOL_OPTIONS: "-agentpath:/opt/cprof/profiler_java_agent.so=-logtostderr,-cprof_enable_heap_sampling=true"
```
{% endcode %}

Redeploy the application with the `app.yaml` file:

```bash
gcloud app deploy target/helloworld.jar \
  --appyaml app.yaml
```

It'll take a couple of minutes before Cloud Profiler can display the information. In Cloud Profiler console, you can find the Default service in the drop down:

![](../../.gitbook/assets/image%20%2821%29.png)
{% endtab %}

{% tab title="Cloud Run" %}
Add the Cloud Profiler Java agent to the container, and configure the agent in the startup command line.

#### Clone

```text
# Clone the sample repository manually
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

#### Containerize with a Dockerfile

In the Dockerfile, download the Cloud Debugger and build it as part of the container image:

{% code title="Dockerfile" %}
```text
FROM openjdk:11

# Create a directory for the Profiler. Add and unzip the agent in the directory.
RUN mkdir -p /opt/cprof && \
  wget -q -O- https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent.tar.gz \
  | tar xzv -C /opt/cprof

COPY target/helloworld.jar /app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```
{% endcode %}

Then build and push the container:

```bash
mvn package

PROJECT_ID=$(gcloud config get-value project)
docker build -t gcr.io/${PROJECT_ID}/helloworld .
docker push gcr.io/${PROJECT_ID}/helloworld
```

#### Containerize with Jib

Download the Cloud Debugger Java agent into `src/main/jib` directory so that Jib can include the agent files as part of the container image:

```text
# Make a directory to store the Java agent
mkdir -p src/main/jib/opt/cprof

# Download and extract the Java agent to the directory
wget -qO- https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent.tar.gz | \
  tar xvz -C src/main/jib/opt/cprof
```

Create the container image with Jib:

```bash
PROJECT_ID=$(gcloud config get-value project)
mvn compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

#### Deploy

Deploy to Cloud Run with Debugger Enabled using the environmental variable:

```bash
gcloud run deploy helloworld \
  --region=us-central1 \
  --platform=managed \
  --allow-unauthenticated \
  --set-env-vars="JAVA_TOOL_OPTIONS=-agentpath:/opt/cprof/profiler_java_agent.so=-logtostderr,-cprof_enable_heap_sampling=true" \
  --image=gcr.io/${PROJECT_ID}/helloworld
```

In Cloud Profiler console, you can see the `helloworld` service in the drop down:

![](../../.gitbook/assets/image%20%2827%29.png)
{% endtab %}

{% tab title="Kubernetes Engine" %}
Add the Cloud Profiler Java agent to the container, and configure the agent in the startup command line.

#### Clone

```text
# Clone the sample repository manually
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

#### Containerize with a Dockerfile

In the Dockerfile, download the Cloud Debugger and build it as part of the container image:

{% code title="Dockerfile" %}
```text
FROM openjdk:11

# Create a directory for the Profiler. Add and unzip the agent in the directory.
RUN mkdir -p /opt/cprof && \
  wget -q -O- https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent.tar.gz \
  | tar xzv -C /opt/cprof

COPY target/helloworld.jar /app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```
{% endcode %}

Then build and push the container:

```bash
mvn package

PROJECT_ID=$(gcloud config get-value project)
docker build -t gcr.io/${PROJECT_ID}/helloworld .
docker push gcr.io/${PROJECT_ID}/helloworld
```

#### Containerize with Jib

Download the Cloud Debugger Java agent into `src/main/jib` directory so that Jib can include the agent files as part of the container image:

```text
# Make a directory to store the Java agent
mkdir -p src/main/jib/opt/cprof

# Download and extract the Java agent to the directory
wget -qO- https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent.tar.gz | \
  tar xvz -C src/main/jib/opt/cprof
```

Create the container image with Jib:

```bash
PROJECT_ID=$(gcloud config get-value project)
mvn compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

#### Deploy

Deploy to Kubernetes Engine with Debugger Enabled using the environmental variable using a Deployment YAML:

```bash
# Make a directory to store Kubernetes YAMLs
mkdir k8s/
```

Create a  Deployment YAML file and configure the environmental variable:

{% code title="k8s/deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: helloworld
  name: helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - image: gcr.io/YOUR_PROJECT_ID/helloworld
        name: helloworld
        env:
        - name: JAVA_TOOL_OPTIONS
          value: "-agentpath:/opt/cprof/profiler_java_agent.so=-logtostderr,-cprof_enable_heap_sampling=true,-cprof_service=helloworld-gke,-cprof_service_version=1.0"
```
{% endcode %}

Deploy the YAML file:

```bash
kubectl apply -f k8s/deployment.yaml
```

In Cloud Debugger console, you can see the `helloworld` service in the drop down:

![](../../.gitbook/assets/image%20%2820%29.png)
{% endtab %}

{% tab title="Compute Engine" %}
Follow the [Compute Engine Hello World!](../../getting-started/helloworld/compute-engine.md) to deploy an application in Compute Engine.

SSH into the Compute Engine instance:

```bash
gcloud compute ssh helloworld
```

From the Compute Engine instance, download the Java agent:

```bash
sudo mkdir -p /opt/cprof
curl -s -o- https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent.tar.gz \
  | sudo tar xvz -C /opt/cprof
```

Run the Java application with the Cloud Debugger agent:

```bash
java -agentpath:/opt/cprof/profiler_java_agent.so=-logtostderr,-cprof_enable_heap_sampling=true,-cprof_service=helloworld-gce,-cprof_service_version=1.0 \
  -jar helloworld.jar
```

In Cloud Debugger console, you can see the `helloworld` service in the drop down:

![](../../.gitbook/assets/image%20%2822%29.png)
{% endtab %}

{% tab title="Non-Google Cloud Environment" %}
You can attach the Cloud Debugger agent to any Java application even if it runs outside of the Google Cloud environment \(whether it's in a container, or on your local laptop, or in another cloud\). Authentication has to be done using Service Account key file rather than using the Machine Credentials.

{% hint style="danger" %}
This only works on a Linux x86 based system.
{% endhint %}

#### Clone

```bash
# Clone the sample repository manually
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

#### Build

```bash
mvn package
```

#### Download Agent

```bash
sudo mkdir -p /opt/cprof
curl -s -o- https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent.tar.gz \
  | sudo tar xvz -C /opt/cprof
```

#### Create a Service Account

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud iam service-accounts create helloworld-app
```

#### Add IAM Permission

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
  --member serviceAccount:helloworld-app@${PROJECT_ID}.iam.gserviceaccount.com \
  --role roles/cloudprofiler.agent
```

#### Create a Service Account Key File

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud iam service-accounts keys create \
  $HOME/helloworld-app-sa.json \
  --iam-account helloworld-app@${PROJECT_ID}.iam.gserviceaccount.com
```

#### Use Service Account Cloud Debugger Agent

```bash
PROJECT_ID=$(gcloud config get-value project)
GOOGLE_APPLICATION_CREDENTAILS=$HOME/helloworld-app-sa.json
java -agentpath:/opt/cprof/profiler_java_agent.so=-logtostderr,-cprof_enable_heap_sampling=true,-cprof_service=helloworld-local,-cprof_service_version=1.0,-cprof_project_id=${PROJECT_ID} \
  -jar target/helloworld.jar
```
{% endtab %}
{% endtabs %}





