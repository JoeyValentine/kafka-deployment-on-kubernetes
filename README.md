# Deploying Kafka with Strimzi(0.23.0)
This tutorial shows you how to run Apache Kafka on Kubernetes.

# Creating a local StorageClass for Kafka
If you are using a different type of PV, you can skip this process.  
<code>
kubectl apply -f ./manifests/local-storage-class.yaml
</code>

# Creating a local persistent volume for Kafka
If you are using a different type of PV, you can skip this process.
In the case of local PV, it must be created for each node.  
<code>
kubectl apply -f ./manifests/local-persistent-volume.yaml
</code>

# Creating a namespace for Kafka
<code>
kubectl create namespace strimzi
</code>

# Add the Strimzi Helm Chart repository:
<code>
helm repo add strimzi https://strimzi.io/charts/
</code>

# To install the chart:
<code>
helm install --generate-name -n strimzi strimzi/strimzi-kafka-operator
</code>

# Create the Kafka cluster
<code>
kubectl apply -n strimzi -f ./manifests/kafka-ephemeral.yaml
</code>

# Add Prometheus and Grafana
You can use Prometheus to provide monitoring data for the example 
[Grafana dashboards](https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples/metrics/grafana-dashboards) 
provided with Strimzi. You need to know the matching labels of the Pod Monitor Selector by entering the following command.  
<code>
kubectl describe prometheus -n monitoring
</code>

```
Pod Monitor Selector:
  Match Labels:
    Release:  kube-prometheus-stack
```

Modify value of namespaceSelector.matchNames property to strimzi in strimzi-pod-monitor.yaml. 
And add <code>Release: kube-prometheus-stack</code> labels to each PodMonitor in strimzi-pod-monitor.yaml.

<code>
kubectl apply -n monitoring -f ./manifests/strimzi-pod-monitor.yaml
</code>

# How to run cassandra-stress benchmark

Get the node port number of the external bootstrap service (replace my-cluster with the name of your cluster):  

```
kubectl get service my-cluster-kafka-external-bootstrap \
  -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}'
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

# Cleaning up

1. Run the following command to get the release name of the kafka namespace  
<code>
helm ls -n strimzi
</code>

2. To uninstall/delete the <code>RELEASE_NAME</code> deployment:  
<code>
helm delete -n strimzi RELEASE_NAME
</code>
   
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
