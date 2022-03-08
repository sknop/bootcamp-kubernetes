# Deploying a production-ready Confluent Platform cluster

### Goals

* Setting up a production-ready Confluent Platform (CP) cluster with CFK.
* Generate TLS certificates with Cert-Manager and integrate with CFK.
* Different networking models to expose CP services externally.
* Integration with LDAP for RBAC.
* Defining new configuration variants with Kustomize overlays and components.

### Dependencies 

Server-side components: 

* **AWS EKS**
* [**Confluent For Kubernetes operator**](https://github.com/confluentinc/confluent-operator/blob/master/charts/README.md)
* [**Prometheus**](https://artifacthub.io/packages/helm/prometheus-community/prometheus)
* [**Grafana**](https://github.com/grafana/helm-charts)
* [**Cert-Manager**](https://cert-manager.io/docs/)
* [**External DNS**](https://github.com/kubernetes-sigs/external-dns)
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

1. Create a new namespace for the production cluster
2. Deploy a cluster using SASL/PLAIN over TLS and expose Confluent Server brokers using NodePorts.

![image](https://user-images.githubusercontent.com/3109377/155852071-05d119d7-01f0-49d5-935b-3199e744970d.png)

3. Try to access the brokers from a jumphost located inside the Bootcamp VPC. 
4. Annotate the nodePort service with `service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "ip"` and deploy the cluster again. What changes?
5. Modify the cluster to expose Confluent Server brokers using host-based routing and Ingress. 

![image](https://user-images.githubusercontent.com/3109377/155852043-d6d47f25-7148-4c14-b3aa-afbf7c9b881a.png)

6. Try to access the brokers using their domain name from a jumphost placed in the Bootcamp VPC. 
 
7. Deploy a cluster using SASL/PLAIN over TLS and expose Confluent Platform services externally with a load balancer. 

![image](https://user-images.githubusercontent.com/3109377/156514342-b9b45cd8-dc62-4ca8-81fd-d3db62edf144.png)

8. Try to access the different services from a jumphost located inside the Bootcamp VPC. 
9. Upgrade the cluster to work with RBAC and integrate with the existing LDAP server. 
10. Deploy a NetworkChaos rule to prevent access to OpenLDAP. Which services are broken and which ones are resilient? 

### Notes 

NodePorts communication relies on security groups to be configured accordingly. The image below shows how the k8s NodePorts range has been made accessible to any instance in the VPC CIDR range. 

![image](https://user-images.githubusercontent.com/3109377/155569452-c18cdd6e-a669-459e-ba36-230facc79aa5.png)

To make access to NodePorts service easier, a Network Load Balancer has been provisioned pointing to a TargetGroup containing all the nodes belonging to the k8s cluster.  

![Screenshot 2022-02-25 at 10 09 23](https://user-images.githubusercontent.com/3109377/155696708-6f08b7a9-5d98-4e0a-b2b5-b79edbb87b1c.png)

Check how TargetGroups differ when the load balancer targets node instances versus IPs

```
kubectl get TargetGroupBinding

NAME                              SERVICE-NAME         SERVICE-PORT   TARGET-TYPE   AGE
k8s-testing-kafka0np-b5b8b13211   kafka-0-np           9092           ip            6m47s
k8s-testing-kafka1np-99f664f296   kafka-1-np           9092           ip            6m47s
k8s-testing-kafka2np-f72aac3854   kafka-2-np           9092           ip            6m46s
k8s-testing-kafkaboo-e2216d1b11   kafka-bootstrap-np   9092           ip            6m47s


kubectl get TargetGroupBinding

NAME                              SERVICE-NAME         SERVICE-PORT   TARGET-TYPE   AGE
k8s-testing-kafka0np-2f6bc0f246   kafka-0-np           9092           instance      66s
k8s-testing-kafka1np-e79750acf8   kafka-1-np           9092           instance      66s
k8s-testing-kafka2np-97843cc761   kafka-2-np           9092           instance      64s
k8s-testing-kafkaboo-1c9f5cae46   kafka-bootstrap-np   9092           instance      66s

```


