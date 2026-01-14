# Kubernetes Aliases and Imperative Commands

## Table of Contents

- [Aliases](#aliases)
- [Apply Alias Changes](#apply-alias-changes)
- [Bash Completion](#bash-completion)
- [Kubectl Commands](#kubectl-commands)
  - [Pods](#pods)
  - [Deployments](#deployments)
  - [Services](#services)
    - [ClusterIP Service](#clusterip-service)
    - [NodePort Service](#nodeport-service)
    - [Endpoints](#endpoints)
  - [Jobs and CronJobs](#jobs-and-cronjobs)
  - [ConfigMaps and Secrets](#configmaps-and-secrets)
  - [ServiceAccounts](#serviceaccounts)
  - [RBAC](#rbac)
  - [Ingress](#ingress)
  - [Network Policies](#network-policies)
  - [Storage](#storage)
  - [Labels & Annotations](#labels--annotations)
  - [Events & Troubleshooting](#events--troubleshooting)
  - [Liveness & Readiness Probes](#liveness--readiness-probes)
  - [Metrics](#metrics)
- [Kubeconfig Commands](#kubeconfig-commands)
- [Networking Tests (curl / wget / nc)](#networking-tests-curl--wget--nc)
- [Control Plane Notes](#control-plane-notes)
- [File Copy Between Nodes](#file-copy-between-nodes)
- [Inspect Docker Images](#inspect-docker-images)

---

## Aliases

Add these to `~/.bashrc` or `~/.zshrc`:

```bash
export dr='--dry-run=client -o yaml'
export now='--grace-period 0 --force'  # use grace with force

alias k=kubectl
alias kn='k config set-context --current --namespace'
alias kaf='k apply -f'
alias kgp='k get po'
alias kgs='k get svc'
alias kc='k config get-contexts'
alias kg='k get'
alias kd='k describe'
alias ke='k explain'
alias ker='k explain --recursive'
alias krf='k replace --force --grace-period=0 -f'
````

---

## Apply Alias Changes

```bash
source ~/.bashrc
```

For all new terminals:

```bash
source ~/.bash_profile
```

---

## Bash Completion

```bash
source /etc/profile.d/bash_completion.sh
source <(kubectl completion bash)
alias k=kubectl
complete -F __start_kubectl k
```

**Why:** kubectl completion doesn’t work automatically for alias `k`.

---

## Kubectl Commands

## Pods

```bash
kubectl run <pod-name> \
--image=<img-name> \
--labels="key1=value1,key2=value2" \
--port=5701 \
--env="key1=value1" \
--env="key2=value2" \
-- <arg1> <arg2>
```

Create pod + service:

```bash
kubectl run <pod-name> \
--image=<img-name> \
--labels="key=value" \
--port=570 \
--expose=true
```

Pod status only:

```bash
kubectl get po pod1 -o jsonpath="{.status.phase}"
```

Describe status context:

```bash
kubectl describe po pod1 | grep -i status: -A 3 -B 3
```

---

## Deployments

```bash
kubectl create deployment <my-dep> --image=<img-name> --replicas=3
kubectl scale deployment <my-dep> --replicas=4
kubectl set image deployment <my-dep> <container-name>=nginx:1.17
kubectl rollout status deployment <my-dep>
kubectl rollout history deployment <my-dep>
kubectl rollout history deployment <my-dep> --revision=3
kubectl rollout undo deployment <my-dep>
kubectl rollout undo deployment <my-dep> --to-revision=2
k rollout restart deployment <my-dep>
kubectl set sa deployment <deployment-name> <sa-name>
```

Check deployment images:

```bash
kubectl get deploy <dep> -o json | jq -r '.spec.template.spec.containers[].image'
```

---

## Services

### ClusterIP Service

**Expose pod (recommended – uses pod labels):**

```bash
kubectl expose pod <pod-name> \
--port=6379 \
--name <svc-name> \
--dry-run=client -o yaml
```

**Create service (does NOT use pod labels):**

```bash
kubectl create service clusterip <svc-name> \
--tcp=<port>:<targetPort> \
--dry-run=client -o yaml
```

> This assumes selector `app=redis`. Modify selector before apply.

---

### NodePort Service

**Expose pod (cannot set nodePort):**

```bash
kubectl expose pod <pod-name> \
--port=80 \
--name <svc-name> \
--type=NodePort \
--dry-run=client -o yaml
```

**Create service (can set nodePort, no selectors):**

```bash
kubectl create service nodeport <svc-name> \
--tcp=<port>:<targetPort> \
--node-port=30080 \
--dry-run=client -o yaml
```

> Recommended: use `expose`, edit YAML, then add nodePort.

**Note:**
`3333:80` → `3333 = port`, `80 = targetPort`

---

### Endpoints

```bash
kubectl get ep -n <namespace>
```

---

## Jobs and CronJobs

```bash
kubectl create job <my-job> --image=<image-name>
kubectl create cronjob <my-job> \
--image=<image-name> \
--schedule="*/1 * * * *"
```

Cron format:

```
* * * * *
| | | | |
m h dom mon dow
```

Exclude jobTemplate:

```bash
kubectl explain cronjob.spec | grep -v jobTemplate
```

---

## ConfigMaps and Secrets

```bash
kubectl create configmap <my-config> \
--from-literal=key1=config1 \
--from-literal=key2=config2 \
--from-file=path \
--from-env-file=foo.env
```

```bash
kubectl create cm configmap --from-file=<key>=<path>
```

Secrets:

```bash
kubectl create secret generic <secret> \
--from-literal=key1=secret
```

Decode secret:

```bash
kubectl get secret <secret> -o jsonpath="{.data.token}" | base64 -d
```

Verify inside pod:

```bash
kubectl exec pod1 -- env | grep TREE1
kubectl exec pod1 -- cat /etc/birke/tree
kubectl exec pod1 -- cat /etc/birke/level
kubectl exec pod1 -- cat /etc/birke/department
```

---

## ServiceAccounts

```bash
kubectl create serviceaccount <sa>
```

---

## RBAC

```bash
kubectl create role <role> \
--namespace=<ns> \
--verb=* \
--resource=pods \
--dry-run=client -o yaml
```

```bash
kubectl create rolebinding <rb> \
--role=<role> \
--user=<user> \
--namespace=<ns> \
--dry-run=client -o yaml
```

Cluster-wide:

```bash
kubectl create clusterrole <cr> --verb=* --resource=pods
kubectl create clusterrolebinding <crb> \
--clusterrole=<cr> \
--user=<user> \
--group=<group>
```

Permission check:

```bash
kubectl auth can-i list pods --as mock-user
```

---

## Ingress

```bash
kubectl create ing <ingress-name> \
--rule="host/path=service:port" \
--annotation="nginx.ingress.kubernetes.io/rewrite-target=/" \
$dr > ingress.yaml
```

IngressClass:

```bash
kubectl get ingressclass
```

---

## Network Policies

```bash
k explain --recursive netpol
```

Testing:

```bash
kubectl run temp --rm -it --image=busybox \
--labels=criteria=allow -- wget svc:80
```

Redis test:

```bash
kubectl run temp --rm -it --image=busybox \
-- nc -zv redis-svc 6379
```

---

## Storage

```bash
k explain --recursive pv
k explain --recursive pvc
k explain --recursive sc
```

---

## Labels & Annotations

```bash
kubectl label po -l app=nginx env=prod
kubectl annotate po -l app=nginx owner=teamA
```

---

## Events & Troubleshooting

```bash
kubectl get events \
--field-selector involvedObject.name=<name>,type=Warning
```

Keep container alive:

```yaml
command: ["sh","-c","while true; do sleep 3600; done"]
```

---

## Liveness & Readiness Probes

Check config:

```bash
kubectl get pod <pod> -o jsonpath="{.spec.containers[*].readinessProbe}"
kubectl get pod <pod> -o jsonpath="{.spec.containers[*].livenessProbe}"
```

Check failures:

```bash
kubectl describe pod <pod>
kubectl get events | grep -i probe
```

Manual test:

```bash
kubectl run curl --rm -it --image=curlimages/curl \
-- curl http://pod-ip:port/health
```

---

## Metrics

```bash
kubectl top nodes
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory
```

---

## Kubeconfig Commands

```bash
kubectl config set-credentials <user> \
--client-certificate cert \
--client-key key

kubectl config set-context <ctx> \
--cluster=<cluster> \
--user=<user> \
--namespace=<ns>
```

---

## Networking Tests (curl / wget / nc)

ClusterIP:

```bash
kubectl exec pod -- curl http://svc.ns:port
```

NodePort:

```bash
curl http://<NodeIP>:<NodePort>
```

Ingress:

```bash
wget -O- ingress-nginx-controller.ingress-nginx/wear
wget --header="host: app.local" http://node:30080
```

---

## Control Plane Notes

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
systemctl restart kubelet
watch crictl ps
```

Admission plugins:

```bash
kubectl exec kube-apiserver-controlplane -n kube-system \
-- kube-apiserver -h | grep admission
```

---

## File Copy Between Nodes

```bash
scp /media/* node01:/web
```

---

## Inspect Docker Images

```bash
docker inspect <image> | grep -A5 '"Cmd"'
docker inspect <image> | grep -A5 '"Entrypoint"'
```