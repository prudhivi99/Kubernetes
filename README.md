# Kubernetes

# Table of Contents

- 1 [KubernetsObjects](#Objects)
    - 1.1 [Pods](#pods)
    - 1.2 [ReplicaSets](#replicasets)
    - 1.3 [Deployments](#deployments)
    - 1.4 [Services](#services)
    - 1.5 [DaemonSets](#daemonsets)
    - 
- 2 [Scheduling](#Scheduling)
    - 2.1 [Taints](#taints)
    - 2.2 [Tolerations](#tolerations)
    - 2.3 [NodeAffinity](#nodeaffinity)

## 1 Objects
### DaemonSets

```
DaemonSet is a Kubernetes controller used for cluster-level operations, ensuring that a specific Pod runs on every node in the cluster. It automatically creates a new Pod when a new node is added and terminates it when a node is removed, maintaining the desired state of the system. A DaemonSet in Kubernetes is like a chef in a restaurant responsible for preparing a specific dish for every table, ensuring consistency and quality in the end result.


Kubernetes: DaemonSets
Table of Contents

· DaemonSets
· Short Name: ds
· DaemonSet with YAML
· Commands
DaemonSets

DaemonSets
DaemonSet is a Kubernetes controller that ensures that a copy of a specific Pod is running on every node in the cluster. It’s typically used for cluster-level operations like logging, monitoring, or other background tasks that need to be performed on every node. When a new node is added to the cluster, the DaemonSet will automatically create a new Pod on that node, and when a node is removed, the DaemonSet will terminate the corresponding Pod. This ensures that the desired state of the system is always maintained, regardless of the cluster’s size or configuration changes.

Analogously, a DaemonSet in Kubernetes can be compared to a chef in a restaurant who is responsible for preparing a specific dish for every table.
The chef is responsible for maintaining the consistency and quality of the dishes served to customers.

Short Name: ds
$ kubectl api-resources
NAME          SHORTNAMES   APIVERSION    NAMESPACED   KIND
daemonsets    ds           apps/v1       true         DaemonSet
DaemonSet with YAML

daemonSet with YAML
daemonSet.yaml

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: <deployment_name>
  labels:
    <key1>: <value1>
    <key2>: <value2>
           :
           :
    <keyM>: <valueM>
spec:
  selector:
    matchLabels:
      <key1>: <value1>
      <key2>: <value2>
             :
             :
      <keyN>: <valueN>
  template:
    # pod template starts here
    metadata:
      name: <pod_name>
      labels:
        <key1>: <value1>
        <key2>: <value2>
               :
               :
        <keyN>: <valueN>
    spec:
      containers:
        - name: <container1_name>
          image: <image>
        - name: <container2_name>
          image: <image>
    # pod template ends here
Commands

commands
Create a daemonSet using a YAML file
$ kubectl create -f <daemonset_name>.yaml
2. Retrieve a list of all daemonSets in a Kubernetes cluster

$ kubectl get daemonsets
3. Get detailed information about a specific daemonSet in the Kubernetes cluster

$ kubectl describe daemonsets <daemonset_name>
4. Delete a daemonSet from the Kubernetes cluster

$ kubectl delete daemonsets <daemonset_name>
```

## 2 Scheduling
### taints

```
Taints and tolerations are a mechanism that allows you to ensure that pods are not placed on inappropriate nodes. Taints
are added to nodes,while tolerations are defined in the pod specification. When you taint a node, it will repel all the
pods except those that have a toleration for that taint. A node can have one or many taints associated with it.
```

use cases
```
For example, most Kubernetes distributions will automatically taint the master nodes so that one of the pods that manages
the control plane is scheduled onto them and not any other data plane pods deployed by users. This ensures that the master
nodes are dedicated to run control plane pods.
```


Q. taint example 

```
Create another pod named bee with the nginx image, which has a toleration set to the taint mortein
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: bee
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  tolerations:
  - key: "spray"
    value: "mortein"
    operator: "Equal"
    effect: "NoSchedule"
status: {}
```

```
Q. Remove the taint on controlplane, which currently has the taint effect of NoSchedule
kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```
### nodeaffinity

```
Node Affinity
Node affinity is a set of rules the Kubernetes scheduler uses to determine where a pod can be placed. It is similar to
the nodeSelector parameter but offers more flexibility and functionality.

How it Works
Node affinity in Kubernetes enables users to constrain which nodes a pod can be scheduled onto using labels on the nodes
and label selectors specified in the pods. Kubernetes supports two types of node affinities:

Required (requiredDuringSchedulingIgnoredDuringExecution): This enforces that the rule must be met for a pod to be scheduled
onto a node. If no node meets the requirement, the pod will not be scheduled.
Preferred (preferredDuringSchedulingIgnoredDuringExecution): This specifies that the Kubernetes scheduler will try to enforce the rules but does not guarantee the placement.
These affinities are specified in the pod specification using the .spec.affinity.nodeAffinity field.
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

see difference b/n nodeAffinity section above and below.

```yaml
Name: red

Replicas: 2

Image: nginx

NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution

Key: node-role.kubernetes.io/control-plane

Use the right operator


spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: red
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: red
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
              operator: Exists
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

