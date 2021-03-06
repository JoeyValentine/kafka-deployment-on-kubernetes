# Deploying Kafka with Strimzi(0.23.0)

This tutorial shows you how to run Apache Kafka on Kubernetes.  



## Creating a local StorageClass for Kafka

If you are using a different type of PV, you can skip this process.  

```
kubectl apply -f ./manifests/local-storage-class.yaml
```


## Creating a local persistent volume for Kafka

If you are using a different type of PV, you can skip this process.
In the case of local PV, it must be created for each node.  

```
kubectl apply -f ./manifests/local-persistent-volume.yaml
```


## Creating a namespace for Kafka

```
kubectl create namespace strimzi
```


## Add the Strimzi Helm Chart repository:

```
helm repo add strimzi https://strimzi.io/charts/
```


## To install the chart:

```
helm install --generate-name -n strimzi strimzi/strimzi-kafka-operator
```


## Create the Kafka cluster

```
kubectl apply -n strimzi -f ./manifests/kafka-ephemeral.yaml
```


## Add Prometheus and Grafana

You can use Prometheus to provide monitoring data for the example 
[Grafana dashboards](https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples/metrics/grafana-dashboards) 
provided with Strimzi.  
You need to know the matching labels of the Pod Monitor Selector by entering the following command.  

```
kubectl describe prometheus -n monitoring
```

```
Pod Monitor Selector:
  Match Labels:
    Release:  kube-prometheus-stack
```

Modify value of namespaceSelector.matchNames property to strimzi in strimzi-pod-monitor.yaml.  
And add `Release: kube-prometheus-stack` labels to each PodMonitor in strimzi-pod-monitor.yaml.

```
kubectl apply -n monitoring -f ./manifests/strimzi-pod-monitor.yaml
```


## How to run Kafka benchmark

Get the node port number of the external bootstrap service (replace my-cluster with the name of your cluster):  

```
kubectl get service --namespace=strimzi my-cluster-kafka-external-bootstrap -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}'
```

kafka-topics.sh

```
bin/kafka-topics.sh \
  --bootstrap-server KAFKA_IP:KAFKA_PORT \
  --create \
  --topic test \
  --partitions 6 \
  --replication-factor 1
```

kafka-consumer-perf-test.sh

```
bin/kafka-consumer-perf-test.sh \
  --bootstrap-server KAFKA_IP:KAFKA_PORT \
  --messages 5000000 \
  --topic test \
  --consumer.config ./consumer.properties
```

kafka-producer-perf-test.sh

```
bin/kafka-producer-perf-test.sh \
  --topic test \
  --num-records 5000000 \
  --record-size 2000 \
  --throughput -1 \
  --producer-props bootstrap.servers=KAFKA_IP:KAFKA_PORT \
  --producer.config ./producer.properties
```


## Cleaning up

1. Run the following command to get the release name of the kafka namespace  

```
helm ls -n strimzi
```

2. To uninstall/delete the `RELEASE_NAME` deployment:  

```
helm delete -n strimzi RELEASE_NAME
```
   
3. You have to manually delete related CRDs.

```
kubectl delete crd kafkabridges.kafka.strimzi.io         
kubectl delete crd kafkaconnectors.kafka.strimzi.io                      
kubectl delete crd kafkaconnects.kafka.strimzi.io                        
kubectl delete crd kafkaconnects2is.kafka.strimzi.io                     
kubectl delete crd kafkamirrormaker2s.kafka.strimzi.io                   
kubectl delete crd kafkamirrormakers.kafka.strimzi.io                    
kubectl delete crd kafkarebalances.kafka.strimzi.io                      
kubectl delete crd kafkas.kafka.strimzi.io                               
kubectl delete crd kafkatopics.kafka.strimzi.io                          
kubectl delete crd kafkausers.kafka.strimzi.io
```
