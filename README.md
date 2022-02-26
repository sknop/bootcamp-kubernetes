# bootcamp-kubernetes
Scripts and setup files for Kubernetes Bootcamp

### Kubeconfig setup

```bash
eksctl utils write-kubeconfig --cluster sknop-bootcamp-cluster
```

Hierarchical Namespaces

```bash
kubectl apply -f https://github.com/kubernetes-sigs/multi-tenancy/releases/download/hnc-v0.8.0/hnc-manager.yaml
```

Hashicorp Vault

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update 
helm upgrade --create-namespace --install vault --set='server.dev.enabled=true' hashicorp/vault --namespace vault
```

Monitoring

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install --create-namespace --namespace monitoring  prometheus prometheus-community/kube-prometheus-stack
```

External DNS for Route53 integration 

```bash
helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/
helm repo update
helm upgrade --install --create-namespace -n external-dns external-dns external-dns/external-dns --set extraArgs="{--aws-zone-type=private}"

eksctl create iamserviceaccount \
    --name external-dns \
    --namespace external-dns \
    --cluster sknop-bootcamp-cluster \
    --attach-policy-arn arn:aws:iam::492737776546:policy/EKSExternalDNSBootcampPolicy \
    --approve \
    --override-existing-serviceaccounts
```

Certificate Manager 

```
helm repo add jetstack https://charts.jetstack.io
helm repo update
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.crds.yaml
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.4 \
```

OpenLDAP 

```
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install \ 
  bootcamp stable/openldap \
  --namespace openldap \  
  --create-namespace \
  -f configurations/ldap/values.yaml 
```

EBS CSI Drivers 

```
curl -o example-iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-ebs-csi-driver/master/docs/example-iam-policy.json

aws iam create-policy \
    --policy-name AmazonEKS_EBS_CSI_Driver_Policy \
    --policy-document file://example-iam-policy.json

eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --override-existing-serviceaccounts \
    --namespace kube-system \
    --cluster sknop-bootcamp-cluster \
    --attach-policy-arn arn:aws:iam::492737776546:policy/AmazonEKS_EBS_CSI_Driver_Policy \
    --approve \
    --role-only

eksctl create addon --name aws-ebs-csi-driver --cluster sknop-bootcamp-cluster --service-account-role-arn arn:aws:iam::492737776546:role/CSIEBSBootcampRole
 --force
```

AWS Load Balancer Controller

```
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.0/docs/install/iam_policy.json

aws iam create-policy \
   --policy-name AWSLoadBalancerControllerIAMPolicy \
   --policy-document file://iam_policy.json

eksctl create iamserviceaccount \
    --name aws-load-balancer-controller-sa \
    --override-existing-serviceaccounts \
    --namespace kube-system \
    --cluster sknop-bootcamp-cluster \
    --attach-policy-arn arn:aws:iam::492737776546:policy/AWSLoadBalancerControllerIAMPolicy \
    --approve

helm repo add eks https://aws.github.io/eks-charts
helm repo update

kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    --set clusterName=sknop-bootcamp-cluster \
    --set serviceAccount.create=false \
    --set region=eu-west-1 \
    --set vpcId=vpc-08da1069e2646f90f \
    --set serviceAccount.name=aws-load-balancer-controller-sa \
    -n kube-system
```

NGINX Ingress Controller

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm upgrade -n kube-system --install ingress-nginx ingress-nginx/ingress-nginx \
  --set controller.extraArgs.enable-ssl-passthrough="true"
```
