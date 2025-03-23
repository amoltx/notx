# Kubernetese

Kubernetest is used to orchastrate containers to maintain high availablility.

Main conponents of Keb architecture are

- API server: frontend
- Scheduler: starts a container and assigns node and distrubutes the load
- Etc: distributed keyvalue db
- Container runtime
- Controller: controller is the main brain behind orchastration. they monitor and respond such as bringing node, container, endpoint up
- kubelet: is the engine running on each node, sort of agent

There are master and worker nodes. 

Master: Main kubernetese node. Following components run on master: kube API serve, controller manager, etc

Slaves: have kubelet

Minikube is a smiple to use k8s implementation for learning

### Troubleshooting
To troubleshoot a k8s object such as a pod or a replicaset, look at the "Events" under the "kubectl describe" command for that object

## Pod

- k8 encapsulates container into a POD. Pod is a single instance of an application. Pod is smallest object you can create in k8. A pod usually has only one container but may rarely have more than one containers 

## ReplicaSet

- ReplicaSet specifies how many instances of a pod to be started (was called ReplicationControler in earlier releases)
- ReplicaSet is specified as a "kind" in the yaml file
- If a pod crashes, new pods are started automatically to maintain the desired number of replicase
- If we externally start a pod with matching labels, that new pod will be terminated if the desired number of replicas are already running

After the ReplicaSet definition is updated, we can apply it using `kubectl replace` or `kubectl edit` commands
We can also change the number of replicas for a pod by using `kubectl scale replicaset` command if we do not want to edit the replicaser definition


Resource:
- Kubernetes for the Absolute Beginners - Hands-on