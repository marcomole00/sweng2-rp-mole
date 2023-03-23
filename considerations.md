
# introduction

The Kubernetes configuration language is used to define and configure Kubernetes resources, such as deployments, services, and pods, which make up a Kubernetes application.

The Kubernetes configuration language is expressed in YAML or JSON format, and it consists of a set of declarative statements that describe the desired state of the Kubernetes resources. These statements include specifications for the containers that run within the resources, as well as other settings such as environment variables, ports, and volumes.

One of the key features of the Kubernetes configuration language is that it allows you to define resources as code, which means that you can store your application configuration in version control systems and apply it to multiple environments. This approach ensures consistency and reduces the potential for human error when deploying and managing Kubernetes applications.


The scope of this report is to analyze the configuration options of the language.


## a short primer on kubernetes components

- Pods: Pods are the smallest deployable units in Kubernetes. They can contain one or more containers, and they are scheduled to run on nodes in the cluster.

- Services: Services provide a stable IP address and DNS name for a set of pods. They allow other components in the cluster to access the pods using a consistent endpoint.

- Deployments: Deployments manage the rolling updates and rollbacks of a set of pods. They ensure that a specified number of replicas of a pod are running at any given time.

- ConfigMaps and Secrets: ConfigMaps and Secrets are used to store configuration data and sensitive information, respectively. They can be used to configure applications running in pods or to provide environment variables and other configuration information to containers.




# stateless example

In this example is analyzed a multi-tier web application, that is composed by a replicated redis backend and a replicated php frontend.

The files are in ./examples/stateless.

## redis-leader-deployment



### metadata

``` yaml
metadata:
  name: redis-leader
  labels:
    app: redis
    role: leader
    tier: backend
```

The labels are used by the redis-leader service to find the deployment and the corresponding pods.

It's good practice to use a hierarchical system of labels in a cluster to make the architecture of the system explicit.
 In this case, just by reading  the  file redis-leader-deployment.yaml one can guess various aspects such as: 
- app: redis -> the presence of the "app" tag implies that there are several different container images 
- role: leader -> the application, in this case redis, requires a differentiation of roles of the replicas. In this statement we are instantiating the leader. 
- tier:backend -> the system has several tiers, here we are declaring a backend element

### Deployment Specification

The configuration of the deployment is done through an object called DeploymentSpec

We can specify the number of replicas of the leader pods, even though
the redis-follower containers used are not expecting more than one
leader.

This leads to an inherently limitation of the kubernetes language: when setting up a K8s 
Deployment the architectural requirements of the software must be known
to the person writing the deployment file. There is no mechanism in the
Kubernetes configuration language that it's able to infer and enforce  these requirements on the writer. 
For example, container X has to have at least/at most N replicas.

#### Pod Specification

In the PodSpec (Object that describes the Pod that will be replicated by
the Deployment) one of the most interesting configuration possibilities
is the "resources" object that states the minimum and maximum amount of
computing resources of each container.

Being able to tailor the resources used for each container is a useful
feature in a pay-per-use cloud world.

``` yaml
 resources:
  requests:
    cpu: 100m
    memory: 100Mi
  limits:
    cpu: cpu-max
    memory: ram_max
```

## redis-leader-service

As said before the service uses a label system to identify the
deployment to bind to.

The main freedom degree in the services is the ServiceSpec.type option.

In this case, it's set to the default value "ClusterIP" which creates an
IP in the internal network of the cluster. This is done because we want
the redis leader endpoint to be reachable only by the other internal
components of our cluster, like the redis followers and the front end.

## redis-follower-deployment

## redis-follower-service

## frontend-deployment

Here we can see another form of the configuration of the containers
through the use of environment variables.

``` yaml
env:
  - name: GET_HOSTS_FROM
        value: "dns"
```

Most of the containers obtainable through an online registry, like
docker.io or gcr.io, offer configuration options via environment
variables.

## frontend-service

The frontend should be exposed to outside the cluster network, hence the
ServiceSpec.type is set to "LoadBalancer", which means that the service
will be exposed via an external load balancer.

The kubernetes configuration language forces the developer to explicitly
state if a service is exposed outside the cluster virtual network. From
a security standpoint, in a service-oriented architecture it's nice to
have all internal services not exposed by default.

