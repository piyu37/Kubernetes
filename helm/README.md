# Helm Command Reference

## 📌 Topics / Sections

1. [List Helm Releases](#list-helm-releases)
2. [Delete / Uninstall Helm Release](#delete--uninstall-helm-release)
3. [Download Helm Chart](#download-helm-chart)
4. [Install Helm Chart](#install-helm-chart)
5. [Install with Custom Values](#install-with-custom-values)
6. [Upgrade Helm Release](#upgrade-helm-release)
7. [Rollback Helm Release](#rollback-helm-release)
8. [Search Helm Charts](#search-helm-charts)
9. [Add Helm Repository](#add-helm-repository)
10. [Search Helm Repository](#search-helm-repository)
11. [List Helm Repositories](#list-helm-repositories)
12. [Update Helm Repositories](#update-helm-repositories)
13. [Show Chart Values](#show-chart-values)
14. [Lint Helm Chart](#lint-helm-chart)

---

## List Helm Releases

### Description

View Helm releases installed in the cluster.

### Commands

List releases in the **default namespace**:

```bash
helm ls
```

List releases in **all namespaces**:

```bash
helm ls -A
```

List **all releases** including failed, uninstalled, superseded:

```bash
helm ls -a
```

---

## Delete / Uninstall Helm Release

### Description

Remove a Helm release from a namespace.

### Command

```bash
helm -n team-yellow uninstall apiserver
```

---

## Download Helm Chart

### Description

Download a Helm chart locally without installing it.

### Command

```bash
helm pull --untar <helm-chart-name>
```

> This only downloads and extracts the chart.

---

## Install Helm Chart

### Description

Install a Helm chart into a namespace.

### Command

```bash
helm -n <namespace> install <release-name> <helm-chart-name>
```

Generate a random release name:

```bash
helm -n <namespace> install <helm-chart-name> --generate-name
```

> `<helm-chart-name>` format:
> `repo-name/chart-name`
> Example: `bitnami/nginx`

---

## Install with Custom Values

### Description

Install a chart while overriding default values.

### Command

```bash
helm -n <namespace> install <release-name> <helm-chart-name> \
--set key1=value1,key2=value2,key3=value3
```

### Example

```bash
helm install my-app bitnami/nginx --set replicaCount=2
```

---

## Upgrade Helm Release

### Description

Upgrade an existing Helm release.

### Command

```bash
helm upgrade <release-name> <helm-chart-name>
```

---

## Rollback Helm Release

### Description

Rollback a release to the previous revision.

### Command

```bash
helm rollback <release-name>
```

---

## Search Helm Charts

### Description

Search charts from Helm Hub (Artifact Hub).

### Command

```bash
helm search hub <helm-chart-name>
```

---

## Add Helm Repository

### Description

Add a Helm chart repository.

### Command

```bash
helm repo add <repo-name> <repo-url>
```

### Example

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

---

## Search Helm Repository

### Description

Search charts inside added repositories.

### Command

```bash
helm search repo <repo-name>
```

Search with available versions:

```bash
helm search repo <repo-name> --versions
```

---

## List Helm Repositories

### Description

List all configured Helm repositories.

### Command

```bash
helm repo list
```

---

## Update Helm Repositories

### Description

Fetch latest chart versions from repositories.

### Command

```bash
helm repo update
```

---

## Show Chart Values

### Description

Display all configurable values for a Helm chart.

### Command

```bash
helm show values bitnami/apache
```

> This outputs a **long list of supported configuration options**.

---

## Lint Helm Chart

### Description

Validate Helm charts for:

* YAML errors
* Missing or invalid values
* Deprecated Kubernetes APIs

### Command

```bash
helm lint
```
