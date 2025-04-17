k3s Lightweight Kubernetes
==========================

Installation
------------

Install Debian 12

```bash
apt update
apt upgrade -y
apt install curl git -y
apt install yamllint -y # optional
```

Install k3s

```bash
curl -sfL https://get.k3s.io | sh -s - --disable-cloud-controller --cluster-domain zulu.ar \
     --tls-san api-server --tls-san api-server.vaquita-morray.ts.net
```

Create user


```bash
useradd -m -s /bin/bash kube
install -D --mode 600 <(kubectl config view --raw) /home/kube/.kube/config
```

Pull Repo

```bash
su - kube
## Create o restore ssh key
git clone git@github.com:downbot/dojo.cd.git dojocd
cd dojocd
echo -e "\nexport dojocd=`pwd`\n" | tee -a ~/.bashrc
bash
```

export DOMAIN=$(hostname -d)
export DOMAIN=zulu.ar  # $(hostname -d)

`/etc/systemd/system/k3s.service`


#### Ingress customization
kubectl apply -f $dojocd/k3s/traefik/serverstransport-backend-untrusted-tls.yaml


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

Inatall ArgoCD

```bash
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml

kubectl get csv -n operators # watch your operator come up

kubectl create -f $dojocd/argocd/config/argocd-instance.yaml  ## require operator to install crd

cat $dojocd/argocd/config/traefik-ingress-routes.yaml | envsubst | kubectl apply -f-  # create traefik ingress routes

oc extract --to=- secrets/argocd-cluster -n argocd  # Get admin password with oc
kubectl get secrets/argocd-cluster -n argocd -o jsonpath='{.data.admin\.password}' | base64 -d
```

Configure ArgoCD

```bash
kubectl apply -f $dojocd/argocd/config/dojo-project.yaml
```


Tailscale VPN
-------------

```bash
kubectl apply -f $dojocd/cluster/tailscale/operator.yaml

kubectl -n default annotate svc kubernetes tailscale.com/expose=true tailscale.com/hostname=api-server
```

