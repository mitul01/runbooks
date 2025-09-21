# Krew Plugins for kubectl

Krew is a plugin manager for kubectl that makes it easy to discover, install, and manage plugins.

Installation
```commandline
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)

```

---
## Plugins

### 1.cnf - Kubectl config manager
Installation
```commandline
kubectl krew install cnf
```
Usage
```commandline
kubectl cnf helps switch between current-contexts in multiple kubeconfigs

Usage:
  kubectl cnf [-h] [<string>]

Flags:
  -h, --help        show this message
  -v, --version     show plugin version

Dependencies:
  fzf - https://github.com/junegunn/fzf
  bat - https://github.com/sharkdp/bat

Prerequisites:
  directory '/home/user/.kube/configs' populated with kubeconfig files
```
Ref - https://github.com/hedgieinsocks/kubectl-cnf

### 2. clog - colorize log outputs
Installation
```commandline
kubectl krew install clog
```
Usage
```commandline
# everything just like kubectl logs, just replace logs with clog
kubectl clog deploy/helloworld --tail=1 -f
```
Ref - https://github.com/orangetangerine/kubectl-clog

### 3. Lineage - display all dependent resources
Installation
```commandline
kubectl krew install lineage
```
Usage
```commandline
Display all dependencies or dependents of a Kubernetes object.

 TYPE is a Kubernetes resource. Shortcuts and groups will be resolved. NAME is the name of a particular Kubernetes resource.

Usage:
  kubectl lineage (TYPE[.VERSION][.GROUP] [NAME] | TYPE[.VERSION][.GROUP]/NAME) [flags]
  kubectl [command]

Examples:
  # List all dependents of the deployment named "bar" in the current namespace
  kubectl lineage deployments bar

  # List all dependents of the cronjob named "bar" in namespace "foo"
  kubectl lineage cronjobs.batch/bar --namespace=foo

  # List all dependents of the node named "k3d-dev-server" & the corresponding relationship type(s)
  kubectl lineage node/k3d-dev-server --output=wide

  # List all dependents of the persistentvolume named "disk", excluding event & secret resource types
  kubectl lineage pv/disk --dependencies --exclude-types=ev,secret

  # List all dependencies of the pod named "bar-5cc79d4bf5-xgvkc"
  kubectl lineage pod.v1. bar-5cc79d4bf5-xgvkc --dependencies

  # List all dependencies of the serviceaccount named "default" in the current namespace, grouped by resource type
  kubectl lineage sa/default --dependencies --output=split
```
Ref - https://github.com/tohjustin/kube-lineage

### 4. ns - Switch namespaces
Installation
```commandline
kubectl krew install ns
```
Usage
```commandline
USAGE:
  kubectl ns                    : list the namespaces in the current context
  kubectl ns <NAME>             : change the active namespace of current context
  kubectl ns -                  : switch to the previous namespace in this context
  kubectl ns -c, --current      : show the current namespace
```

### 4. resource-capacity - overview of resource requests, limits, and utilization
Installation
```commandline
kubectl krew install resource-capacity
```
Usage
```commandline
kubectl resource-capacity <flag>
Flags:
  -a, --available                 includes quantity available instead of percentage used
  -c, --containers                includes containers in output
  -o, --output string             output format for information (supports: [table csv tsv json yaml]) (default "table")
      --pod-count                 includes pod count per node in output
  -l, --pod-labels string         labels to filter pods with
  -p, --pods                      includes pods in output
      --sort string               attribute to sort results by (supports: [cpu.util cpu.request cpu.limit mem.util mem.request mem.limit cpu.util.percentage cpu.request.percentage cpu.limit.percentage mem.util.percentage mem.request.percentage mem.limit.percentage name]) (default "name")
  -u, --util                      includes resource utilization in output
      --sort                      Sort by one of: cpu.util, cpu.request, cpu.limit, mem.util, mem.request, mem.limit, name
```
Ref - https://github.com/robscott/kube-capacity
