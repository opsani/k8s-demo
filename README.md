### General reading
 - https://www.infoworld.com/article/3207686/cloud-computing/how-to-get-started-with-kubernetes.html

 - https://blog.codeship.com/getting-started-with-kubernetes/

### Playgrounds

  - Play with K8S: http://labs.play-with-k8s.com/

  - Katacoda: (courses) https://www.katacoda.com/courses/kubernetes/playground

### Install options:
https://kubernetes.io/docs/setup/

### Install minikube on a ubuntu host.

Requires docker to be pre installed, sets up a single node cluster on the host, no extra VMs started (besides the one the host may run on):

https://github.com/kubernetes/minikube/blob/v0.23.0/README.md

### Get started with minikube/kubectl (hello world):
https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/

### kubectl

Overview (mini ref)
https://kubernetes.io/docs/user-guide/kubectl-overview/

Cheat sheet
https://kubernetes.io/docs/user-guide/kubectl-cheatsheet/

### Brief Terminology Summary

#### Pod
https://kubernetes.io/docs/concepts/workloads/pods/pod/

A pod (as in a pod of whales or pea pod) is a group of one or more containers (such as Docker containers), with shared storage/network, and a specification for how to run the containers. A pod’s contents are always co-located and co-scheduled, and run in a shared context. A pod models an application-specific “logical host” - it contains one or more application containers which are relatively tightly coupled — in a pre-container world, they would have executed on the same physical or virtual machine.

#### Deployment
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/


A Deployment controller provides declarative updates for Pods and ReplicaSets.
You describe a desired state in a Deployment object, and the Deployment controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

#### ReplicaSet vs Deployment

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don’t require updates at all.

#### Service
https://kubernetes.io/docs/concepts/services-networking/service/

A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service. The set of Pods targeted by a Service is (usually) determined by a Label Selector (see below for why you might want a Service without a selector).


### Example voting app on k8s

Run the following to deploy the app.

```
# List objects
kubectl get deployment,svc,pods

# Deploy voting app
kubectl create -f https://raw.githubusercontent.com/opsani/k8s-demo/master/app.yaml

# List objects
kubectl get deployment,svc,pods

# Delete objects
kubectl delete -f https://raw.githubusercontent.com/opsani/k8s-demo/master/app.yaml
```
