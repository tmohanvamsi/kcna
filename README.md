# kcna

Quick kubectl command cheat sheet and shortcuts for CKA/CKAD/KCNA/KCSA prep.

## Setup shortcuts

```sh
# Shell aliases
alias k=kubectl
alias kctx='kubectl config current-context'
alias kctxs='kubectl config get-contexts'
alias kctxu='kubectl config use-context'
alias kns='kubectl config set-context --current --namespace'
```

## Global flags

```sh
# Set namespace on any command
kubectl -n <NAMESPACE> ...

# Use a specific kubeconfig
kubectl --kubeconfig <PATH> ...
```

## Resource short names

```sh
po=pods        svc=services    deploy=deployments
rs=replicasets sts=statefulsets ds=daemonsets
cm=configmaps  sa=serviceaccounts ns=namespaces
rb=rolebindings crb=clusterrolebindings
pv=persistentvolumes pvc=persistentvolumeclaims
```

## Contexts and namespaces

```sh
kubectl config get-contexts
kubectl config current-context
kubectl config use-context <CONTEXT>
kubectl config set-context --current --namespace <NAMESPACE>
```

## Get and describe

```sh
kubectl get nodes
kubectl get ns
kubectl get pods -A
kubectl get po -n <NAMESPACE>
kubectl describe pod <POD> -n <NAMESPACE>
kubectl get events -n <NAMESPACE> --sort-by=.metadata.creationTimestamp
```

## Create and apply

```sh
kubectl apply -f <FILE.yaml>
kubectl apply -k <DIR>
kubectl create namespace <NAMESPACE>
kubectl create deploy <NAME> --image=<IMAGE>
kubectl expose deploy <NAME> --port=<PORT> --type=ClusterIP
```

## kubectl create (core resources)

```sh
# Pods
kubectl create pod <POD> --image=<IMAGE> -n <NAMESPACE>

# Deployments
kubectl create deployment <NAME> --image=<IMAGE> -n <NAMESPACE>

# ReplicaSets
kubectl create rs <NAME> --image=<IMAGE> -n <NAMESPACE>

# Services
kubectl create service clusterip <NAME> --tcp=<PORT>:<TARGET_PORT> -n <NAMESPACE>
kubectl create service nodeport <NAME> --tcp=<PORT>:<TARGET_PORT> -n <NAMESPACE>
kubectl create service loadbalancer <NAME> --tcp=<PORT>:<TARGET_PORT> -n <NAMESPACE>

# HPA
kubectl autoscale deployment <NAME> --min=2 --max=5 --cpu-percent=70 -n <NAMESPACE>

# RBAC
kubectl create role <ROLE> --verb=get,list,watch --resource=pods -n <NAMESPACE>
kubectl create rolebinding <RB> --role=<ROLE> --user=<USER> -n <NAMESPACE>
kubectl create clusterrole <CR> --verb=get,list,watch --resource=pods
kubectl create clusterrolebinding <CRB> --clusterrole=<CR> --user=<USER>

# PV and PVC
kubectl create pv <PV_NAME> --capacity=1Gi --access-modes=ReadWriteOnce --host-path=/data
kubectl create pvc <PVC_NAME> --storage-class=<SC> --access-modes=ReadWriteOnce --resources=requests.storage=1Gi -n <NAMESPACE>
```

## Edit and patch

```sh
kubectl edit deploy <NAME> -n <NAMESPACE>
kubectl patch deploy <NAME> -n <NAMESPACE> -p '{"spec":{"replicas":3}}'
```

## Logs and exec

```sh
kubectl logs <POD> -n <NAMESPACE>
kubectl logs <POD> -n <NAMESPACE> -c <CONTAINER>
kubectl logs -f <POD> -n <NAMESPACE>
kubectl exec -it <POD> -n <NAMESPACE> -- /bin/sh
```

## Scale and rollout

```sh
kubectl scale deploy <NAME> -n <NAMESPACE> --replicas=3
kubectl rollout status deploy <NAME> -n <NAMESPACE>
kubectl rollout history deploy <NAME> -n <NAMESPACE>
kubectl rollout undo deploy <NAME> -n <NAMESPACE>
```

## Delete

```sh
kubectl delete pod <POD> -n <NAMESPACE>
kubectl delete deploy <NAME> -n <NAMESPACE>
kubectl delete -f <FILE.yaml>
kubectl delete ns <NAMESPACE>
```

## RBAC (role + rolebinding from CLI)

