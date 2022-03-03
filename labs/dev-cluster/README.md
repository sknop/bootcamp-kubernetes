# Deploying a development Confluent Platform cluster

![image](https://user-images.githubusercontent.com/3109377/153306660-b0104933-ce44-4442-8558-257c324ea4fe.png)

### Goals

* Setting up a basic Confluent Platform (CP) cluster with CFK.
* Understanding CP CRDs transformation into native primitive resources. 
* Troubleshooting with Confluent kubectl plugin.
* Performing common Kafka operations with CFK. 
* Resizing, snapshotting and migration of storage volumes. 

### Dependencies 

Server-side components: 

* **AWS EKS**
* [**Confluent For Kubernetes operator**](https://github.com/confluentinc/confluent-operator/blob/master/charts/README.md)
* [**Prometheus**](https://artifacthub.io/packages/helm/prometheus-community/prometheus)
* [**Grafana**](https://github.com/grafana/helm-charts)
* [**EBS CSI Driver**](https://github.com/kubernetes-sigs/aws-ebs-csi-driver)

Client-side components: 

* [**helm:**](https://helm.sh/) package manager with templating support for Kubernetes.  
* [**eksctl:**](https://eksctl.io/) CLI utility tool to easily manage and provision EKS clusters.
* [**kubectl:**](https://kubernetes.io/docs/tasks/tools/) CLI Kubernetes client.
* [**krew:**](https://krew.sigs.k8s.io/) Kubectl plugins manager.   
* [**CFK plugin:**](https://github.com/confluentinc/confluent-operator#install-kubectl-plugin) Kubectl Confluent plugin for CFK.
* [**Kustomize**:](https://kustomize.io/) Kubernetes native tool for configuration management.
* [**Go:**](https://go.dev/doc/install) Golang compiler and toolchain necessary to build Confluent plugin. 

### Tasks

1. Deploy CP cluster without security on a new namespace. 
2. Check the deployment status using the Confluent kubectl plugin.
3. Visualize in your terminal all the k8s native resources (e.g., Services, Pods, Secrets, etc) that originated as a result of the deployment.
4. Get the Confluent Server shared and per-instance configuration running on k8s.
5. Scale-out the number of brokers to five replicas. 
6. Check the health of your cluster on Control Center UI. 
7. Remove one node from the cluster in an orderly manner. 
8. Check the health of your cluster on Grafana.
9. Create a topic and run a producer performance test.
10. Clone the cluster data and restore it to a new cluster. 

### Evaluation 

![image](https://user-images.githubusercontent.com/3109377/153624746-4a988ae5-26d8-4354-9842-5871fcc0628c.png)

![image](https://user-images.githubusercontent.com/3109377/153685189-1eb8fd13-cba1-42f0-8cb9-fd2f8ee24850.png)


![image](https://user-images.githubusercontent.com/3109377/153683421-13d00f01-608f-4875-9dec-4e06a085e694.png)

![Screenshot 2022-02-11 at 23 50 38](https://user-images.githubusercontent.com/3109377/153685831-870b4a5d-f4bd-4bdb-b6ec-2bb31e885681.png)

![image](https://user-images.githubusercontent.com/3109377/153686761-e528d081-148f-4d01-8d4b-8f4e6e31e759.png)

![image](https://user-images.githubusercontent.com/3109377/153686855-5e541335-f87a-4ef1-8159-06ff8d4a927c.png)

Check that there are no partitions assigned to the removed broker. 

```bash
k exec -it kafka-4 -- bash

bash-4.4$ ls /mnt/data/data0/logs/
cleaner-offset-checkpoint    meta.properties                   replication-offset-checkpoint
log-start-offset-checkpoint  recovery-point-offset-checkpoint
```

Scale down the cluster to four brokers.

```
k scale kafka kafka --replicas=4
kafka.platform.confluent.io/kafka scaled
```

Note: Control Center keeps a record of the existence broker-4 but there are no URPs so the cluster is happy. 

![image](https://user-images.githubusercontent.com/3109377/154800724-789d7a85-8881-416d-a4e3-148a4e34b3fb.png)

