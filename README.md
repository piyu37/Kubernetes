# Kubernetes Aliases and Commands

This document provides a collection of useful `kubectl` aliases, commands, and best practices to streamline Kubernetes operations.

## Table of Contents

- [Aliases](#aliases)
- [Applying Changes](#applying-changes)
- [Bash Completion](#bash-completion)
- [Kubectl Commands](#kubectl-commands)
  - [Pod Management](#pod-management)
  - [Deployment Management](#deployment-management)
  - [Services](#services)
    - [ClusterIP Service](#clusterip-service)
    - [NodePort Service](#nodeport-service)
  - [Job and CronJob Management](#job-and-cronjob-management)
    - [CronJob Schedule Syntax](#cronjob-schedule-syntax)
  - [ConfigMaps and Secrets](#configmaps-and-secrets)
  - [ServiceAccounts](#serviceaccounts)
  - [Roles and RoleBindings](#roles-and-rolebindings)
  - [ClusterRole and ClusterRoleBinding](#clusterrole-and-clusterrolebinding)
  - [Ingress Management](#ingress-management)
  - [Network Policies](#network-policies)
  - [PersistentVolume and PersistentVolumeClaim](#persistentvolume-and-persistentvolumeclaim)
- [Kubeconfig Commands](#kubeconfig-commands)
- [Inspecting Docker Images](#inspecting-docker-images)

## Aliases

To speed up `kubectl` operations, add these aliases to your shell configuration file:

```bash
export dr='--dry-run=client -o yaml'
export now='--grace-period 0 --force'

alias k=kubectl
alias kn='k config set-context --current --namespace'
alias kaf='k apply -f'
alias kgp='k get po'
alias kgs='k get svc'
alias kc='k config get-contexts'
alias kg='k get'
alias ke='k explain --recursive'
```

## Applying Changes

To apply alias changes, run:

```bash
source ~/.bashrc
```

For persistent changes, add them to `~/.bash_profile`:

```bash
source ~/.bash_profile
```

## Bash Completion

Enable Bash completion for `kubectl` and aliases:

```bash
source /etc/profile.d/bash_completion.sh
source <(kubectl completion bash)
alias k=kubectl
complete -F __start_kubectl k
```

## Kubectl Commands

### Pod Management

```bash
kubectl run <pod-name> --image=<img-name> --labels="key1=value1,key2=value2" --port=5701 --env="key1=value1" --env="key2=value2" -- <arg1> <arg2> ... <argN>
```

### Deployment Management

```bash
kubectl create deployment <my-dep> --image=<img-name> --replicas=3
kubectl scale deployment <my-dep> --replicas=4
kubectl set image deployment <my-dep> <container-name>=nginx:1.17
kubectl rollout status deployment <my-dep>
kubectl rollout history deployment <my-dep>
kubectl rollout undo deployment <my-dep>
```

### Services

#### ClusterIP Service

```bash
kubectl expose pod <pod-name> --port=6379 --name <svc-name> --dry-run=client -o yaml
```
This will automatically use the pod's labels as selectors.

Or

```bash
kubectl create service clusterip <svc-name> --tcp=<port>:<targetPort> --dry-run=client -o yaml
```
This will not use the pod's labels as selectors. Instead, it assumes `app=redis` as the selector. Modify the selectors before creating the service.

#### NodePort Service

```bash
kubectl expose pod <pod-name> --port=80 --name <svc-name> --type=NodePort --dry-run=client -o yaml
```
This will automatically use the pod's labels as selectors, but you cannot specify the node port.

Or

```bash
kubectl create service nodeport <svc-name> --tcp=<port>:<targetPort> --node-port=30080 --dry-run=client -o yaml
```
This will not use the pod's labels as selectors.

To specify a node port manually, generate a definition file using `kubectl expose` and modify the node port before applying.

### Job and CronJob Management

```bash
kubectl create job <my-job> --image=<image-name>
kubectl create cronjob <my-job> --image=<image-name> --schedule="*/1 * * * *"
```

#### CronJob Schedule Syntax

The `.spec.schedule` field follows standard Cron syntax:

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday)
# │ │ │ │ │
# * * * * *
```
For example, `0 3 * * 1` runs the job weekly on Monday at 3 AM.

### ConfigMaps and Secrets

```bash
kubectl create configmap <my-config> --from-literal=key1=config1 --from-literal=key2=config2 --from-file=path/to/bar --from-env-file=path/to/foo.env
kubectl create secret generic <my-secret> --from-literal=key1=supersecret --from-literal=key2=topsecret

To decode the secret:
kubectl get secret my-secret -o jsonpath="{.data.username}" | base64 -d
```

### ServiceAccounts

```bash
kubectl create serviceaccount <my-service-account>
```

### Roles and RoleBindings

```bash
kubectl create role <role-name> --namespace=<namespace> --verb=* --resource=pods,services,persistentvolumeclaims --dry-run=client -o yaml > role-developer.yaml
kubectl create rolebinding <rolebinding-name> --role=<role-name> --user=<user-name> --namespace=<namespace> --dry-run=client -o yaml > rolebinding-developer.yaml
```

### ClusterRole and ClusterRoleBinding

```bash
kubectl create clusterrole <role-name> --verb=* --resource=pods,services,persistentvolumeclaims --dry-run=client -o yaml > role-developer.yaml
kubectl create clusterrolebinding <rolebinding-name> --clusterrole=<cluster-role-name> --user=<user-name1> --user=<user-name2> --group=group1
```

### Ingress Management
```bash
kubectl create ing <ingress-name> --rule="host1/path1=service1:port1" --rule="host2/path2=service2:port2" --annotation="nginx.ingress.kubernetes.io/rewrite-target=/" $dr > ingress.yaml
```

### Network Policies
For network policies, refer to the Kubernetes documentation or use the following command:

```bash
k explain --recursive netpol
```

### PersistentVolume and PersistentVolumeClaim
For persistent volumes & persistent volume claims, refer to the Kubernetes documentation or use the following commands:

```bash
k explain --recursive pv
k explain --recursive pvc
```

### To display only the fields inside spec of a CronJob but excluding spec.jobTemplate, you can use the kubectl explain command along with grep to filter out the unwanted section.

```bash
k explain cronjob.spec | grep -v "jobTemplate"
```
This will show all spec fields except jobTemplate.

## Kubeconfig Commands

Set up credentials and context for Kubernetes:

```bash
kubectl config set-credentials <user-name> --client-certificate <client-certificate-path> --client-key <client-key-path>
kubectl config set-context <context-name> --cluster=<cluster-name> --user=<user-name> --namespace=<namespace>
```

## Inspecting Docker Images

To check default commands and entrypoints for an image:

```bash
docker inspect <image-name> | grep -A5 '"Cmd"'
docker inspect <image-name> | grep -A5 '"Entrypoint"'
```

