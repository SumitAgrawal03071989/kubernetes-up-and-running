kubectl<!-- markdownlint-disable -->
# kubernetes-up-and-running

## Google Cloud platform kubernetes
### Install google cloud SDK on local machine
```

echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list

curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -

sudo apt-get update && sudo apt-get install google-cloud-sdk

// select appropriate project while initiating.
gcloud init

// Here I created kubernetes cluster from UI.
// and then setup kubectl command on local.

gcloud container clusters get-credentials my-first-cluster-1 

```

## Prerequisite:
* Check that docker is installed and running.
```
docker version
```
* Check that kubernetes is stalled and runnning.
    * kuberneties is installed by following this documentation.
    https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#1-overview

    * created alias in .bashrc file.

    * ```alias kubectl='microk8s kubectl'```
    * this version command should return client and server version without any error
    * ```kubectl version```


```
token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
kubectl -n kube-system describe secret $token
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
```

## CHAPTER 4, Common kubectl Commands

### Namespaces
* Kubernetes uses namespaces to organize objects in the cluster. You can think of each
namespace as a folder that holds a set of objects.

```
kubectl get all --all-namespaces
```

### Contexts

* If you want to change the default namespace more permanently, you can use a context. 

### Viewing Kubernetes API Objects

Everything contained in Kubernetes is represented by a RESTful resource. Through‐
out this book, we refer to these resources as Kubernetes objects. Each Kubernetes
object exists at a unique HTTP path; for example, https://your-k8s.com/api/v1/name-spaces/default/pods/my-pod leads to the representation of a Pod in the default name‐space named my-pod.

### Creating, Updating, and Destroying Kubernetes Objects

Objects in the Kubernetes API are represented as JSON or YAML files. These files are
either returned by the server in response to a query or posted to the server as part of
an API request. You can use these YAML or JSON files to create, update, or delete
objects on the Kubernetes server.

```
kubectl apply -f obj.yaml
```
* we can save changes in obj.yaml and run apply again for cluster to respond to those chages.

### Labeling and Annotating Objects
* Labels and annotations are tags for your objects. 

### Debugging Commands

```
kubectl logs <pod-name>
```
* You can also copy files to and from a container using the cp command
```
kubectl cp <pod-name>:</path/to/remote/file> </path/to/local/file>
```

## CHAPTER 5, Pods

In earlier chapters we discussed how you might go about containerizing your applica‐tion, but in real-world deployments of containerized applications you will often want
to colocate multiple applications into a single atomic unit, scheduled onto a single
machine

### Pods in Kubernetes
* A Pod represents a collection of application containers and volumes running in the
same execution environment
* Pods, not containers, are the smallest deployable artifact in a Kubernetes cluster. 
* cgroups: cgroups is a Linux kernel feature that limits, accounts for, and isolates the resource usage of a collection of processes. 
* Each container within a Pod runs in its own cgroup, but they share a number of
Linux namespaces.
* Applications running in the same Pod share the same IP address and port space (network namespace), have the same hostname (UTS namespace)

### Thinking with Pods
Thumb rule: 
“Will these containers work correctly if they land on different machines?” If the answer is “no,” a Pod
is the correct grouping for the containers. If the answer is “yes,” multiple Pods is
probably the correct solution"

## The Pod Manifest
* Pods are described in a Pod manifest. The Pod manifest is just a text-file representation of the Kubernetes API object.
* Kubernetes strongly believes in declarative configuration
  * Declarative configuration: 
    * here you declare desired state and system like kubernetes acts to achive desired state.
    * Example, number of replicas for container or pod.
    * if you manually remove one container then kubernetes will add it, and vice a versa.
  
  * Imperative configuration:
    * series of steps to achive state of world.
    * This is old approach and results in hard to maintain systems.

### Creating a Pod

```
kubectl run kuard --generator=run-pod/v1 --image=gcr.io/kuar-demo/kuard-amd64:blue
or
kubectl apply -f /home/sumit/projects/kubernetes-up-and-running/chapter-05/kuard-pod.yaml

kubectl get pods

kubectl delete pods/myhelloworld
kubectl delete pods/kuard

kubectl run myhelloworld --generator=run-pod/v1 --image=sumitagrawal4548/helloworld
```

### Running Commands in Your Container with exec
```
kubectl exec -it kuard ash
kubectl exec -it myhelloworld ash


kubectl exec --stdin --tty myhelloworld -- /bin/bash

```

### Copying Files to and from Containers
```
// This command failed with file does not exists on container.
kubectl cp kuard:/captures/capture3.txt ./capture3.txt


kubectl cp /home/sumit/projects/kubernetes-up-and-running/chapter-05/TestMe.txt kuard:/TestMe.txt
tar: can't open 'TestMe.txt': Permission denied
command terminated with exit code 
```

