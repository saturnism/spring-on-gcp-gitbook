# Internal Load Balancing

This section continues from the previous section - make sure you do the tutorial in sequence.

{% page-ref page="../service.md" %}

## In-Cluster Load Balancer

A [Kubernetes Service](../service.md#service) acts as an internal L4 load balancer only accessible from within the same Kubernetes Cluster. See the [Service section](../service.md#service) for more information.

## Internal Network Load Balancer

The setup of the Internal Network Load Balancer is similar to the [External Network Load Balancer](external-load-balancing.md#external-network-load-balancer), but with an additional annotation.

## Internal HTTP\(s\) Load Balancer

