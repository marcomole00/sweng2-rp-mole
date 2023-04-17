---
title: "Research Project - Software Engineering 2"
author:  "Marco Molè"
date: AA 2022/2023
toc: true
numbersections: true
fontsize: 11pt
listings: true
---




# Introduction


Kubernetes, also known as K8s, is an open-source container orchestration platform designed to automate the deployment, scaling, and management of containerized applications. Originally developed by Google and later donated to the Cloud Native Computing Foundation, Kubernetes is now widely used in cloud computing environments to manage the deployment of containerized applications across a cluster of nodes.

At its core, Kubernetes operates by organizing containers into logical units called pods, which are then deployed and managed as a single entity. These pods are designed to be highly portable and can be deployed across a range of cloud and on-premises environments, enabling organizations to build and deploy applications in a way that is both scalable and cost-effective.

One of the key benefits of Kubernetes is its ability to automate many of the tasks associated with managing containerized applications, including load balancing, scaling, and failover. This automation not only helps to reduce the operational overhead of managing large-scale container deployments but also ensures that applications are always available and running smoothly.

Overall, Kubernetes represents a significant advancement in the field of container orchestration, providing developers and operations teams with a powerful tool for managing containerized applications at scale. As such, it has become an essential technology for organizations looking to leverage the benefits of containerization and cloud computing to build more agile, scalable, and resilient applications.

The Kubernetes configuration language is expressed in YAML or JSON format, and it consists of a set of declarative statements that describe the desired state of the Kubernetes resources. These statements include specifications for the containers that run within the resources, as well as other settings such as environment variables, ports, and volumes.

One of the key features of the Kubernetes configuration language is that it allows you to define resources as code, which means that you can store your application configuration in version control systems and apply it to multiple environments. This approach ensures consistency and reduces the potential for human error when deploying and managing Kubernetes applications.


The scope of this report is to analyze the configuration (not true->) options of the language.



## How kubernetes works

At its core, Kubernetes is built around the concept of a cluster, which is a group of machines (called nodes) that work together to run containerized applications. The Kubernetes control plane, which is responsible for managing the cluster, consists of several components that work together to provide a robust and scalable platform for deploying and managing containerized applications.

One of the key concepts in Kubernetes is the pod, which is the smallest deployable unit in the system. A pod is a logical host for one or more containers, and it provides a shared environment for those containers to run in. Each pod has a unique IP address and hostname, which makes it easy to communicate with other pods in the same cluster.

The Kubernetes control plane consists of several components, including the API server, etcd, the scheduler, and the controller manager. The API server provides a RESTful API that users can use to interact with the Kubernetes cluster. Etcd is a distributed key-value store that stores the configuration data for the cluster, such as the desired state of the system. The scheduler is responsible for assigning pods to nodes in the cluster based on various criteria, such as resource availability and affinity. Finally, the controller manager is responsible for ensuring that the current state of the cluster matches the desired state.

Kubernetes also provides several key features that make it easy to manage containerized applications, such as service discovery, load balancing, and auto-scaling. Service discovery allows applications to easily find and communicate with other services in the same cluster. Load balancing ensures that traffic is distributed evenly across all instances of a service, while auto-scaling makes it easy to automatically scale the number of instances up or down based on demand.

In summary, Kubernetes is a powerful tool for managing containerized applications. At its core, Kubernetes is built around the concept of a cluster, which is a group of machines that work together to run containerized applications. The Kubernetes control plane consists of several components that work together to provide a robust and scalable platform for deploying and managing containerized applications. Kubernetes also provides several key features that make it easy to manage containerized applications, such as service discovery, load balancing, and auto-scaling.


## What are the main components
In this section I'll provide a more detailed explanation of the most important components.

### Pods

In Kubernetes, a pod is the smallest deployable unit that can be created, scheduled, and managed. Pods can contain one or more containers, which share the same network namespace and can communicate with each other using inter-process communication (IPC). The declarative language of Kubernetes is used to define pods in a Kubernetes manifest file, which is a YAML or JSON file that describes the desired state of the pod.

To define a pod using Kubernetes' declarative language, the manifest file must include the following information:

1. metadata: This includes information such as the name and labels of the pod.

