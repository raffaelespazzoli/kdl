# Kubernetes Deployment Language

KDL is notation to describe Kubernetes deployments using Kubernetes API objects.


## Introduction
This blog posts illustrates a graphical notation for Kubernetes API objects: Kubernetes Deployment Language (in short KDL). Kubernetes API objects can be used to describe how a solution will be deployed in Kubernetes.

I think there is a need to describe and document how applications will be deployed in Kubernetes, especially when these applications are comprised of several components.
I wanted to create a simple graphical convention to describe these deployments, so that these diagrams could be easily drawn on a whiteboard and or captured in a document.

To better explain the objective we can draw a parallel to [UML](https://en.wikipedia.org/wiki/Unified_Modeling_Language), which had several graphical languages to describe different aspects of an application architecture. A difference with UML is, though, that here in KDL, we don’t have the objective to do forward or reverse engineering (i.e. we don’t convert the diagrams in yaml files or vice versa). This way we have the opportunity to manage how much of information we want to display in the diagrams. As a general rule of thumb we will only display architecturally relevant information.

Here, you can download a [visio stencil](media/kdl.vssx) for the proposed notation.

### Objectives

The objectives of this notation are the following:

- To create a common graphical language to describe how applications are deployed in Kubernetes.
- To represent the most architecturally relevant aspects of the Kubernetes API objects
- To be simple, ideally one person with a whiteboard and some colored markers should be able to create these diagrams

a non-objective of this notation is:

- To auto-generate API Objects definitions

### Color Coding

In general Kubernetes API objects cover the following areas:

| Area  | Color convention  | Example  |
|---|---|---|
| Kubernetes Cluster  | Red  | The Kubernetes cluster(s) involved in the solution  | 
| Compute  | Green  | Deployment  |  
| Networking  | Yellow  | Service  |  
| Storage  | Blue  | Persistent Volume Claim, Persistent Volume  |    
 

## Kubernetes cluster
The Kubernetes cluster is simply represented as a rectangle:

![KubernetesCluster](media/kubernetes-template.png)

All the other API objects will live inside the cluster or at its edges.
There should never be a need to call out individual nodes of an Kubernetes cluster.

You can represents components outside the cluster and how they connect to components inside the cluster. This graphical convention does not cover components outside the cluster.

## Compute 
The compute objects are the most complex. In general they are represented by a rectangle with badges around it to show additional information. Here is a template:

![ComputeTemplate](media/compute-template.png)

The central section of the picture represents a [pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/). In it we can find one or more containers. Both pod and containers should have a name.

On the left side of the pod we have additional compute information. The top badge specify the type of controller for this pod. Here are the types of controllers and their abbreviations:


| Type of controller  | Abbreviation  |  
|---|---|
| [Replication Controller](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)  | RC  | 
| [Replica Set](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)  | RS  |   
| [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)  | D  |  
| [DeploymentConfig](https://docs.openshift.com/container-platform/latest/architecture/core_concepts/deployments.html#deployments-and-deployment-configurations) (OpenShift only)  | DC  | 
| [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)  | DS  |  
| [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)  | SS  |  
| [Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/) | J |
| [Cron Job](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) | J |


On the bottom we have the cardinality of the instances of that pod. This field assumes different meaning and format depending on the type of controller, here is a reference table:

| Type Of Controller  | Format |  
|---|---|
| Replication Controller  | A number or a range (ex 3 or 2:5)  | 
| ReplicaSet  | A number or a range (ex 3 or 2:5)  |  
| Deployment  | A number or a range (ex 3 or 2:5)  | 
| DeploymentConfig (OpenShift only) | A number or a range (ex 3 or 2:5)  |
| DaemonSet  | The node selector: storage-node=true  |
| StatefulSet  | A number: 3  |
| Job  | A number representing the degree of parallelism: 3  |
| Cron Job  | A number representing the degree of parallelism: 3  |



At the top of the pod we have the exposed ports. You can use the little badges to just show the port number or also add the port name. Here is an example:

![PortExample](media/port-example.png)

These badges are in yellow because the represent networking config.
You can connect each port with the container that is actually exposing that port if relevant. But in most cases this will not be necessary because most pods have just one container.

At the bottom of the pod we have the [attached volumes](https://kubernetes.io/docs/concepts/storage/volumes/). The name of the volume should be displayed in the rectangle. In most cases these will be persistent volumes. If the volume type is not persistent volume it may be relevant to show it. Also, sometimes it may be important to also show the mount point. Here are examples of acceptable notation:

![VolumeExample](media/volume-example.png)


On the right side of the pod with have volumes that pertain to the configuration of the pod: [secrets](https://kubernetes.io/docs/concepts/configuration/secret/) and [configmaps](https://kubernetes.io/docs/tasks/configure-pod-container/configmap/). As for the data volumes, the name of the volume should be indicated, usually it is important to distinguish between configmaps and secrets, so also the type of volume should be indicated and if necessary also the mount point can be shown. Here are some examples:

![SecretExample](media/secret-example.png)


## Networking
There are two types of networking objects: [services](https://kubernetes.io/docs/concepts/services-networking/service/) and [ingresses](https://kubernetes.io/docs/concepts/services-networking/ingress/) ([routes](https://docs.openshift.com/container-platform/latest/architecture/core_concepts/routes.html) in OpenShift).

### Services
A service can be represented with an oval as in the following picture:

![ServiceTemplate](media/service-template.png)

On the left side there is a badge representing the type of service. Here are the possible abbreviations

| Type  | Abbreviation  | 
|---|---|
| [Cluster IP](https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies)  | CIP  | 
| [Cluster IP, ClusterIP: None](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)  |  HS a.k.a. Headless Service | 
| [Node Port](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)  | NP  | 
| [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#type-loadbalancer) |  LB |
| [External Name](https://docs.openshift.com/container-platform/3.5/dev_guide/integrating_external_services.html#using-fqdn-2) (OpenShift only) | EN  |
| [External IP](https://kubernetes.io/docs/concepts/services-networking/service/#external-ips) |  EIP |  


At the top of the service there are the exposed ports. Same convention applies here as for the compute ports.

The service should be connected to a compute object. This will implicitly define the service selector, so there is no need to have it indicated in the picture.

If a service is allows traffic from the outside of the cluster to internal pods (such as for Load Balancer or Node Port or External IP) it should be depicted on the edge of the cluster.

![EdgeService](media/edge-service.png)

Same concept applies to services that regulate outbound traffic (such as External Name), although in this case they would probably appear at the bottom of the Kubernetes cluster rectangle.

### Ingresses 
Ingresses can be indicated with a parallelogram as in the following picture:

![IngressTemplate](media/ingress-template.png)

An ingress shows the ingress name and optionally the host exposed. An ingress will be connected to a service (the same rules apply to OpenShift routes). 
Ingresses are always shown at the edge of the OpenShift cluster.  

![EdgeIngress](media/edge-ingress.png)

#### Routes (OpenShift only)

OpenShift routes are represented with the same notation as Ingresses.

## Storage
Storage is used to indicate persistent volumes. The color of storage is blue and it’s shape is a bucket deployed as the following picture:

![StorageTemplate](media/storage-template.png)

Storage should indicate the persistent volume name and the storage provider (example NFS, gluster etc...).
Storage lives always at the edge of the cluster because it is a configuration pointing to an externally available storage.

![EdgeStorage](media/edge-storage.png)

## Putting it all together
In this section we will go over an example of how this notation can be used to describe the deployment of an application.
Our application is an bank service application that uses a mariadb database as its datastore. Being a bank application everything must be in HA.
Here is the deployment diagram:

![mariadb-example](media/mariadb-example.png)

Notice that the mariadb pod uses StatefulSet and a persistent volume for its data. This pod is not exposed externally to the cluster, but its service is consumed by the BankService app.
The BankService app is a stateless pod controlled by a deployment config which has a secret with the credentials to access the database. It also has a service and a route so that it can accept inbound connection from outside the cluster.