# Stateful

This example analyzes the deployment of a Wordpress site and a MySQL database.
Both applications use PersistentVolumes and PersistentVolumeClaims to store data, thus are considered to be stateful.

The files are in ./example/stateful/: 
1. kustomization.yaml 
  - secret generator (for mysqldb credentials) 
  - MySQL resource configs 
  - wordpress resource configs 
2. wordpress-deployment.yaml 
  - wordpres-service 
  - wp-pv-claim 
  - wordpress (actual deployment) 
3. mysql-deployment.yaml 
  - wordpress-mysql (service used by wordpress to access mysql db) 
  - mysql-pv-claim - wordpress-mysql (deployment of the mysql instance)

## kustomization.yaml

A new method for organizing and applying declaration file is presented: kustomize.

kustomize uses a file called kustomization.yaml that contains the
reference to all the yaml file of the k8s configuration to be applied,
together with additional customization of those resources.For
example is possible to declare secret and configMaps directly in the
kustomization.yaml, like in the example.

``` yaml
secretGenerator:
- name: mysql-pass
  literals:
  - password=example
```

This approach makes more explicit and easy to read all the applied configuration options. 
It's also more straightforward to modify an exisiting declaration, since all the configuration options are in one file.

It's a possible to apply "patches" that override the "base" declaration.

## wordpress-deployment

Since both wordpress and mysql require a persistent storage solution,
i.e. a volume with a lifecycle that's indipendent from the pod's lifecycle, a PersistentVolumeClaim is used.

```yaml
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

A PVC is an object that represent a request for storage. Since no
StorageClassName attribute was defined in the pvc spec, the default type
of volume was provioned. Since i'm testing all of this on linode.com, a linode volume of 20Gi was automatically provisioned.

In general a PVC is a request for storage that is decoupled from the details of how that storage will be provisioned or where it will be located in the cluster.
The storage class defines the type and parameters of the storage needed. It allows cluster administrators to decouple the storage infrastructure from the cluster infrastructure.

Storage classes define the following attributes:

- Provisioner: The name of the Kubernetes storage provisioner that will be used to create PVs.
- Parameters: Storage-specific parameters that will be passed to the provisioner.
- Reclaim policy: Defines how PVs will be reclaimed when they are no longer needed.
- Mount options: Additional mount options that will be applied when the PV is mounted.

The linode environment offers two classes, which differ only in the reclaim policy.




# from here


- Research the limitation of the language on the enforcing of the container requirements
  - che cosa si potrebbe fare 
  - ricerca su stack over flow / et simila su quanto sia un problema reale.
  

# On enforcing container requirments:
 When writing the specification of a pod one can specify the image to run and the registry to pull it from. The registry is just a private or public repository  that host the container images. 

As said before  images require configuration through the use of  Environment Variables. An example use of EV can be found in the case studies.

An erranous configuration of a container  is hard to debug, since the point of failure may not be immediately recognizable due to the highly level of coupling of microservices architecture.  It could be also a potentially costly error,  remember that most of k8s cluster live on a pay-per-use cloud environment.

The documentation of these variables is generally found on the registry itself, to be consulted by the developer when writing the specification of the Pod.

There are no automatic mechanism that inform the developer if the configuration they have written is correct, or at least if it satisfies some minimum configuration requirements.

An example of  configuration requirement for a database image could be: 
- admin username ->  mandatory
- admin password ->  mandatory
- public port -> not mandatory , defaults to a well known port number

This mechanism could be included in the development environment, for example as a vscode plugin. Implementing this mechanism into the k8s cli utility is nearly useless, due to the complexity of the sofware and the potentially escalation of API calls done to the registry endpoint. This is a check that is best done before testing the actual configuration on the k8s runtime.



The registries should offer an API endpoint that replies with the configuration option of the requested image.

Informations about the configuration options could be
- mandatory or facultative (ie have a meaningful and/or well know default value)
- type of the configuration options -> int, float, string , some other kind of structured data (like json?)

``` json
{
  "image": "imagename:version",
  "parA" : {
    "mandatory" : true,
    "type":"int"
  },
  "parB" : {
    "mandatory" : true,
    "type":"int"
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
