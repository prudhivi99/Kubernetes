# Kubernetes

# Table of Contents

- 1 [Scheduling](#Scheduling)
    - 1.1 [Taints](#taints)
    - 1.2 [Tolerations](#tolerations)
    - 1.3 [NodeAffinity](#nodeaffinity)
 

## 1 Scheduling
### taints

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

