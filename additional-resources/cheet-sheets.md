# Cheat Sheets

Here are some `gcloud` CLI command lines and reference documentations that are frequently used.

## Basics

| Task | Command |
| :--- | :--- |
| Enable an API | `gcloud services enable ${API}` |
| Current Project ID | `gcloud config get-value project` |
| Export to PROJECT\_ID | `PROJECT_ID=$(gcloud config get-value project)` |
| Authentication | `gcloud auth login` |
| Application Credentials Login | `gcloud auth application-default login` |
| Default Zone | `gcloud config set compute/zone us-central1-c` |
| Default Region | `gcloud config set compute/region us-central1` |
| All Regions and Zones | [Regions and Zones](https://cloud.google.com/compute/docs/regions-zones) |

## Identity Access Management

<table>
  <thead>
    <tr>
      <th style="text-align:left">Task</th>
      <th style="text-align:left">Command</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Create a Service Account</td>
      <td style="text-align:left"><code>gcloud iam service-accounts create sa</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Service Account E-Mail</td>
      <td style="text-align:left"><code>sa@${PROJECT_ID}.iam.gserviceaccount.com</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Add Permission</td>
      <td style="text-align:left">
        <p><code>gcloud projects add-iam-policy-binding ${PROJECT_ID} \</code>
        </p>
        <p><code>  --member serviceAccount:${SERVICE_ACCOUNT_EMAIL} \</code>
        </p>
        <p><code>  --role ${ROLES}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Create a Service Account Key File</td>
      <td style="text-align:left">
        <p><code>gcloud iam service-accounts keys create \</code>
        </p>
        <p><code>  $HOME/sa-key.json \</code>
        </p>
        <p><code>  --iam-account ${SERVICE_ACCOUNT_EMAIL}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">All Possible Roles</td>
      <td style="text-align:left"><a href="https://cloud.google.com/iam/docs/understanding-roles">Understanding roles</a>
      </td>
    </tr>
  </tbody>
</table>