2. spec: This includes information about the desired state of the pod, such as the containers it should contain, the image(s) to use, and the commands to run. In this section, you can also specify the ports to expose, any environment variables to set, and any volumes to mount.

For example, the following YAML manifest file defines a pod with one container:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: my-container
    image: nginx
    ports:
    - containerPort: 80

```
In this example, the manifest file specifies that the pod should be named "my-pod" and labeled with "app: my-app". It also specifies that the pod should contain one container named "my-container" that runs the Nginx image and exposes port 80.

Once the manifest file has been created, it can be applied to the Kubernetes cluster using the *kubectl apply* command. Kubernetes will then compare the desired state specified in the manifest file with the actual state of the cluster and make any necessary changes to ensure that the desired state is achieved.


### Deployments
A deployment is an object that defines the desired state of a set of replicated pods. Deployments are a higher-level abstraction that enables declarative updates to the desired state of the pod set, allowing for easier management of the underlying resources.

A Kubernetes deployment is responsible for creating and managing a ReplicaSet, which is a Kubernetes object that ensures a specified number of identical replicas of a pod are running at any given time. Deployments provide a way to manage the scaling and rolling updates of pods, by automating the creation and deletion of replica sets as necessary. The deployment object also provides rollback and pause/resume functionality, allowing for more fine-grained control over the lifecycle of the pod set.

When creating a Kubernetes deployment, a user specifies the desired number of replicas, the container images and configurations, and the update strategy. The deployment controller then uses this information to ensure that the desired state of the pod set is achieved and maintained. If the actual state of the pod set deviates from the desired state, the deployment controller automatically performs rolling updates, scaling, or deletion of pods as necessary.

Deployments are a critical component of the Kubernetes platform, as they enable users to manage containerized applications at scale with ease. By abstracting away the complexity of managing individual pods, Kubernetes deployments provide a higher-level interface that is more intuitive for users and reduces the potential for human error. With Kubernetes deployments, users can confidently manage their applications in a highly available and fault-tolerant manner.

To define a deployment using Kubernetes' declarative language, the manifest file must include the following information:

1. metadata: This includes information such as the name and labels of the deployment.

2. spec: This includes information about the desired state of the deployment, such as the number of replicas, the selector that identifies the pods managed by the deployment, and the configuration of the containers.
For example, the following YAML manifest file defines a deployment with 3 replicas:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx
        ports:
        - containerPort: 80
```

In this example, the manifest file specifies that the deployment should be named "my-deployment" and should include three replicas. It also specifies that the deployment should manage pods labeled with "app: my-app" and should create pods with one container running the Nginx image and exposing port 80.

### Services

A Service is an abstraction that enables a stable IP address and DNS name to be assigned to a set of Pods. In essence, it serves as a mechanism for defining a logical grouping of Pods and providing a single point of access to them, regardless of the specific Pod that a client may be communicating with at any given time. Services are typically used to enable load balancing and horizontal scaling of application components running within a Kubernetes cluster.

A Service is defined through a declarative YAML configuration file, which specifies the selector label that matches the Pods to be included in the Service, as well as the port(s) that the Service should expose. When a Service is created, Kubernetes automatically assigns it a cluster-unique IP address and creates an associated DNS entry. Requests sent to the Service's IP address and port are then automatically routed to one of the available Pods that match the Service's selector, based on a simple round-robin algorithm. This allows clients to access the application components running within a Kubernetes cluster in a consistent and scalable manner, without needing to know the specific IP addresses or hostnames of individual Pods.

Service can be configured in several ways depending on the requirements of the application or workload. Some of the common configurations include:

1. ClusterIP: This is the default configuration for a Service and provides a stable IP address and DNS name within the cluster. The Service is only accessible from within the cluster and is not exposed to the external network.

2. NodePort: This configuration exposes the Service on a static port on each node in the cluster. This allows the Service to be accessed from outside the cluster using the node's IP address and the static port number.
3. LoadBalancer: This configuration creates a load balancer in a cloud provider's network, which distributes incoming traffic to the Service across multiple nodes. This is typically used when external traffic needs to be distributed across multiple nodes or when a high degree of availability is required.
4. ExternalName: This configuration provides a way to create a Service that simply maps to an external DNS name. This is useful when an application needs to access an external resource, such as a database or web service, using a stable DNS name.

