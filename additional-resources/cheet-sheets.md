# Cheat Sheets

Here are some  commands and links to reference documentations that are frequently used.

## gcloud

### Basics

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

### Identity Access Management

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
      <td style="text-align:left">
        <p><code>gcloud iam service-accounts create \</code>
        </p>
        <p><code>  ${SA_NAME}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Service Account E-Mail</td>
      <td style="text-align:left"><code>${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Add Permission</td>
      <td style="text-align:left">
        <p><code>gcloud projects add-iam-policy-binding ${PROJECT_ID} \</code>
        </p>
        <p><code>  --member serviceAccount:${SA_EMAIL} \</code>
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
        <p><code>  --iam-account ${SA_EMAIL}</code>
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

### Serverless Deployments

<table>
  <thead>
    <tr>
      <th style="text-align:left">Task</th>
      <th style="text-align:left"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">App Engine</td>
      <td style="text-align:left"><code>gcloud app deploy ${JAR_FILE}</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"><code>gcloud app deploy ${JAR_FILE} --appyaml app.yaml</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Cloud Run</td>
      <td style="text-align:left">
        <p><code>gcloud run deploy ${NAME} \</code>
        </p>
        <p><code>  --platform=managed \</code>
        </p>
        <p><code>  --allow-unauthenticated \</code>
        </p>
        <p><code>  --image=gcr.io/${PROJECT_ID}/${IMAGE_NAME}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p><code>gcloud run deploy ${NAME} \</code>
        </p>
        <p><code>  --platform=managed \</code>
        </p>
        <p><code>  --allow-unauthenticated \</code>
        </p>
        <p><code>  --cpu=2 \</code>
        </p>
        <p><code>  --memory=512M \</code>
        </p>
        <p><code>  --set-env-vars=&quot;JAVA_TOOL_OPTIONS=-Dproperty=value&quot;</code>
        </p>
        <p><code>  --image=gcr.io/${PROJECT_ID}/${IMAGE_NAME}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Cloud Function</td>
      <td style="text-align:left">
        <p><code>gcloud functions deploy ${NAME}</code>
        </p>
        <p><code>  --trigger-http \</code>
        </p>
        <p><code>  --runtime=java11 \</code>
        </p>
        <p><code>  --allow-unauthenticated \</code>
        </p>
        <p><code>  --entry-point=${FUNCTION_CLASS_FQN}</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>

## Jib

<table>
  <thead>
    <tr>
      <th style="text-align:left">Task</th>
      <th style="text-align:left">Command</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Create a remote Image</td>
      <td style="text-align:left"><code>mvn jib:build</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"><code>gradle jib</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Create a local image</td>
      <td style="text-align:left"><code>mvn jib:dockerBuild</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"><code>gradle jibDockerBuild</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>Run Jib without</p>
        <p>pre-configured plugin</p>
      </td>
      <td style="text-align:left">
        <p><code>mvn compile \</code>
        </p>
        <p><code>com.google.cloud.tools:jib-maven-plugin:2.4.0:build \</code>
        </p>
        <p><code>-Dimage=gcr.io/${PROJECT_ID}/${IMAGE_NAME}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Jib READMEs</td>
      <td style="text-align:left"><a href="https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin">Maven</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"><a href="https://github.com/GoogleContainerTools/jib/tree/master/jib-gradle-plugin">Gradle</a>
      </td>
    </tr>
  </tbody>
</table>

## Spring Initializer

<table>
  <thead>
    <tr>
      <th style="text-align:left"></th>
      <th style="text-align:left"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p>Create a new app w/</p>
        <p>Spring Cloud GCP</p>
      </td>
      <td style="text-align:left">
        <p><code>curl https://start.spring.io/starter.zip  \</code>
        </p>
        <p><code>  -d dependencies=web,cloud-gcp \</code>
        </p>
        <p><code>  -d bootVersion=2.3.1.RELEASE \</code>
        </p>
        <p><code>  -d baseDir=demo</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>

