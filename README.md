# Certified Kubernetes Application Developer (CKAD)

## KUBERNETES ARCHITECTURE

The purpose of Kubernetes is to host your applications in the form of containers in an automated fashion so that you can easily deploy as many instances of your application.

![](./images/masterandworkernodes.jpg)

We have 2 kind of ships in this analogy.
- Cargo ships that does the actual work of carrying containers across the sea.
- Control ships that are responisble for monitoring and managing the cargo ships.

The Kubernetes clusters consists of a set of nodes which may be physical or virtual on premise or on cloud that host applications in the form of containers. This relate to the cargo ships in this analogy. Worker nodes in the clusters are ships that can load containers, but somebody needs to load the containers on the ships not just load, plan how to load, identify the right ships, store information about the ships, monitor and track the location of containers on the ships. This is done by control ships that host different offices and departments, monitoring equipments, communication equipments. The control ships relate to the master node in the Kubernetes cluster. The master node is responsible for managing the Kubernetes cluster, storing information regarding the different nodes, planning which containers goes where, monitoring the nodes and containers on them.


### Components of Master and Worker Node

![](./images/masterandworkercomponentes.png)


### Master Node Components
#### 1) ETCD

Etcd is a database that stores information in a key-value format. Every information you see when you run the kubectl get command is from the etcd server. Every change you make to your cluster such as adding additional nodes, deploying pods or replica sets are updated in the etcd server.

#### 2) Controller Manager

In Kubernetes, we have controllers availabe that take care of different areas.
The Node controller takes care of nodes. They are responsible for onboarding new nodes to the cluster, handling situations where nodes become unavailable or get destroyed.
The Replication controller ensures that the desired number of containers are running at all times in a replication group.

#### 3) Scheduler

When ships arrive, you load containers on them using cranes. The cranes identify the containers that need to be placed on ships. It identify the right ship based on its size, its capacity, the number of containers already on the ship, the type of containers it is allowed to carry etc. Those are schedulers in a Kubernetes cluster. A scheduler identifies the right node to place a container on based on the containers resource requirements, the worker nodes capacity.

The scheduler is only responsible for deciding which pod goes on which node. It does not actually place the pod on the nodes. That is the job of the kubelet. The kubelet or captain on the ship is who creates the pod on the ship. The scheduler only decides which pod goes where.
#### 4) Kube-apiserver

You have seen different components like different ships, offices, the data store, the cranes. But, how do these communicate with each other?
The Kube-apiserver is the primary management component of Kubernetes. The Kube API server is responsible for orchestrating all operations within the cluster. It exposes the Kubernetes API which is used by external users to perform management operations on the cluster as well as the various controllers to monitor the state of the cluster.
We all work with containers in here, so we need Docker installed on all the nodes in the cluster.

When you run a kubectl command, the kubectl utility is in fact reaching to the kube-apiserver.

Let's look at an example of creating a pod. When you do that, the request is authenticated first, then validated. In this case, the kube-apiserver creates a pod object without assigning it to a node. Updates the information in the etcd server, updated the user that the pod has been created. The scheduler continuosly monitors the API server and realizes that there is a new pod with no node assigned. The scheduler identifies the right node to place the new pod on, and communicates that back to the kube-apiserver. The API server then updates the information in the etcd cluster. The API server then passes that information to the kubelet in the appropriate worker node. The kubelet then creates the pod on the node and instructs the container runtine engine to deploy the application image. Once done, the kubelet updates the status back to the API server, and the API server then updates the data back to the etcd cluster.

### Worker Node Components

#### 1) Kubelet

Let us now turn our focus onto the cargo ships. Now, every ship has a captain. The captain is responsible for managing all activities on the ship. The captain is responsible for liasing with the master ship, starting with letting the master ship know they are interested in joining the group, receiving information about the containers to be loaded on the ship, and loading the appropriate containers as required, sending reports back to the master node about the status of this ship, and the status of the containers on the ship etc. The captain of the ship is the kubelet in Kubernetes.
A kubelet is an agent that runs on each node in a cluster. It listens for instructions from the Kube API server, and deploys and destroys containers on the nodes as required. The Kube API server periodically fetches status reports  from the kubelet to monitor the status of nodes and containers on them.

Kubelet is the captain of the ship. They are the sole point of contact from the mastership. They load and unload containers on the ship as instructed by the scheduler on the master. They also send back reports at regular intervals on the status of the ship and containers on them.

The kubelet in the kubernetes worker node registers the node with a Kubernetes cluster. When it receives instructions to load a container or a pod on the node, it requests the container runtime engine which may be Docker to pull the required image and run an instance. The kubelet then continues to monitor the state of the pod and containers in it and reports back to the kube API server on a timely basis.

#### 2) Kube-proxy

