# Cloud Shell

[Cloud Shell](https://cloud.google.com/shell/docs) is an interactive shell environment for Google Cloud Platform that works directly in the browser. Cloud Shell has many commonly used CLI tools preinstalled, including `gcloud`. Almost all the tasks/examples on this site can be done in Cloud Shell, so you don't need to install any CLIs!

## Select a Project

After you logged into the Google Cloud console, make sure you have selected a project in the top bar, by clicking **Select a project**.

![Select a project](../.gitbook/assets/image%20%2832%29.png)

From the **Select a project** dialog, select a Google Cloud Project to use. This will help pre-configure Cloud Shell to use that project as the default project.

## Activate Cloud Shell

On the top right, click the **Activate Cloud Shell** icon.

![Activate Cloud Shell icon](../.gitbook/assets/image%20%2828%29.png)

If it's your first time using Cloud Shell, in the introduction dialog, click **Start Cloud Shell** to continue. Wait for the Cloud Shell machine to provision \(it may take a few minutes\).

Make sure your Cloud Shell is configured with the current project by checking the current Project ID configured for `gcloud`:

```bash
gcloud config get-value project
```

## Home Directory

The Cloud Shell instance is ephemeral, but your home directory will persist and its contents will be carried to future Cloud Shell sessions. Any data/binaries stored outside of the home directory may be lost.

If you want to install any additional binaries, make sure to store them inside your home directory, maybe under a `bin` directory.

```bash
mkdir $HOME/bin
echo 'export PATH="$HOME/bin:$PATH"' >> $HOME/.bashrc
```

## Boost Mode

When working with Java applications and running heavier workloads in Cloud Shell, it'll be useful to enable Boost Mode. In the Cloud Shell's **More** menu, click **Boost Cloud Shell**.

![Boost Cloud Shell](../.gitbook/assets/image%20%2830%29.png)

This will re-provision your Cloud Shell instance and replace the original `e2-small` \(0.5 vCPU, 2GB of memory\) machine type with a larger `e2-medium` \(1 vCPU, 4GB of memory\) machine type.

{% hint style="info" %}
See [Compute Engine Machine Types documentation](https://cloud.google.com/compute/docs/machine-types#e2_machine_types) for more details on the machine types.
{% endhint %}

## Multiple Tabs

You can open new Cloud Shell tabs by clicking the **+** icon.

![Open a new tab + icon](../.gitbook/assets/image%20%2823%29.png)

## Code Editor

Cloud Shell comes with common text editing tools, such as `vi`, `emacs`, `nano`. It also has a built-in web-based text editor. You can open the web-based editor by clicking **Open Editor**.

![Open Editor](../.gitbook/assets/image%20%2834%29.png)

This will launch an embedded editor where you can open and edit text files.

You can switch back to the terminals by clicking **Open Terminal**.

![Open Terminal](../.gitbook/assets/image%20%2821%29.png)


## Default Zone and Region

You can specify the default `zone` or `region` with a `gcloud` command. If you primarily operate within a single zone or region, set the default `zone` and default `region`.

```bash
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-c
gcloud config set run/region us-central1
```

{% hint style="info" %}
See the complete list in [Regions and Zones documentation](https://cloud.google.com/compute/docs/regions-zones).
{% endhint %}
