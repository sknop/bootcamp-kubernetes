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

### Useful 
