# Kubernetes Kubectl Runbook

---

## Common Options

### Common Flags

- Output formats:
  ```
  kubectl get pods -o=json
  kubectl get pods -o=yaml
  kubectl get pods -o=wide
  ```
- Specify namespace:
  ```
  kubectl get pods -n=<namespace_name>
  ```
- Filter by label:
  ```
  kubectl get pods -l <label_name>:<label_value>
  ```
- Get all resources in all namespaces:
  ```
  kubectl get all --all-namespaces
  kubectl get all -A
  ```
- Apply recursively:
  ```
  kubectl apply -f <folder-path> --recursive
  ```
  > 💡 `--recursive` works with any operation that accepts `-f`, such as `kubectl create`, `kubectl get`, `kubectl delete`, `kubectl describe`, or `kubectl rollout`.

- Dry run:
  ```
  kubectl run|apply|create <resource|opts> --dry-run=<value>
  ```
  > 💡 Values: `none` (default), `client` (print YAML without sending), `server` (server-side request without persisting).

### Manage Resources

- Create, apply, delete:
  ```
  kubectl create -f <filename>
  kubectl apply -f <filename>
  kubectl delete -f <filename>
  ```

| kubectl apply | kubectl create |
| --- | --- |
| Declarative | Imperative |
| Accepts JSON/YAML | Accepts JSON/YAML |
| Updates or creates | Errors if exists |
| Can update existing | Only creates new |

---

## Cluster Management & Context

- Cluster info:
  ```
  kubectl cluster-info
  ```
- Kubernetes version:
  ```
  kubectl version
  ```
- View config:
  ```
  kubectl config view
  ```
- List users:
  ```
  kubectl config view -o jsonpath='{.users[*].name}'
  ```
- Current context:
  ```
  kubectl config current-context
  ```
- List contexts:
  ```
  kubectl config get-contexts
  ```
- Switch context:
  ```
  kubectl config use-context <cluster_name>
  ```
- List API resources/versions:
  ```
  kubectl api-resources
  kubectl api-versions
  ```

---

## Namespace

- Create:
  ```
  kubectl create namespace <namespace_name>
  ```
- List:
  ```
  kubectl get namespace <namespace_name>
  ```
- Describe:
  ```
  kubectl describe namespace <namespace_name>
  ```
- Delete:
  ```
  kubectl delete namespace <namespace_name>
  ```
- Edit:
  ```
  kubectl edit namespace <namespace_name>
  ```
- Resource usage:
  ```
  kubectl top namespace <namespace_name>
  ```

---

## Nodes

- List nodes:
  ```
  kubectl get node
  ```
- Taint node:
  ```
  kubectl taint node <node_name> key=value:Effect
  ```
- Delete node:
  ```
  kubectl delete node <node_name>
  ```
- Resource usage:
  ```
  kubectl top node <node_name>
  ```
- Pods on node:
  ```
  kubectl get pods -o wide | grep <node_name>
  ```
- Annotate node:
  ```
  kubectl annotate node <node_name>
  ```
- Cordon/uncordon:
  ```
  kubectl cordon node <node_name>
  kubectl uncordon node <node_name>
  ```
- Drain node:
  ```
  kubectl drain node <node_name>
  ```
- Label node:
  ```
  kubectl label node <node_name> key=value
  ```

---

## Pods

- List pods:
  ```
  kubectl get pod --show-labels
  kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'
  kubectl get pods --field-selector=status.phase=Running
  kubectl get pods -l key1=value1,key2=value2
  kubectl get po --label-columns=<label-name>
  ```
- Delete pod:
  ```
  kubectl delete pod <pod_name>
  ```
- Describe pod:
  ```
  kubectl describe pod <pod_name>
  ```
- Run pod:
  ```
  kubectl run <pod_name> --image=<image-name> --restart=Always|Never --command -- <cmd> --port=<port-number> expose=false|true
  ```
- Update image:
  ```
  kubectl set image pod/<pod-name> <container-name>=<image>:<new_version>
  ```
- Exec command:
  ```
  kubectl exec <pod_name> -c <container_name> <command>
  kubectl exec -it <pod_name> -- /bin/sh
  ```
- Resource usage:
  ```
  kubectl top pod
  ```
- Annotate/label pod:
  ```
  kubectl annotate pod <pod_name> <annotation>
  kubectl label pods <pod_name> new-label=<label>
  ```
- Port-forward:
  ```
  kubectl port-forward <pod_name> <external_port>:<container_port>
  ```

---

## Deployments

- List:
  ```
  kubectl get deployment
  ```
- Describe:
  ```
  kubectl describe deployment <deployment_name>
  ```
