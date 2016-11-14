Instructions of running Kafka in CfC

Contact liguangc@cn.ibm.com for any comments for questions

# Setup zookeeper

Setup zookeeper service through the instructions under "zookeeper-deployment" or "zookeeper-petset" directory in this repository, make sure zookeeper is up and running before proceeding

# Create kafka cluster

kubectl create -f kafka-service.yaml
kubectl create -f kafka-deployment.yaml

# Verify Kafka is running correctly

kubectl exec into any kafka-deployment POD, run the following commands:


```
/opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --list --zookeeper zoo-1:2181 zoo-2:2181 zoo-3:2181
/opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --create --zookeeper zoo-1:2181 zoo-2:2181 zoo-3:2181 --replication-factor 1 --partitions 1 --topic testtopic
/opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --list --zookeeper zoo-1:2181 zoo-2:2181 zoo-3:2181
```


Then kubectl exec into another kafka-deployment POD, run the following command to make sure the testtopic is in the list:

```
/opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --list --zookeeper zoo-1:2181 zoo-2:2181 zoo-3:2181
```

