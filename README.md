<!-- markdownlint-disable -->
# kubernetes-up-and-running

## Prerequisite:
* Check that docker is installed and running.
```
docker version
```
* Check that kubernetes is stalled and runnning.
    * kuberneties is installed by following this documentation.
    https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#1-overview

    * created alias in .bashrc file.

    * ```alias kctl='microk8s kubectl'```
    * this version command should return client and server version without any error
    * ```kctl version```


```
token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
kctl -n kube-system describe secret $token
```

## The Kubernetes Client
The official Kubernetes client is kubectl

### Checking Cluster Status

```
kubectl get componentstatuses

NAME                 STATUS    MESSAGE   ERROR
scheduler            Healthy   ok        
controller-manager   Healthy   ok 
```

* controller-manager is responsible for running various controllers that regulate
behavior in the cluster
* The scheduler is responsible for placing different Pods onto
different nodes in the cluster.

### Listing Kubernetes Worker Nodes

```
kubectl get nodes

// to get specific information about any node. 
kctl describe nodes sumit-lenovo-ideapad-s540-15iml-d

Name:               sumit-lenovo-ideapad-s540-15iml-d
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=sumit-lenovo-ideapad-s540-15iml-d
                    kubernetes.io/os=linux
                    microk8s.io/cluster=true
Annotations:        node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 172.18.0.1/16
                    projectcalico.org/IPv4VXLANTunnelAddr: 10.1.68.192
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Thu, 17 Sep 2020 07:52:15 +0530
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  sumit-lenovo-ideapad-s540-15iml-d
  AcquireTime:     <unset>
  RenewTime:       Sun, 20 Sep 2020 14:29:51 +0530
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Sun, 20 Sep 2020 12:19:20 +0530   Sun, 20 Sep 2020 12:19:20 +0530   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Sun, 20 Sep 2020 14:29:42 +0530   Thu, 17 Sep 2020 07:52:15 +0530   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Sun, 20 Sep 2020 14:29:42 +0530   Thu, 17 Sep 2020 07:52:15 +0530   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Sun, 20 Sep 2020 14:29:42 +0530   Thu, 17 Sep 2020 07:52:15 +0530   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Sun, 20 Sep 2020 14:29:42 +0530   Sun, 20 Sep 2020 12:19:21 +0530   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  192.168.0.105
  Hostname:    sumit-lenovo-ideapad-s540-15iml-d
Capacity:
  cpu:                8
  ephemeral-storage:  181487144Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             7881632Ki
  pods:               110
Allocatable:
  cpu:                8
  ephemeral-storage:  180438568Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             7779232Ki
  pods:               110
System Info:
  Machine ID:                 00ac7ef85db743de9c75ee4d8456ef43
  System UUID:                20200717-f8ac-652c-eef8-f8ac652ceefc
  Boot ID:                    3a73035d-8fa6-4841-bbba-6905cbff14b1
  Kernel Version:             5.4.0-47-generic
  OS Image:                   Ubuntu 20.04.1 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.3.7
  Kubelet Version:            v1.19.0-34+09a4aa08bb9e93
  Kube-Proxy Version:         v1.19.0-34+09a4aa08bb9e93
Non-terminated Pods:          (6 in total)
  Namespace                   Name                                          CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                   ----                                          ------------  ----------  ---------------  -------------  ---
  kube-system                 dashboard-metrics-scraper-6c4568dc68-h5qv5    0 (0%)        0 (0%)      0 (0%)           0 (0%)         3d6h
  kube-system                 kubernetes-dashboard-7ffd448895-9bg8z         0 (0%)        0 (0%)      0 (0%)           0 (0%)         3d6h
  kube-system                 metrics-server-8bbfb4bdb-kl64v                0 (0%)        0 (0%)      0 (0%)           0 (0%)         3d6h
  kube-system                 calico-kube-controllers-847c8c99d-gw8l5       0 (0%)        0 (0%)      0 (0%)           0 (0%)         3d6h
  kube-system                 calico-node-mf88l                             250m (3%)     0 (0%)      0 (0%)           0 (0%)         3d6h

```


## CHAPTER 4, Common kubectl Commands

### Namespaces
* Kubernetes uses namespaces to organize objects in the cluster. You can think of each
namespace as a folder that holds a set of objects.

```
kctl get all --all-namespaces
```

### Contexts

* If you want to change the default namespace more permanently, you can use a con‐
text. 

### Viewing Kubernetes API Objects

Everything contained in Kubernetes is represented by a RESTful resource. Through‐
out this book, we refer to these resources as Kubernetes objects. Each Kubernetes
object exists at a unique HTTP path; for example, https://your-k8s.com/api/v1/name‐
spaces/default/pods/my-pod leads to the representation of a Pod in the default name‐
space named my-pod.

### Creating, Updating, and Destroying Kubernetes Objects