- Edit:
  ```
  kubectl edit deployment <deployment_name>
  ```
- Create:
  ```
  kubectl create deployment <deployment_name>
  ```
- Delete:
  ```
  kubectl delete deployment <deployment_name>
  ```
- Rollout status:
  ```
  kubectl rollout status deployment <deployment_name>
  ```
- Update image:
  ```
  kubectl set image deployment/<name> <container>=<image>:<new_version>
  ```
- Rollback:
  ```
  kubectl rollout undo deployment/<deployment_name> --revision=<revision>
  ```
- Force replace:
  ```
  kubectl replace --force -f <configuration_file>
  ```
- Autoscale:
  ```
  kubectl autoscale deploy nginx --max=10 --cpu-percent=80
  ```

---

## Daemonsets

- List:
  ```
  kubectl get daemonset
  ```
- Edit:
  ```
  kubectl edit daemonset <daemonset_name>
  ```
- Delete:
  ```
  kubectl delete daemonset <daemonset_name>
  ```
- Create:
  ```
  kubectl create daemonset <daemonset_name>
  ```
- Rollout:
  ```
  kubectl rollout daemonset
  ```
- Describe:
  ```
  kubectl describe ds <daemonset_name> -n <namespace_name>
  ```

---

## Replication Controller & Replica Sets

- List RC:
  ```
  kubectl get rc
  kubectl get rc --namespace=<namespace_name>
  ```
- List ReplicaSets:
  ```
  kubectl get replicasets
  ```
- Describe ReplicaSet:
  ```
  kubectl describe replicasets <replicaset_name>
  ```
- Scale:
  ```
  kubectl scale --replicas=[x] <resource>/<name>
  ```

---

## StatefulSet

- List:
  ```
  kubectl get statefulset
  ```
- Delete only StatefulSet (not pods):
  ```
  kubectl delete statefulset/<name> --cascade=false
  ```

---

## Jobs

- Create job:
  ```
  kubectl create job <name> --image=<image> -- <args>
  ```
- Create cronjob:
  ```
  kubectl create cronjob <name> --image=<name> --schedule='*/1 * * * *' -- <args>
  ```
- Create job from cronjob:
  ```
  kubectl create job --from=cronjob/sample-cron-job <name>
  ```

---

## Services

- List:
  ```
  kubectl get services
  ```
- Describe:
  ```
  kubectl describe services
  ```
- Expose:
  ```
  kubectl expose deployment <deployment_name>
  ```
- Edit:
  ```
  kubectl edit services
  ```

---

## Labels & Annotations

- Add/update:
  ```
  kubectl label <resource> <name> key1=value1
  kubectl annotate <resource> <name> key1=value1
  kubectl label <resource> <name> key1=value1 --overwrite
  kubectl annotate <resource> <name> key1=value1 --overwrite
  ```
- Update all pods:
  ```
  kubectl label pods --all key=value
  kubectl annotate pods --all key=value
  ```
- Update selected pods:
  ```
  kubectl label po -l "app in(v1,v2)" label=key
  kubectl label po -l app=v1,app=v2 label=key
  ```
- Remove label/annotation:
  ```
  kubectl label pods <name> <label>-
  kubectl annotate pods <name> <annotation>-
  ```
- List labels/annotations:
  ```
  kubectl label pods <name> --list
  kubectl annotate pods <name> --list
  ```

---

## Logs & Events

- Pod logs:
  ```
  kubectl logs <pod_name>
  kubectl logs --since=6h <pod_name>
  kubectl logs --tail=50 <pod_name>
  kubectl logs -f <pod_name>
  kubectl logs -c <container_name> <pod_name>
  kubectl logs <pod_name> > pod.log
  kubectl logs --previous <pod_name>
  ```
- Service logs:
  ```
  kubectl logs -f <service_name> [-c <container>]
  ```
- Events:
  ```
  kubectl get events
  kubectl get events --field-selector type=Warning
  kubectl get events --sort-by=.metadata.creationTimestamp
  kubectl get events --field-selector involvedObject.kind!=Pod
  kubectl get events --field-selector involvedObject.kind=Node,involvedObject.name=<node_name>
  kubectl get events --field-selector type!=Normal
  ```

---

## Service Account & Roles

- Create service account:
  ```
  kubectl create serviceaccount <name>
  ```
- Create time-bound token (1hr):
  ```
  kubectl create token <svc-acc-name>
  ```
- Create role/role binding:
  ```
  kubectl create role pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod
  kubectl create rolebinding myappnamespace clusterrole=view --serviceaccount=myappnamespace:myapp
  ```

---