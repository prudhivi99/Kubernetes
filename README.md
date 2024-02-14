# Kubernetes

# Table of Contents

- 1 [KubernetsObjects](#Objects)
    - 1.1 [Pods](#pods)
    - 1.2 [ReplicaSets](#replicasets)
    - 1.3 [Deployments](#deployments)
    - 1.4 [Services](#services)
    - 1.5 [DaemonSets](#daemonsets)
    - 1.6 [StaticPods](#staticpods)
    - 
- 2 [Scheduling](#Scheduling)
    - 2.1 [Taints](#taints)
    - 2.2 [Tolerations](#tolerations)
    - 2.3 [NodeAffinity](#nodeaffinity)
- 3 [RequestsAndLimits](#RequestsAndLimits)
    - 3.1 [RequestsAndLimits](#RequestsAndLimits)
- 4 [ClusterMaintenance](#ClusterMaintenance)
    - 4.1 [Maintenance](#Maintenance)
    - 4.2 [ClusterUpgrade](#ClusterUpgrade)
    - 4.3 [ETCDTL](#etcdtl)
    - 4.4 [ETCD](#etcd)
- 5 [Security](#Security)
    - 5.1 [Kubeconfig](#Kubeconfig)
    - 5.2 
  

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

### staticpods

```
Static pods in Kubernetes are created and managed directly by the kubelet service on a node, without the need for a Kubernetes API server.
The staticPodPath configuration setting specifies the directory path where the kubelet service should look for static pod manifests.
Although static pods cannot be managed from the Kubernetes API server, they can be retrieved using the kubectl get pods command. To modify
a static pod, we must edit the pod definition file on the node where the pod is running.
```

![image](https://github.com/prudhivi99/Kubernetes/assets/63187046/c24e7f5f-fffe-4946-9c06-212f8c859291)


### ClusterMaintenance

```
Q. We need to take node01 out for maintenance. Empty the node of all applications and mark it unschedulable.

kubectl drain node01 --ignore-daemonsets
```

```
Q. The maintenance tasks have been completed. Configure the node node01 to be schedulable again.

kubectl uncordon node01

After uncordon command, erlier pods it won't come back, only if we are creating new pods those will come.
```
```
Q. Mark node01 as unschedulable so that no new pods are scheduled on this node.

controlplane ~ ➜  kubectl cordon node01
node/node01 cordoned
```

### ClusterUpgrade

```
Q. How to get current version of cluster
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   33m   v1.26.0
node01         Ready    <none>          32m   v1.26.0
```
```
Q. You are tasked to upgrade the cluster. Users accessing the applications must not be impacted, and you cannot provision new VMs. What strategy would you use to upgrade the cluster?

upgrade one node at a time
```

```
Q. What is the latest stable version of Kubernetes as of today?

Look at the remote version in the output of the kubeadm upgrade plan command.
```
```
Q. We will be upgrading the controlplane node first. Drain the controlplane node of workloads and mark it UnSchedulable ?

controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   47m   v1.26.0
node01         Ready    <none>          46m   v1.26.0

controlplane ~ ✖ kubectl drain controlplane --ignore-daemonsets
node/controlplane already cordoned

first we need to drain/cordon then we need to upgrade 

```

```
Q. How to upgrade controlplane node and worker nodes ?

https://v1-28.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

to upgrade worker node

https://v1-28.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#upgrade-worker-nodes
```

```
Q. Mark the controlplane node as "Schedulable" again ? After upgrade control plane should be available .

controlplane ~ ➜  kubectl uncordon controlplane
node/controlplane uncordoned

controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   78m   v1.27.0
node01         Ready    <none>          78m   v1.26.0
```

```
Q. Drain the worker node of the workloads and mark it UnSchedulable

controlplane ~ ➜  kubectl drain node01
node/node01 cordoned
```

```
Q. Upgrade the worker node to the exact version v1.27.0

first you need to login into ssh node01, most of the people will execute from controlplane node, you will ended up with faulty result.

apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.27.0-00 && apt-mark hold kubeadm

https://v1-28.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/

```

### ETCDTL

```
etcdctl is a command line client for etcd.

export ETCDCTL_API=3

```

```
For example, if you want to take a snapshot of etcd, use:

etcdctl snapshot save -h and keep a note of the mandatory global options.

Since our ETCD database is TLS-Enabled, the following options are mandatory:

–cacert                verify certificates of TLS-enabled secure servers using this CA bundle

–cert                    identify secure client using this TLS certificate file

–endpoints=[127.0.0.1:2379] This is the default as ETCD is running on master node and exposed on localhost 2379.

–key                  identify secure client using this TLS key file

```
### etcd

```
Q. What is the version of etcd running on the cluster ?

controlplane ~ ➜  kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-5d78c9869d-qqcmm               1/1     Running   0          4m30s
coredns-5d78c9869d-zp4wb               1/1     Running   0          4m30s
etcd-controlplane                      1/1     Running   0          4m43s
kube-apiserver-controlplane            1/1     Running   0          4m45s
kube-controller-manager-controlplane   1/1     Running   0          4m41s
kube-proxy-gfh9j                       1/1     Running   0          4m30s
kube-scheduler-controlplane            1/1     Running   0          4m41s

controlplane ~ ➜  cd /etc/kubernetes/manifests/

controlplane /etc/kubernetes/manifests ➜  ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

controlplane /etc/kubernetes/manifests ➜  vim etcd.yaml

```

```
Q. if etcdctl command is not worked ?

controlplane /etc/kubernetes/manifests ➜  etcdctl snapshot
No help topic for 'snapshot'

controlplane /etc/kubernetes/manifests ✖ 

controlplane /etc/kubernetes/manifests ✖ 

controlplane /etc/kubernetes/manifests ✖ ETCDCTL_API=3 etcdctl snapshot

```

```
controlplane /etc/kubernetes/manifests ➜  export ETCDCTL_API=3

controlplane /etc/kubernetes/manifests ✖ etcdctl snapshot
NAME:

```

```
Q. Take a snapshot of the ETCD database using the built-in snapshot functionality.
   Store the backup file at location /opt/snapshot-pre-boot.db


controlplane /etc/kubernetes/manifests ✖ etcdctl snapshot save --endpoints=127.0.0.1:2379  \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key /opt/snapshot-pre-boot.db
Snapshot saved at /opt/snapshot-pre-boot.db

controlplane /etc/kubernetes/manifests ➜  ls /opt/snapshot-pre-boot.db
/opt/snapshot-pre-boot.db

```

```
Q. Luckily we took a backup. Restore the original state of the cluster using the backup file.

controlplane /etc/kubernetes/manifests ➜  etcdctl snapshot restore --data-dir /var/lib/etcd-from-backup /opt/snapshot-pre-boot.db
2024-02-14 11:41:26.035573 I | mvcc: restore compact to 3256
2024-02-14 11:41:26.041831 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32

we need to modify data directory of beow yaml file. becasue we have taken restore.
```

```yaml
controlplane /etc/kubernetes/manifests ✖ cat etcd.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.33.58.3:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.33.58.3:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --experimental-initial-corrupt-check=true
    - --experimental-watch-progress-notify-interval=5s
    - --initial-advertise-peer-urls=https://192.33.58.3:2380
    - --initial-cluster=controlplane=https://192.33.58.3:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.33.58.3:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.33.58.3:2380
    - --name=controlplane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: registry.k8s.io/etcd:3.5.7-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health?exclude=NOSPACE&serializable=true
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health?serializable=false
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
status: {}
```

```yaml
Q. How to show/view configured clusters ?

student-node ~ ✖ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://cluster1-controlplane:6443
  name: cluster1
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.35.127.6:6443
  name: cluster2
contexts:
- context:
    cluster: cluster1
    user: cluster1
  name: cluster1
- context:
    cluster: cluster2
    user: cluster2
  name: cluster2
current-context: cluster1
kind: Config
preferences: {}
users:
- name: cluster1
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: cluster2
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

```
How many nodes (both controlplane and worker) are part of cluster1?

Make sure to switch the context to cluster1:

kubectl config use-context cluster1

```

```
Q. How to find which  ETCD configured for cluster1?

kubectl config use-context cluster1

kubectl describe pod kube-apiserver-cluster1-controlplane -n kube-system

--etcd-servers=https://127.0.0.1:2379

In Api server if it is configured local host, then it should be a local/stacked etcd

```

```
Q. How many nodes are part of the ETCD cluster that etcd-server is a part of?

etcd-server ~ ✖ etcdctl --endpoints=https://192.35.127.18:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd.pem --key=/etc/etcd/pki/etcd-key.pem member list


3c219a26b410bb54, started, etcd-server, https://192.35.127.18:2380, https://192.35.127.18:2379, false

```

### Kubeconfig

```
Q. Where will be the kubeconfig located ?

controlplane ~ ✖ cat /root/.kube/config
```

```
Q. How to use custom kubeconfig file ?

controlplane ~ ➜  kubectl config use-context research --kubeconfig /root/my-kube-config
Switched to context "research".

```

```
Q. Set the my-kube-config file as the default kubeconfig by overwriting the content of ~/.kube/config with the content of the my-kube-config file.

 mv /root/my-kube-config /root/.kube/config

```











