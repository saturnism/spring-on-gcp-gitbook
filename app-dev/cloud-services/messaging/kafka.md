# Kafka

Google Cloud does not have first-party managed Kafka service. For messaging, you can mostly use Cloud Pub/Sub. If you need capabilities of Kafka, then you can run Kafka cluster either as a third-party managed service \(e.g., from Confluent Cloud\), or run it on Kubernetes with an operator.

## Confluent Cloud

Confluent can create managed Kafka clusters using [Confluent Cloud](https://www.confluent.io/confluent-cloud/) on Google Cloud. Check out the [Confluent Cloud's Quickstart documentation](https://docs.confluent.io/current/quickstart/cloud-quickstart/index.html) for more information.

## Confluent Operator

If you want to run Confluent's Kafka platform yourself, you can use the [Confluent Operator](https://docs.confluent.io/current/installation/operator/index.html), which can provision Kafka clusters on Kubernetes Engine. See [Confluent Platform on Google Kubernetes Engine documentation](https://docs.confluent.io/current/tutorials/examples/kubernetes/gke-base/docs/index.html#quickstart-demos-operator-gke) for more detail.

## Strimzi Operator

You can run Kafka in Kubernetes using the [Strimzi Operator](https://strimzi.io). See [Strimzi Quickstart ](https://strimzi.io/quickstarts/)documentation and the more detailed [Strimzi Quick Start Guide](https://strimzi.io/docs/operators/latest/quickstart.html) for more information.  