```sh
# Role: read pods in a namespace
kubectl create role pod-reader \
  --verb=get,list,watch \
  --resource=pods \
  -n <NAMESPACE>

# RoleBinding: bind to a user
kubectl create rolebinding pod-reader-binding \
  --role=pod-reader \
  --user=<USER_NAME> \
  -n <NAMESPACE>

# RoleBinding: bind to a service account
kubectl create rolebinding pod-reader-binding \
  --role=pod-reader \
  --serviceaccount=<NAMESPACE>:<SA_NAME> \
  -n <NAMESPACE>
```

## Labels and selectors

```sh
kubectl label pod <POD> env=dev -n <NAMESPACE>
kubectl label pod <POD> env=dev --overwrite -n <NAMESPACE>
kubectl get pods -l env=dev -n <NAMESPACE>
kubectl label node <NODE> disktype=ssd
```

## Node maintenance

```sh
kubectl cordon <NODE>
kubectl drain <NODE> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <NODE>
```

## Cluster upgrade (kubeadm)

```sh
# Check versions
kubectl version --short
kubeadm version

# Control plane upgrade (example)
kubeadm upgrade plan
kubeadm upgrade apply v1.29.0

# Upgrade kubelet/kubectl on control plane node
apt-get update && apt-get install -y kubelet=1.29.0-00 kubectl=1.29.0-00
systemctl daemon-reload && systemctl restart kubelet

# Worker node upgrade (example)
kubeadm upgrade node
apt-get update && apt-get install -y kubelet=1.29.0-00 kubectl=1.29.0-00
systemctl daemon-reload && systemctl restart kubelet
```

## ETCD backup and restore (self-managed clusters)

```sh
# Snapshot backup
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Snapshot status
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db

# Restore (example)
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore
```

## Helm commands

```sh
helm repo add <NAME> <URL>
helm repo update
helm search repo <KEYWORD>
helm show values <REPO>/<CHART>
helm install <RELEASE> <REPO>/<CHART> -n <NAMESPACE> --create-namespace
helm upgrade <RELEASE> <REPO>/<CHART> -n <NAMESPACE>
helm rollback <RELEASE> <REVISION> -n <NAMESPACE>
helm uninstall <RELEASE> -n <NAMESPACE>
helm list -A
helm status <RELEASE> -n <NAMESPACE>
```

## Troubleshooting

```sh
kubectl get pod <POD> -n <NAMESPACE> -o wide
kubectl describe pod <POD> -n <NAMESPACE>
kubectl logs <POD> -n <NAMESPACE> --previous
kubectl get events -n <NAMESPACE> --sort-by=.lastTimestamp
```

## CNI notes (general + Weave)

- CNI = Container Network Interface; kubelet calls CNI plugins to attach pod networking.
- CNI config files live in `/etc/cni/net.d/`, binaries in `/opt/cni/bin/`.
- CNI decides pod IP assignment and routing so pods can talk across nodes.
- Weave Net is a CNI plugin that builds an overlay network (VXLAN) and includes IPAM.
- Weave runs as a DaemonSet in `kube-system` (`weave-net`), one pod per node.
- Weave supports NetworkPolicy; ensure policies are enabled in the cluster.
- Quick checks:
  - `kubectl get pods -n kube-system -l name=weave-net`
  - `kubectl logs -n kube-system -l name=weave-net --tail=200`


# serviceAccount

jq -R 'split(".") | select(length > 0) | .[0],.[1] | @base64d | fromjson' <<< eywrgfuwihufiweuhvcuenvui......

# secret

kubectl create secret docker-registry regcred \
  --docker-server=REGISTRY_URL \
  --docker-username=USERNAME \
  --docker-password=PASSWORD \
  --docker-email=EMAIL

## GitHub vs GitLab vs GitOps (quick compare)

- GitHub: code hosting + GitHub Actions for CI/CD; massive OSS ecosystem; strong marketplace.
- GitLab: code hosting + built-in CI/CD and DevSecOps features; single product for repo, CI, registry.
- GitOps: an operating model for deployments, not a hosting platform; desired state stored in Git.
- GitOps tools (e.g., Argo CD, Flux) sync clusters from Git and reconcile drift automatically.
- Typical flow: GitHub/GitLab host repos; GitOps tools deploy to Kubernetes from those repos.

## Helm chart: kube-prometheus-stack

```sh
# Add repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install (example namespace)
kubectl create namespace monitoring
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring

# Upgrade
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring

# Uninstall
helm uninstall kube-prometheus-stack -n monitoring
```


# Istio

curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH="$PWD/bin:$PATH"
istioctl version

export PATH="$HOME/istio-*/bin:$PATH"

istioctl install --set profile=demo -y

istioctl verify-install


istioctl version
istioctl analyze -A


kubectl label ns demo istio-injection=enabled
