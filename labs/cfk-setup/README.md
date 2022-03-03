# Setting up Confluent For Kubernetes operator

![image](https://user-images.githubusercontent.com/3109377/153290401-b6a94bc6-bc22-43f0-ba3d-582dd81815f1.png)

### Goals

* Performing a new CFK installation on a Kubernetes (k8s) cluster
* Getting familiarity with foundational tools from the Kubernetes ecosystem  
* Assessing CFK installation with client-side CFK plugin


### Dependencies 

Server-side components: 

* **AWS EKS**
* [**Confluent For Kubernetes operator**](https://github.com/confluentinc/confluent-operator/blob/master/charts/README.md)

Client-side components: 

* [**helm:**](https://helm.sh/) package manager with templating support for Kubernetes.  
* [**eksctl:**](https://eksctl.io/) CLI utility tool to easily manage and provision EKS clusters.
* [**kubectl:**](https://kubernetes.io/docs/tasks/tools/) CLI Kubernetes client.
* [**krew:**](https://krew.sigs.k8s.io/) Kubectl plugins manager.   
* [**CFK plugin:**](https://github.com/confluentinc/confluent-operator#install-kubectl-plugin) Kubectl Confluent plugin for CFK.
* [**Kustomize**:](https://kustomize.io/) Kubernetes native tool for configuration management.
* [**Go:**](https://go.dev/doc/install) Golang compiler and toolchain necessary to build Confluent plugin. 

Optional but recommended: 

* **kubectl plugins:** my kubectl setup is not completed without the following plugins:

	* **lineage:** simplifies the tracking of dependencies associated with k8s resources. Quite useful to understand what k8s native objects emerge as a result of a CRD (e.g., CFK manifest for Kafka).     
	
* [**stern:**](https://github.com/wercker/stern) Kubernetes logging utility with improved functionality over `kubectl logs` and some similar options to `journalctl`.  
* [**kubens/kubectx:**](https://github.com/ahmetb/kubectx) CLI utilities to easily switch namespaces and clusters context respectively. 
  

### Tasks

1. Install all the client-side prerequisites.
2. Provision your EKS cluster on our AWS account.
3. Sort out your `.kubeconfig` file and switch context to point to your new cluster.
4. Run CFK pre-checks using the kubectl Confluent plugin.
5. Create a new k8s namespace on your cluster for the operator. 
6. In that namespace, create a `kubernetes.io/dockerconfig` secret type with the credentials to access our private container image repository: JFrogArtifactory. This is not necessary if you plan to retrieve CFK from a public repository. 
7. Install CFK in the operator namespace using Helm.
8. Install Cert-Manager in a different namespace using Helm.
9. Generate your own CA and private key files and create a `kubernetes.io/tls` secret type with the name `ca-pair-ssl` on Cert-Manager's namespace.

### Evaluation 

Some quick checks that you can do to assess your installation: 

* Check that CFK pod is running and health checks passing

```bash
export CKF_NAMESPACE=confluent
kubectl get pods -n $CKF_NAMESPACE --field-selector=status.phase=Running

NAME                                  READY   STATUS    RESTARTS   AGE
confluent-operator-74c987d595-hgk5s   1/1     Running   0          18h
```

* Check CFK logs

```bash
stern confluent-operator -A
```

* Check CFK CRDs have been installed in the cluster

```bash
kubectl get crds  | grep platform.confluent.io
 
confluentrolebindings.platform.confluent.io   2021-08-03T15:39:28Z
connectors.platform.confluent.io              2021-10-05T15:48:14Z
connects.platform.confluent.io                2021-08-03T15:39:29Z
controlcenters.platform.confluent.io          2021-08-03T15:39:29Z
kafkarestclasses.platform.confluent.io        2021-08-03T15:39:29Z
kafkarestproxies.platform.confluent.io        2021-10-05T15:48:18Z
kafkas.platform.confluent.io                  2021-08-03T15:39:29Z
kafkatopics.platform.confluent.io             2021-08-03T15:39:30Z
ksqldbs.platform.confluent.io                 2021-08-03T15:39:30Z
migrationjobs.platform.confluent.io           2021-08-03T15:39:30Z
schemaregistries.platform.confluent.io        2021-08-03T15:39:31Z
schemas.platform.confluent.io                 2021-10-05T15:48:21Z
zookeepers.platform.confluent.io              2021-08-03T15:39:31Z
``` 

* Deploy a minimal Kafka cluster configuration on the `confluent` namespace.

```bash
kubectl apply -f https://raw.githubusercontent.com/confluentinc/confluent-kubernetes-examples/master/quickstart-deploy/confluent-platform-singlenode.yaml

zookeeper.platform.confluent.io/zookeeper created
kafka.platform.confluent.io/kafka created
connect.platform.confluent.io/connect created
ksqldb.platform.confluent.io/ksqldb created
controlcenter.platform.confluent.io/controlcenter created
schemaregistry.platform.confluent.io/schemaregistry created

### check confluent platform status

kubectl confluent status -n confluent

### remove the deployment 

kubectl delete -f https://raw.githubusercontent.com/confluentinc/confluent-kubernetes-examples/master/quickstart-deploy/confluent-platform-singlenode.yaml
```

* Try to request a new cluster issuer and certificate in the `default` namespace

```bash
### create cluster wide issuer 

kubectl apply -n default -f - <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: ca-issuer
spec:
  ca:
    secretName: ca-pair-sslcerts
EOF

clusterissuer.cert-manager.io/ca-issuer created

### create certificate 

kubectl apply -n default -f - <<EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: testing-cert
spec:
  secretName: tls-testing-cert
  duration: 2160h # 90d
  renewBefore: 360h # 15d
  subject:
    organizations:
      - confluent
  commonName: nginx.default.svc.cluster.local
  isCA: false
  privateKey:
    algorithm: RSA
    encoding: PKCS1
    size: 2048
  usages:
    - server auth
  dnsNames:
    - nginx.default.svc.cluster.local
    - "*.nginx.default.svc.cluster.local"
  issuerRef:
    name: ca-issuer
    kind: ClusterIssuer
    group: cert-manager.io
EOF

certificate.cert-manager.io/testing-cert created

### check that a new secret containing the certificate has been created

kubectl get secrets tls-testing-cert -n default -o yaml

apiVersion: v1
data:
  ca.crt: <redacted>
  tls.crt: <redacted>
  tls.key: <redacted>
kind: Secret
metadata:
  annotations:
    cert-manager.io/alt-names: nginx.default.svc.cluster.local,*.nginx.default.svc.cluster.local
    cert-manager.io/certificate-name: testing-cert
    cert-manager.io/common-name: nginx.default.svc.cluster.local
    cert-manager.io/ip-sans: ""
    cert-manager.io/issuer-group: cert-manager.io
    cert-manager.io/issuer-kind: ClusterIssuer
    cert-manager.io/issuer-name: ca-issuer
    cert-manager.io/uri-sans: ""
  creationTimestamp: "2022-02-09T16:49:59Z"
  name: tls-testing-cert
  namespace: default
  resourceVersion: "67515690"
  uid: 368b2052-5b27-4d2f-8ff9-38d5cdea30bd
type: kubernetes.io/tls
```


