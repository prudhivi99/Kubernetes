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
- 3 [RequestsAndLimits](#RequestsAndLimits)
    - 3.1 [RequestsAndLimits](#RequestsAndLimits)
  

## 1 Objects
### DaemonSets

```
DaemonSet is a Kubernetes controller used for cluster-level operations, ensuring that a specific Pod runs on every node
in the cluster.It automatically creates a new Pod when a new node is added and terminates it when a node is removed,
maintaining the desired state ofthe system. A DaemonSet in Kubernetes is like a chef in a restaurant responsible for
preparing a specific dish for every table, ensuringconsistency and quality in the end result.

```

![image](https://github.com/prudhivi99/Kubernetes/assets/63187046/aeb1be1d-a29d-4a59-8258-0946a512ebc5)

![image](https://github.com/prudhivi99/Kubernetes/assets/63187046/af475caa-7cbb-4c72-8a7a-f24f1afb3ad3)

![image](https://github.com/prudhivi99/Kubernetes/assets/63187046/5e3adf92-8f50-4e1f-88aa-ebf15ceca840)


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
### RequestsAndLimits

```
Kubernetes uses YAML files to specify resource requirements for pods and containers, including CPU and memory resources.
CPU resources are allocated using CPU requests and CPU limits in millicores. If a container exceeds the CPU limit, it
will be throttled by the kernel. Memory resources are allocated using memory requests and memory limits in units such as
bytes, kilobytes, megabytes, or gigabytes. If a container exceeds its memory limit, Kubernetes will terminate and restart
it, which is known as an OOMKilled event. Properly configuring memory limits is important to avoid degraded performance and
downtime.
```

![image](https://github.com/prudhivi99/Kubernetes/assets/63187046/9b6d3437-ad6f-43c6-aded-a63c0fd3c9de)

Resource requirements for pods and containers in Kubernetes can be specified in the YAML file. Kubernetes uses this information to 
schedule and manage the deployment of the pod or container across the available nodes in the cluster.

CPU Resources
In Kubernetes, CPU resources can be allocated to Pods and containers to ensure that they have the required processing power to run 
their applications. CPU resources are defined using the CPU resource units (millicores or milliCPU) and can be specified as CPU 
requests and CPU limits.

CPU Requests and Limits with YAML
pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
    resources:
      requests:
        cpu: "100m"
      limits:
        cpu: "200m"
```
This example specifies a pod with a single container that has a CPU request of 100 millicores (100m) and a CPU limit of 200 millicores (200m):

Impact of Exceeding CPU Limits
If a container tries to exceed its CPU request, it will be scheduled on a node that has enough CPU resources to satisfy the request. However, 
if a container tries to exceed its CPU limit, it will be throttled by the kernel. This can cause the container to become unresponsive, 
leading to degraded performance and potentially impacting other containers running on the same node.

Memory Resources
In Kubernetes, memory resources can be allocated to Pods and containers to ensure that they have the required memory to run their applications. 
Memory resources are defined using the memory resource units (such as bytes, kilobytes, megabytes, or gigabytes) and can be specified as memory
requests and memory limits.

Memory Requests and Limits with YAML
pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
    resources:
      requests:
        memory: "64Mi"
      limits:
        memory: "128Mi"
```

This example specifies a pod with a single container that has a memory request of 64 megabytes (64Mi) and a memory limit of 128 megabytes (128Mi).

Impact of Exceeding Memory Limits

![image](https://github.com/prudhivi99/Kubernetes/assets/63187046/2648babf-b291-4367-806f-d6e1e1f0dadb)

When a container exceeds its memory limit in Kubernetes, it is terminated and restarted by Kubernetes. This event is known as OOMKilled, 
which stands for Out Of Memory Killed. This happens when a container or a pod consumes all the available memory resources allocated to it, 
and the kernel terminates it to prevent it from consuming more memory and impacting the stability of the node. It is important to properly
configure memory limits for containers and pods to avoid OOMKilled events, as they can lead to degraded performance and downtime.