The kubelet was more of a captain on the ship that manages containers on the ship, but the application running on the worker nodes need to be able to communicate with each other. For example, you might have a web server running in one container on one of the nodes and a database server running on another container on another node. How would the web server reach the database server on the other node? Communication between worker nodes are enabled by another component that runs on the worker node know as the Kube-proxy Service. The Kube-proxy Service ensures that the necessary rules are in place on the worker nodes to allow containers running on them to reach each other.


Within a Kubernetes cluster, every pod can reach every every other pod. This is accomplished by deploying a pod networking solution to the cluster. A POD Network is an internal virtual network that spans across all the nodes in the cluster to which all the pods connect to. Through this network, they're able to communicate with each other. There are many solutions available for deploying such a network. In this case, I have a web application deployed on the first node, and a database application deployed on the second node. The web app can reach the database simply by using the IP of the pod , but there is no guarantee that the IP of the database pod will always remain the same. If you have gone through the lecture on services, you must know that a better way for the web application to access the database is using a service, so we create a service to expose the database application across the cluster. The web application can now access the database using the same name of the service, db. The service also gets an IP address assigned to it. Whenever a pod tries to reach the service using its IP or name, it forwards the traffic to the backend pod in this case the database. But, what is this service, and how does it get an IP? Does the service join the same POD network? The service cannot join the POD network because the service is not an actual thing. It is not a container like pods, so it doesn't have any interfaces. It is a virtual component that only lives in the Kubernetes memory. But then, we also said that the service should be accessible accross the cluster from any nodes. So, how is that achieved? That's where kube-proxy comes in. Kube-proxy is a process that runs on each node in the Kubernetes cluster. Its job is to look for new services.



-------------------------------------------------------------------------------------------------


## COMMANDS

### 1) PODS

- #### To create a Pod running nginx

    ```
    kubectl run nginx --image nginx
    ```
- #### To create a Pod by using a pod-definition.yml file
    ```
    - kubectl create -f pod-definition.yml
    ```
- #### To list all Pods
    ```
    - kubectl get pods
    ```
- #### To see the details of a Pod named as myapp-pod
    ```
    - kubectl describe pod myapp-pod
    ```
- #### To delete a Pod named as myapp-pod
    ```
    - kubectl delete pod myapp-pod
    ```

### 2) REPLICA SETS

- #### To create a Replica Set using a replicaset-definition.yml
    ```
    kubectl create -f replicaset-definition.yml
    ```
- #### To list all Replica Sets
    ```
    kubectl get replicaset
    ```
- #### To delete a Replica Set named as myapp-replicaset
    ```
    kubectl delete replicaset myapp-replicaset
    ```
- ####  To update/replace the existing Replica Set
    ```
    kubectl replace -f replicaset-definition.yml
    ```
- #### To scale replicas in the .yml file ( Note: This will not edit the .yml file )
    ```
    kubectl scale --replicas=6 -f replicaset-definition.yml
    ```
- #### To edit a Replica Set named as new-replica-set (This will open the .yml file)
    ```
    kubectl edit replicaset new-replica-set
    ```
### 3) DEPLOYMENTS

- #### To create a Deployment using a deployment-definition.yml
    ```
    kubectl create -f deployment-definition.yml
    ```
- #### To list all deployments
    ```
    kubectl get deployments
    ```
- #### To see all the created objects at once
    ```
    kubectl get all
    ```
- #### To create a new deployment with these values (name: httpd-frontend, replicas: 3, image: httpd:2.4-alpine)
    ```
    kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
    ```

### 4) NAMESPACES

- #### To see all namespaces
    ```
    kubectl get namespaces
    ```
- #### To see the pods in kube-system namespace
    ```
    kubectl get pods --namespace=kube-system
    ```
- #### To create a namespace using namespace-definition.yml
    ```
    kubectl create -f namespace-definition.yml
    ```
- #### Another way to create a namespace with a name dev
    ```
    kubectl create namespace dev
    ```
- #### To view pods in all namespaces
    ```
    kubectl get pods --all-namespaces
    ```
- #### To create a pod (with .yml file) inside a specific namespace named as team1
    ```
    kubectl create -f pod-definition.yml --namespace=team1
    ```
- #### To create a pod (with name and image name as redis) inside a specific namespace named as team1
    ```
    kubectl run redis --image=redis --namespace=team1
    ```
- #### To switch to the dev namespace permanently;
    ```
    kubectl config set-context $(kubectl config current-context) --namespace=dev
    ```


#### Notes (Imperative Commands):

- --dry-run: By default, as soon as the command is run, the resource will be created. If you simply want to test your command, use the --dry-run=client option. This will not create the resource. Instead, tell you whether the resource can be created and if your command is right.

  - -o yaml: This will output the resource definition in YAML format on the screen.

      Syntax:

      ```
      kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
      ```

      ```
      kubectl create deployment nginx --image=nginx--dry-run=client -o yaml > nginx-deployment.yaml
      ```

#### Notes (Editing Pods and Deployments):

- Syntax:

  ```
  kubectl edit pod <pod name>
  ```

- Syntax: 

  ```
  kubectl edit deployment my-deployment
  ```