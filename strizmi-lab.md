# Strizmi Kafka Operator

## Installation

Create a kafka namepsace

`kubectl create namespace kafka`

Next we apply the Strimzi install files, including `ClusterRoles`, `ClusterRoleBindings` and some __Custom Resource Definitions (CRDs)__. The CRDs define the schemas used for declarative management of the Kafka cluster, Kafka topics and users.

`kubectl apply -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka`

To check on deplyment of the kafka cluster:

`kubectl get pod -n kafka --watch`

You can also follow the operator’s log:

`kubectl logs deployment/strimzi-cluster-operator -n kafka -f`

## Provision the Apache Kafka cluster

Here we create a new Kafka custom resource, which will give us a small persistent Apache Kafka Cluster with one node for each - Apache Zookeeper and Apache Kafka:

```bash
# Apply the `Kafka` Cluster CR file
kubectl apply -f https://strimzi.io/examples/latest/kafka/kafka-persistent-single.yaml -n kafka
```

We now need to wait while Kubernetes starts the required pods, services and so on:

`kubectl wait kafka/my-cluster --for=condition=Ready --timeout=1000s -n kafka`

The above command might timeout if you’re downloading images over a slow connection. If that happens you can always run it again.

## Send and receive messages

Once the cluster is running, you can run a simple producer to send messages to a Kafka topic (the topic will be automatically created):

`kubectl -n kafka run kafka-producer -ti --image=strimzi/kafka:0.19.0-kafka-2.5.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic my-topic`

And to receive them in a different terminal you can run:

`kubectl -n kafka run kafka-consumer -ti --image=strimzi/kafka:0.19.0-kafka-2.5.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning`