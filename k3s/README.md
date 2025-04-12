k3s Lightweight Kubernetes
==========================

Installation
------------

```bash
curl -sfL https://get.k3s.io | sh -
```

export dojocd=~/REPO/dojo.cd   # path to repo


export DOMAIN=$(hostname -d)

Create a public ingress class
kubectl apply -f $dojocd/k3s/public-ingressclass.yaml



#### Ingress customization
oc apply -f $dojocd/k3s/traefik/serverstransport-backend-untrusted-tls.yaml
oc apply -f $dojocd/k3s/traefik/traefik-https-redirect-middleware.yaml


Operator Lifecycle Manager (OLM)
--------------------------------
Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.

```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.31.0/install.sh | bash -s v0.31.0
```

```bash
export dojocdpath=~/dojocd
OLMinstall=$dojocdpath/k3s/OLM/install.sh

curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.31.0/install.sh -o $OLMinstall

bash $OLMinstall v0.31.0
```

Operators
---------

### cert-manager

```bash
kubectl create -f https://operatorhub.io/install/cert-manager.yaml

# Create Cluster Issuer
cat $dojocd/k3s/cert-manager-cluster-issuer.yaml         | envsubst | kubectl apply -f-
cat $dojocd/k3s/cert-manager-cluster-issuer-staging.yaml | envsubst | kubectl apply -f-
```

### ArgoCD


```bash
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml

kubectl get csv -n operators # watch your operator come up

kubectl create -f $dojocd/argocd/config/argocd-instance.yaml

cat $dojocd/argocd/config/traefik-ingress-routes.yaml | envsubst | kubectl apply -f-  # create traefik ingress routes
```
