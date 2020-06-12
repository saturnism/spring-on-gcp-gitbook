# Cloud Firestore

Cloud Firestore is a managed, highly scalable, NoSQL database service. Cloud Firestore automatically handles sharding and replication, providing you with a highly available and durable database that scales automatically to handle your applications' load.

Cloud Firestore has two modes - Datastore Mode, and Native Mode. While both modes are NoSQL databases, there are a lot of difference between them, primarily:

* Native Mode is a real-time database, meaning you can listen to updates in real-time, and the data model is composed of Document and Collection of documents..
* Datastore mode does not have the real-time capability, and the data model is composed of Entity and organized in Kind of entities.

{% hint style="info" %}
Read [Choosing between Native mode and Datastore mode](https://cloud.google.com/datastore/docs/firestore-or-datastore) for more information.
{% endhint %}

For traditional backend applications where real-time data updates is not needed, the Datastore mode is simple to use. For backend applications that wants to adopt reactive programming model, then the Native mode is better suited. 

{% hint style="danger" %}
A single Google Cloud Project can only choose one of the modes.  Once the mode chosen, you cannot change the mode. I.e., if you created a project that chose to use the Native mode, then the same project can no longer use the Datastore mode.
{% endhint %}

Learn how to use each of the mode in the following pages.

{% page-ref page="cloud-firestore-datastore.md" %}



