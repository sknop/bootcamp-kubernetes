![image](https://user-images.githubusercontent.com/3109377/172832027-985b3cc5-ce24-4d8c-b975-b40c1c7c0ad4.png)

# Multi-Regional Cluster with partial network partition

Reproduces a Kafka MRC into one EKS cluster using separated namespaces and customised labelling in the worker nodes. 

## pre-requisites

* CFK operator
* Chaos Mesh operator
* Kubectl Confluent Plugin

## setup 

1. Label nodes with `confluent.io/west` and `confluent.io/east` labels to create separate logical regions 

```bash
kubectl label node ip-172-30-5-167.eu-west-1.compute.internal confluent.io/west=true
kubectl label node ip-172-30-37-91.eu-west-1.compute.internal confluent.io/east=true

#[... continue with all the remaining nodes ...]

```

2. Label nodes in each logical region with artificial `confluent.io/rack` availability zones

```bash
kubectl label node ip-172-30-5-167.eu-west-1.compute.internal  confluent.io/rack=eu-west-a
kubectl label node ip-172-30-37-91.eu-west-1.compute.internal  confluent.io/rack=eu-east-a

#[... continue for eu-west-b, eu-east-b, eu-west-c and eu-east-c ...]

```

3. Deploy manifest file with MRC cluster configuration 

```bash
kubectl apply -f mrc.yaml
```

4. Validate brokers are spread across all AZs by checking the `broker.rack` configuration 

```bash
for i in `seq 0 2`; do kubectl confluent cluster kafka instance-config --pod-name kafka-$i -n kafka-east | grep broker; done

### expected output example kafka-east namespace
broker.id=0
broker.rack=eu-east-a
broker.id=1
broker.rack=eu-east-c
broker.id=2
broker.rack=eu-east-b

for i in `seq 0 2`; do kubectl confluent cluster kafka instance-config --pod-name kafka-$i -n kafka-west | grep broker; done

### expected output example for kafka-west namespace
broker.id=10
broker.rack=eu-west-c
broker.id=11
broker.rack=eu-west-a
broker.id=12
broker.rack=eu-west-b
```

5. Create your placement policy 

```json
{
  "observerPromotionPolicy": "under-min-isr",
  "version": 2,
  "replicas": [
    {
      "count": 1,
      "constraints": {
        "rack": "eu-east-a"
      }
    },
    {
      "count": 1,
      "constraints": {
        "rack": "eu-east-b"
      }
    },
    {
      "count": 1,
      "constraints": {
        "rack": "eu-west-a"
      }
    },
    {
      "count": 1,
      "constraints": {
        "rack": "eu-west-b"
      }
    }
  ],
  "observers": [
    {
      "count": 1,
      "constraints": {
        "rack": "eu-west-c"
      }
    },
    {
      "count": 1,
      "constraints": {
        "rack": "eu-east-c"
      }
    }
  ]
}

```

5. Create a topic using the previous placement policy 

```bash
# copy the policy to one of the broker pods
kubectl -n kafka-west cp placement.json kafka-2:/tmp
# execute kafka-topics in the broker pod
kubectl -n kafka-west exec -it kafka-2 -- kafka-topics --bootstrap-server localhost:9092 --create --topic perf-topic-obs --partitions 15 --replica-placement /tmp/placement.json --config min.insync.replicas=3

# validate observers and replicas by describing the topic 
kubectl -n kafka-west exec -it kafka-2 -- kafka-topics --bootstrap-server localhost:9092 --describe --topic perf-topic-obs

Topic: perf-topic-obs   PartitionCount: 15      ReplicationFactor: 6    Configs: min.insync.replicas=3,segment.bytes=1073741824,message.format.version=2.6-IV0,confluent.placement.constraints={"observerPromotionPolicy":"under-min-isr","version":2,"replicas":[{"count":1,"constraints":{"rack":"eu-east-a"}},{"count":1,"constraints":{"rack":"eu-east-b"}},{"count":1,"constraints":{"rack":"eu-west-a"}},{"count":1,"constraints":{"rack":"eu-west-b"}}],"observers":[{"count":1,"constraints":{"rack":"eu-west-c"}},{"count":1,"constraints":{"rack":"eu-east-c"}}]}
        Topic: perf-topic-obs   Partition: 0    Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 1    Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 2    Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 3    Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 4    Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 5    Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 6    Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 7    Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 8    Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 9    Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 10   Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 11   Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 12   Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 13   Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
        Topic: perf-topic-obs   Partition: 14   Leader: 0       Replicas: 0,2,11,12,10,1        Isr: 12,11,0,2  Offline:        Observers: 10,1
```

## Happy path with Automatic Observer Promotion

1. Deploy a producer client for the `perf-topic-obs` topic. Tail the logs to observe workloads graceful failover

```bash
kubectl apply -f client.yaml

# you can kubectl logs instead if you prefer to
stern kafka-producer -n kafka-west
```

2. Scale down the `kafka-east` cluster to simulate the failure of the datacenter. 

```bash
# prevent CFK to apply reconciliation to our changes
kubectl -n kafka-east annotate --overwrite kafka kafka platform.confluent.io/block-reconcile="true"
# remove pod disruption budget so we can scale to zero
kubectl -n kafka-east delete poddisruptionbudgets kafka 
# scale down the east cluster
kubectl -n kafka-east scale sts/kafka --replicas=0 
```

3. Check automatic observer promotion happened by describing the topic or using Control Center 

```bash
kubectl -n kafka-west exec -it kafka-2 -- kafka-topics --bootstrap-server localhost:9092 --describe --topic perf-topic-obs

# as an alternative fire Control Center dashboard
kubectl -n kafka-west confluent dashboard controlcenter 
```

## Partial network failure 

1. Deploy a producer client for the `perf-topic-obs` topic. Tail the logs to observe network failures and retries

```bash
kubectl apply -f client.yaml

# you can kubectl logs instead if you prefer to
stern kafka-producer -n kafka-west
```

2. Apply NetworkChaos rules to create the partial network partition between brokers in each region

```bash
kubectl apply -f mrc-network-partition.yaml 
```

3. Validate that the ISRs remain the same despite the availability being compromised. Automatic Observer Promotion is not triggered. 

```bash
kubectl -n kafka-west exec -it kafka-2 -- kafka-topics --bootstrap-server localhost:9092 --describe --topic perf-topic-obs

# as an alternative fire Control Center dashboard
kubectl -n kafka-west confluent dashboard controlcenter 
```

### Why observers are not being promoted under a partial network failure? 

* Zookeeper connectivity is not affected so the Controller does not see any membership changes. No need for new partition leader elections or metadata updates. 

* On top of that, the leaders cannot communicate with the cluster Controller due to the network failure so the ISRs remain the same. According to [KIP-497](https://cwiki.apache.org/confluence/display/KAFKA/KIP-497%3A+Add+inter-broker+API+to+alter+ISR) this is the safest behaviour if the Controller cannot acknowledge the ISR shrink operation.

`The leader will continue relying on the previous ISR until the LeaderAndIsr with the successful change has been received.`

### What happens if the cluster Controller and the partitions leaders are co-located?

* Observers are not automatically promoted either because the Controller needs to communicate with the isolated replicas in order to shrink the ISR. 

In both scenarios, as soon as there is any change on the cluster membership (e.g. we shutdown the isolated replicas), the Controller removes the offline replicas from the ISR and promotes the observer.
