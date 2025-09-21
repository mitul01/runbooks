# Commands useful during debugging

### 1. Use Case: Quickly find failing resources
```commandline
# Get only pods with errors
kubectl get pods -A --field-selector=status.phase=Failed

# Watch events sorted by timestamp
kubectl get events -A --sort-by=.lastTimestamp

# Pods crashlooping with restart counts
kubectl get pods -A --no-headers | awk '$4=="CrashLoopBackOff" || $5 > 3'
```

### 2. Use Case: Verify service pod connectivity
```commandline
# Check which endpoints a service is routing to
kubectl get endpoints <service-name> -n <namespace> -o wide

# Run a debug pod for connectivity testing
kubectl run net-debug --rm -it --image=busybox:1.36 --restart=Never -- sh
# Inside pod:
#   nslookup <service-name>
#   wget http://<service-name>:<port>
```
### 3. Use Case: Pod scheduling failures
```commandline
# Find the node selection criteria of pod
kubectl get pod <pod-name> -n <namespace> -o jsonpath='
NodeName: {.spec.nodeName}{"\n"}
NodeSelector: {.spec.nodeSelector}{"\n"}
Tolerations: {.spec.tolerations}{"\n"}
Affinity: {.spec.affinity}{"\n"}
TopologySpreadConstraints: {.spec.topologySpreadConstraints}{"\n"}'

# Find the list of nodes with important details which may influence scheduling
kubectl get nodes -o custom-columns="NAME:.metadata.name,TAINTS:.spec.taints,CAPACITY_CPU:.status.capacity.cpu,CAPACITY_MEM:.status.capacity.memory,ALLOCATABLE_CPU:.status.allocatable.cpu,ALLOCATABLE_MEM:.status.allocatable.memory"
```

