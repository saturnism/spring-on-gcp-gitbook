# Secure Container Image

While [Creating a new container image](container-image.md#containerize) is fairly easy and straightforward for development purposes, you should consider building a secure container image for production use. Below are some basic considerations.

## No Source Code

It's a common mistake to copy too much data/information into a container image. In general, you should only have the content that's absolutely necessary to run your application. But there are things that often make it into a container image that you may not realize:

* Source code, build files are easily copied into a runtime container image by accident when using a Dockerfile.
* Version control directories, such as `.git` are easily copied into a runtime container image by accident when using a Dockerfile.

{% tabs %}
{% tab title="Jib" %}
Jib automatically builds thin container images without the source.
{% endtab %}

{% tab title="Buildpacks" %}
Paketo automatically builds thin container images without the source.

GCP Buildpack needs to set `GOOGLE_CLEAR_SOURCE=true` to remove the source from the container image. See [GCP Buildpack README](https://github.com/GoogleCloudPlatform/buildpacks#configuration) for more information.

```bash
PROJECT_ID=$(gcloud config get-value project)                                                            ⬢ system ⎈ demo-cluster
pack build \
  -e "GOOGLE_CLEAR_SOURCE=true"
  --builder gcr.io/buildpacks/builder:v1 \
  --publish \
  gcr.io/${PROJECT_ID}/helloworld
```
{% endtab %}
{% endtabs %}

## No Secrets/Credentials

Do not copy secrets and/or credentials into a container image \(e.g., do not copy a service account key file!\). For the most part, secrets can be stored in the runtime environment \(e.g., a Kubernetes Secret\), or better, a secret store \(e.g., [Cloud Secret Manager](../../app-dev/cloud-services/secret-management.md), or HashiCorp Vault\).

## Minimal Base Image

Many base images comes with all the command line utilities from a typical Linux distribution \(e.g., a shell, package manager, etc\). These container images may allow you \(or an attacker!\) to get into a shell, and install additional tools. To reduce the attack surface, consider using a minimal base image that has the least attack surface. These images will be more secure, but may also be harder to debug.

{% tabs %}
{% tab title="Jib" %}
Jib uses the [Distroless](https://github.com/GoogleContainerTools/distroless/blob/master/java/README.md) base image by default.
{% endtab %}

{% tab title="Buildpacks" %}
Buildpack's runtime image ultimately does not use Distroless. Paketo, for example, executed shell script to calculate memory needs, and thus Shell is needed. There is no easy way to switch out the base image when using a Buildpack. You may need to create your own to change the base image.
{% endtab %}
{% endtabs %}

## Non-Root User

One of the most overlooked configuration for a container image is which user is used to run your application? In a VM environment, you would never want to run an application as `root`. It's no different in a container. Every container image may have a different set of non-privileged users.

For example, for a Distroless base image \(using a debug image that has a shell\):

```bash
docker run -ti --rm --entrypoint=sh \
  gcr.io/distroless/java:debug -c "cat /etc/passwd"
```

You'll see that it has only 3 users:

```text
root:x:0:0:root:/root:/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/sbin/nologin
nonroot:x:65532:65532:nonroot:/home/nonroot:/sbin/nologin
```

But, an AdoptOpenJDK base image has more system users, and you'll need to pick the one you want to use as the user to run your application:

```bash
docker run -ti --rm adoptopenjdk:11-jre-hotspot-bionic cat /etc/passwd
```

{% tabs %}
{% tab title="Jib" %}
Jib uses `root` user by default. You should configure it to use a non-root user according to the base image you use. For example:

```bash
PROJECT_ID=$(gcloud config get-value project)
./mvnw compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Djib.container.user=nonroot:nonroot
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

Validate that the JVM was started with the `nonroot` user:

```bash
PROJECT_ID=$(gcloud config get-value project)

docker pull gcr.io/${PROJECT_ID}/helloworld
docker run -ti --rm --entrypoint=java \
  gcr.io/${PROJECT_ID}/helloworld \
  -XshowSettings:properties -version
```

Look for the `user.name` property is now `nonroot`.
{% endtab %}

{% tab title="Buildpacks" %}
Buildpacks \(with Paketo and GCP builders\) run as a non-root user by default, as the user `cnb`.
{% endtab %}
{% endtabs %}

## Summary

So, what do the automated tools do by default?

<table>
  <thead>
    <tr>
      <th style="text-align:left"></th>
      <th style="text-align:left">Jib</th>
      <th style="text-align:left">Paketo Builder</th>
      <th style="text-align:left">GCP Builder</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Source Code</td>
      <td style="text-align:left">No source in runtime</td>
      <td style="text-align:left">No source in runtime</td>
      <td style="text-align:left">
        <p>Set <code>GOOGLE_CLEAR_SOURCE.</code> 
        </p>
        <p>See <a href="https://github.com/GoogleCloudPlatform/buildpacks#configuration">README</a>.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Minimal Base Image</td>
      <td style="text-align:left">Uses Distroless</td>
      <td style="text-align:left">Not Distroless</td>
      <td style="text-align:left">Not Distroless</td>
    </tr>
    <tr>
      <td style="text-align:left">Non-Root User</td>
      <td style="text-align:left">
        <p>Defaults to <code>root</code>,</p>
        <p>Configure to non-root.</p>
      </td>
      <td style="text-align:left"><code>cnb</code> user</td>
      <td style="text-align:left"><code>cnb</code> user</td>
    </tr>
  </tbody>
</table>
