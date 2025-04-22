microk8s Lightweight Kubernetes
===============================

Installation
------------

Install Ubuntu 24.04

```bash
apt update
apt upgrade -y
apt install git -y
apt install yamllint -y # optional
```

Install MicroK8s

```bash
snap install microk8s --classic --channel=1.32/stable
microk8s status --wait-ready
```

Enable addons

```bash 
microk8s enable ingress
microk8s enable hostpath-storage
```

Create user


```bash
useradd -m -s /bin/bash -G microk8s kube

cat <<EoF >> /home/kube/.bashrc
###################################
export dojocd=/home/kube/dojocd
export DOMAIN=zulu.ar
export DOMAIN_TSNET=your-domain.ts.net
export INGRESSCLASS=public
alias kubectl='microk8s kubectl'
alias kc='microk8s kubectl'
###################################
EoF
```

```bash
cat <<EoF > /usr/local/bin/kubectl
#!/bin/sh
exec /snap/bin/microk8s kubectl "\$@"
EoF

chmod a+x /usr/local/bin/kubectl
```

Pull Repo

```bash
su - kube
## Create o restore ssh key
git clone git@github.com:downbot/dojo.cd.git ${dojocd:?define repo path}
cd $dojocd
```


Operator Lifecycle Manager (OLM)
--------------------------------
Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.

```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.31.0/install.sh | bash -s v0.31.0
```

```bash
OLMinstall=$dojocd/engine/k3s/OLM/install.sh

curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.31.0/install.sh -o $OLMinstall

bash $OLMinstall v0.31.0
```

Operators
---------

### cert-manager

```bash
kubectl create -f $dojocd/operator/cert-manager/subscription.yaml

kubectl get csv -n operators -w # watch operator come up 

# Create Cluster Issuer
cat $dojocd/operator/cert-manager/cluster-issuer.yaml         | envsubst | kubectl apply -f-
cat $dojocd/operator/cert-manager/cluster-issuer-staging.yaml | envsubst | kubectl apply -f-
```

### ArgoCD

Inatall ArgoCD

```bash
kubectl create -f $dojocd/operator/argocd/subscription.yaml

kubectl get csv -n operators -w # watch operator come up

kubectl create -f $dojocd/operator/argocd/argocd-instance.yaml ## require operator to install crd

cat $dojocd/operator/argocd/argocd-ingress-nginx.yaml | envsubst | kubectl apply -f-  # create ingress routes

oc extract --to=- secrets/argocd-cluster -n argocd  # Get admin password with oc
kubectl get secrets/argocd-cluster -n argocd -o jsonpath='{.data.admin\.password}' | base64 -d
```

Configure ArgoCD

```bash
kubectl apply -f $dojocd/operator/argocd/config/dojo-project.yaml
```

Deploy Applications

```bash
kubectl apply -f $dojocd/operator/argocd/config/dojo-cluster-applicationset.yaml
```


Tailscale VPN
-------------

Install Tailscale operator

```bash
kubectl apply -f $dojocd/operator/tailscale/install.yaml
```

Expose API server vía VPN

```bash
kubectl -n default annotate svc kubernetes tailscale.com/expose=true tailscale.com/hostname=api-server
```

