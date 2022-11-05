## **DEPLOYING APPLICATIONS INTO KUBERNETES CLUSTER** ###

In this project, I built upon the knowledge of Kubernetes architecture, and begin to deploy applications on a K8s cluster. Kubernetes has a lot of moving parts; it operates with several layers of abstraction between applications and host machines where it runs.

**Choosing the right Kubernetes cluster set up**

When it comes to using a Kubernetes cluster, there is a number of options available depending on the ultimate use of it. For example, if you just need a cluster for development or learning, you can use lightweight tools like Minikube, or k3s. These tools can run on your workstation without heavy system requirements. Obviously, there is limit to the amount of workload you can deploy there for testing purposes, but it works exactly like any other Kubernetes cluster.

On the other hand, if you need something more robust, suitable for a production workload and with more advanced capabilities such as horizontal scaling of the worker nodes, then you can consider building own Kubernetes cluster from scratch just as you did in Project 21.
Other options will be to leverage a Managed Service Kubernetes cluster from public cloud providers such as: AWS EKS, Microsoft AKS, or Google Cloud Platform GKE. There are so many more options out there. Regardless of whichever one you choose, the experience is usually very similar.

Most organisations choose Managed Service options for obvious reasons such as:

1. Less administrative overheads
1. Reduced cost of ownership
1. Improved Security
1. Seamless support
1. Periodical updates to a stable and well-tested version
1. Faster cluster spin up

However, there is usually strong reasons why organisations with very strict compliance and security concerns choose to build their own Kubernetes clusters. Most of the companies that go this route will mostly use on-premises data centres. When there is need to store data privately due to its sensitive nature, companies will rather not use a public cloud provider. Because, if they do, they have no idea of the physical location of the data centre in which their data is being persisted. Banks and Governments are typical examples of this.

Some setup options can combine both public and private cloud together. For example, the master nodes, etcd clusters, and some worker nodes that run stateful applications can be configured in private datacentres, while worker nodes that require heavy computations and stateless applications can run in public clouds. This kind of hybrid architecture is ideal to satisfy compliance, while also benefiting from other public cloud capabilities.

![](hybrid_k8s.png)

**Deploying the Tooling app using Kubernetes objects**
I wrote configuration files for Kubernetes objects (they are usually referred as manifests) in the form of files with yaml syntax and deploy them using kubectl console. But first, let us understand what a Kubernetes object is.
Kubernetes objects are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:

- What containerized applications are running (and on which nodes)
- The resources available to those applications
- The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance

These objects are "record of intent" – once you create the object, the Kubernetes system will constantly work to ensure that the object exists. By creating an object, you are effectively telling the Kubernetes system what you want your cluster’s workload to look like; this is your cluster’s desired state.

To work with Kubernetes objects – whether to create, modify, or delete them – you will need to use the Kubernetes API. When you use the kubectl command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you. It is also possible to use curl to directly interact with the Kubernetes API, or it can be as part of developing a program in different programming languages.

#### **COMMON KUBERNETES OBJECTS** ###

- Pod
- Namespace
- ResplicaSet (Manages Pods)
- DeploymentController (Manages Pods)
- StatefulSet
- DaemonSet
- Service
- ConfigMap
- Volume
- Job/Cronjob

The very first concept to understand is the difference between how Docker and Kubernetes run containers – with Docker, every docker run command will run an image (representing an application) as a container. The running container is a Docker’s smallest entity, it is the most basic deployable object. Kubernetes on the other hand operates with **pods** instead of containers, a pods encapsulates a container. Kubernetes uses pods as its smallest, and most basic deployable object with a unique feature that allows it to run multiple containers within a single Pod. It is not the most common pattern – to have more than one container in a Pod, but there are cases when this capability comes in handy.

In the world of docker, or docker compose, to run the Tooling app, you must deploy separate containers for the application and the database. But in the world of Kubernetes, you can run both: application and database containers in the same Pod. When multiple containers run within the same Pod, they can directly communicate with each other as if they were running on the same localhost. Although running both the application and database in the same Pod is NOT a recommended approach.

A Pod that contains one container is called single container pod and it is the most common Kubernetes use case. A Pod that contains multiple co-related containers is called multi-container pod. There are few patterns for multi-container Pods; one of them is the sidecar container pattern – it means that in the same Pod there is a main container and an auxiliary one that extends and enhances the functionality of the main one without changing it.

### Understanding the common YAML fields for every Kubernetes object ###
Every Kubernetes object includes object fields that govern the object’s configuration:

**kind:** Represents the type of kubernetes object created. It can be a Pod, DaemonSet, Deployments or Service.
**version:** Kubernetes api version used to create the resource, it can be v1, v1beta and v2. Some of the kubernetes features can be released under beta and available for general public usage.
**metadata:** provides information about the resource like name of the Pod, namespace under which the Pod will be running,
labels and annotations.
**spec:** consists of the core information about Pod. Here we will tell kubernetes what would be the expected state of resource, Like container image, number of replicas, environment variables and volumes.
**status:** consists of information about the running object, status of each container. Status field is supplied and updated by Kubernetes after creation. This is not something you will have to put in the YAML manifest.

#### Deploying a random Pod ####
Lets see what it looks like to have a Pod running in a k8s cluster. This section is just to illustrate and get you to familiarise with how the object’s fields work. Lets deploy a basic Nginx container to run inside a Pod.

**apiVersion** is v1
**kind** is Pod
**metatdata** has a name which is set to nginx-pod
The **spec** section has further information about the Pod. Where to find the image to run the container – (This defaults to Docker Hub), the port and protocol.

1. Create a **Pod** yaml manifest on your master node