In addition to these basic configurations, Services can also be combined with other Kubernetes features such as selectors, labels, and endpoints to provide more advanced functionality. For example, Services can be used in conjunction with Ingress controllers to provide an HTTP(S) load balancer that routes traffic based on the URL path or hostname of incoming requests.

### Volumes

A volume is a directory accessible to containers in a Pod. A volume can be used to store data that needs to be shared between containers, or to persist data beyond the lifetime of a container. Volumes in Kubernetes are defined using a declarative configuration file in YAML format.

There are several types of volumes that can be used in Kubernetes, including hostPath volumes, emptyDir volumes, configMap volumes, secret volumes, and persistent volumes. HostPath volumes mount a directory from the host into the container, while emptyDir volumes are created when a Pod is scheduled on a node and deleted when the Pod is terminated. ConfigMap volumes and secret volumes provide a way to inject configuration data or sensitive information into a container, respectively. Persistent volumes are used to store data beyond the lifetime of a Pod, and can be dynamically provisioned or statically created by an administrator.

To define a volume in Kubernetes, the YAML configuration file would include a volumes section with the desired volume type and configuration options. For example, to define a hostPath volume that mounts the /data directory from the host into a container, the YAML file would look like:

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
    volumeMounts:
    - name: my-volume
      mountPath: /data
  volumes:
  - name: my-volume
    hostPath:
      path: /data
```

This configuration file specifies that a hostPath volume named *my-volume* should be mounted at */data *inside the container. The *volumeMounts* field in the container specification specifies which volume to mount, and where to mount it.

## Some examples of the declarative language

In this section I'll look at some examples of the declarative language of Kubernetes and analyze how it helps developers in writing clear, self-explanatory and fine tuned deployments.



# My contribution

## The Problem


When writing the specification of a Pod one can specify the container images to run and the from which registry  pull it from. Container registries are software repositories that store and distribute container images. A container registry is typically used by developers and DevOps teams to store and manage container images, which can be easily shared across different environments, such as development, staging, and production. Container registries provide features like versioning, access control, and image scanning to ensure that container images are secure, reliable, and up-to-date.

Often images require configuration through the use of environment variables.
The documentation of these variables is generally found on the registry itself, to be consulted by the developer when writing the specification of the Pod.

An erroneous configuration of a container is hard to debug, since the point of failure may not be immediately recognizable due to the highly level of coupling of microservices architecture. It could be also a potentially costly error, since most of K8s clusters exist on a pay-per-use cloud environment.


There are no automatic mechanism that inform the developer if the configuration they have written is correct, or at least if it satisfies some minimum configuration requirements.

## A Possible Solution

This mechanism could be included in the development environment, for example as a LSP language server. The Language Server Protocol is an open, JSON-based protocol that allows different editors and IDEs (Integrated Development Environments) to communicate with language-specific servers in a standardized way. By using an LSP server, an editor or IDE can provide advanced features like code completion, error checking, and syntax highlighting for a particular programming language.


Implementing this "type checking" mechanism in the the development environment has a some advantages:

1. highly portable

2. easily extendible with more features and changes

3. More flexible, since the declarative language is only one of the ways to configure kubernetes objects. Secrets and ConfigMap can be stored on external providers

### What registry have to implement

The registries should offer an API endpoint that replies with the configuration option of the requested image.

``` json
{
  "image": "mongo:latest",
  "MONGO_INITDB_ROOT_USERNAME" : {
    "mandatory" : true,
    "type":"string"
  },
  "MONGO_INITDB_ROOT_PASSWORD" : {
    "mandatory" : true,
    "type":"string"
  },
  "parC" : {
    "mandatory" : false,
    "type":"int",
    "default": "admin"
  }
}
```

An example response of the API.
This information can be checked against the PodSpec.containers.env object, which is the object where all the environment variables of the specific containers are specified.


THINGS TO ADD

- very little overhead for the registry

- the LS can cache for a while

- The LS is good because it potentially knows the whole codebase, kubectl only knows what you are applying in that moment.

- talk about how the role of the swDev and the DevOps are gradually diverging. The one who deploys is not the one who wrote the application. This system could facilitate bridging the gap.