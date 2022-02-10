# Deploying a development Confluent Platform cluster

![image](https://user-images.githubusercontent.com/3109377/153306660-b0104933-ce44-4442-8558-257c324ea4fe.png)

### Goals

* Setting up a basic Confluent Platform (CP) cluster with CFK.
* Understanding CP CRDs transformation into native primitive resources. 
* Troubleshooting with Confluent kubectl plugin.
* Performing common Kafka operations with CFK. 

### Dependencies 

Server-side components: 

* **AWS EKS**
* [**Confluent For Kubernetes operator**](https://github.com/confluentinc/confluent-operator/blob/master/charts/README.md)
* [**Prometheus**](https://artifacthub.io/packages/helm/prometheus-community/prometheus)
* [**Grafana**](https://github.com/grafana/helm-charts)

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
5. Scale out the number of brokers to five replicas. 
6. Check the health of your cluster on Control Center UI. 
7. Remove one node from the cluster in an orderly manner. 
8. Check the health of your cluster on Grafana.

### Evaluation


