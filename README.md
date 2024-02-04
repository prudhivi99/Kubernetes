# Kubernetes

# Table of Contents

- 1 [Scheduling](#Scheduling)
    - 1.1 [Taints](#taints)
    - 1.2 [Tolerations](#tolerations)
    - 1.3 [NodeAffinity](#nodeaffinity)
 

## 1 Scheduling
### taints

```
Taints and tolerations are a mechanism that allows you to ensure that pods are not placed on inappropriate nodes. Taints are added to nodes,
while tolerations are defined in the pod specification. When you taint a node, it will repel all the pods except those that have a toleration
for that taint. A node can have one or many taints associated with it.
```

use cases
```
For example, most Kubernetes distributions will automatically taint the master nodes so that one of the pods that manages the control plane is
scheduled onto them and not any other data plane pods deployed by users. This ensures that the master nodes are dedicated to run control plane pods.
```


taint example 

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