## Health Checks
* Kubernetes restarts process if its not running.
* But what if process is stuck in deadlock and its not responding.
* To address this, Kubernetes introduced health checks for application liveness.
Liveness health checks run application-specific logic (e.g., loading a web page) to ver‐
ify that the application is not just still running, but is functioning properly. Since
these liveness health checks are application-specific, you have to define them in your
Pod manifest.

### Liveness Probe

```
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      livenessProbe:
        httpGet:
          path: /healthy
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP


kubectl apply -f /home/sumit/projects/kubernetes-up-and-running/chapter-05/kuard-pod-health.yaml

kubectl port-forward kuard 8080:8080

* open "http://localhost8080"
* Click on liveliness probe tab
* click on fail link.
* The application will fail.
* Kubernetes will restart the pod.

// details can be found with below command.

kubectl describe pods kuard

```

### Readiness Probe
* Liveness determines if an application is running properly. Containers that fail liveness checks are restarted. Readiness
describes when a container is ready to serve user requests.
* Combining the readiness and liveness probes helps ensure only healthy containers
are running within the cluster.

### Types of Health Checks
* Kubernetes also supports tcpSocket health checks
* This style of probe is useful for non-HTTP applications; for example, databases or other non–
HTTP-based APIs.

### Resource Management
* If you purchase a one-core machine,
and your application uses one-tenth of a core, then your utilization is 10%. 
* With scheduling systems like Kubernetes managing resource packing, you can drive your utilization to greater than 50%.
* Kubernetes allows users to specify two different resource metrics.
  * Resource requests specify the minimum amount of a resource required to run the application.
  * Resource limits specify the maximum amount of a resource that an application can consume.

### Resource Requests: Minimum Required Resources

To request that the kuard container lands on a machine with half a CPU
free and gets 128 MB of memory allocated to it, we define the Pod as shown in

```
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

### Request limit details
* We can specify minimum resource required by pod.
* Kubernetes makes sure that node has minimum resource capacity while scheduling pod to node.
* 

### Capping Resource Usage with Limits
In addition to setting the resources required by a Pod, which establishes the mini‐
mum resources available to the Pod, you can also set a maximum on a Pod’s resource
usage via resource limits.

## Persisting Data with Volumes
* Various ways you can persist the data.

# 
# CHAPTER 6. Labels and Annotations
Kubernetes was made to grow with you as your application scales in both size and
complexity. With this in mind, labels and annotations were added as foundational
concepts. Labels and annotations let you work in sets of things that map to how you
think about your application. You can organize, mark, and cross-index all of your
resources to represent the groups that make the most sense for your application.


# CHAPTER 7. Service Discovery

External resources on this 
https://kubernetes.io/docs/concepts/services-networking/service/


While the dynamic nature of Kubernetes makes it easy to run a lot of things, it creates problems when it comes to finding those things. Most of the traditional network infrastructure wasn’t built for the level of dynamism that Kubernetes presents.

The general name for this class of problems and solutions is service discovery. Service-discovery tools help solve the problem of finding which processes are listening at which addresses for which services.


The Domain Name System (DNS) is the traditional system of service discovery on the internet. DNS is designed for relatively stable name resolution with wide and efficient caching. It is a great system for the internet but falls short in the dynamic world of Kubernetes.


### The Service Object

* A Service object is a way to create a named label selector
* We can use kubectl expose to create a service.

```
kubectl create deployment alpaca-prod --image=gcr.io/kuar-demo/kuard-amd64:blue \
  --replicas=3 \
  --port=8080

kubectl label deployments alpaca-prod app- && \
  kubectl label deployments alpaca-prod app=alpaca ver=1 env=prod


kubectl expose deployment alpaca-prod

kubectl create deployment bandicoot-prod \
--image=gcr.io/kuar-d emo/kuard-amd64:green \
--replicas=2 \
--port=8080 

kubectl label deployments bandicoot-prod app- && \
  kubectl label deployments bandicoot-prod app=bandicoot ver=1 env=prod



kubectl expose deployment bandicoot-prod

kubectl get services -o wide
```


To interact with services, we are going to port forward to one of the alpaca Pods. Start and leave this command running in a terminal window. You can see the port forward working by accessing the alpaca Pod at http://localhost:48858:

```
$ ALPACA_POD=$(kubectl get pods -l app=alpaca \
    -o jsonpath='{.items[0].metadata.name}')
$ kubectl port-forward $ALPACA_POD 48858:8080
```

### Service DNS

Because the cluster IP is virtual, it is stable, and it is appropriate to give it a DNS address. All of the issues around clients caching DNS results no longer apply. Within a namespace, it is as easy as just using the service name to connect to one of the Pods identified by a service.

* Kubernetes provides a DNS service exposed to Pods running in the cluster.
* The DNS service is, itself, managed by Kubernetes and is a great example of Kubernetes building on Kubernetes


When referring to a service in your own namespace you can just use the service name (alpaca-prod). You can also refer to a service in another namespace with alpaca-prod.default. And, of course, you can use the fully qualified service name (alpaca-prod.default.svc.cluster.local.). Try each of these out in the “DNS Query” section of kuard.


### Readiness Checks

