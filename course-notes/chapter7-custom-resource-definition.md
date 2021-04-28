# Chapter 7 Kubernetes - Custom Resource Definition


#### 1. Overview
We have been working with built-in resources, or API endpoints. The flexibility of Kubernetes allows for **dynamic addition of
new resources** as well. Once these Custom Resources have been added the objects can be created and accessed using
standard calls and commands like `kubectl`.

The creation of a new object stores new structured data in the `etcd` database and
allows access via `kube-apiserver`.

To make a new, custom resource part of a declarative API there needs to be a `controller` to retrieve the structured data
continually and act to meet and maintain the declared state. This controller, or `operator`, is an agent to create and mange
one or more instances of a specific stateful application. We have worked with built-in controllers such for Deployments,
DaemonSets and other resources.
The functions encoded into a custom operator should be all the tasks a human would need to perform if deploying the
application outside of Kubernetes. The details of building a custom controller are outside the scope of this course and not
included.
There are two ways to add custom resources to your Kubernetes cluster. The easiest, but less flexible, way is by adding
a Custom Resource Definition (**CRD**) to the cluster. The second which is more flexible is the use of Aggregated APIs which
requires a new API server to be written and added to the cluster.
Either way of a new object to the cluster, as distinct from a built-in resource, is called a Custom Resource.
If you are using RBAC for authorization you probably will need to grant access to the new CRD resource and controller. If using
an Aggregated API, you can use the same or different authentication process.

*Notes*: Don't turn CRDs and Operators into a way to "hide" the VM you really want to make

Resources:
 - Operator Framework: open source toolkit to manage K8s native apps, called Operators, in an effectice, automated and scalable way.
- https://operatorframework.io/
- https://github.com/operator-framework/community-operators/tree/master/community-operators
- https://sdk.operatorframework.io/docs/building-operators/helm/tutorial/



**WHAT IS AN OPERATOR?**
The goal of an Operator is to put operational knowledge into software. Previously this knowledge only resided in the minds of administrators, various combinations of shell scripts or automation software like Ansible. It was outside of your Kubernetes cluster and hard to integrate. With Operators, CoreOS changed that.

Operators implement and automate common Day-1 (installation, configuration, etc) and Day-2 (re-configuration, update, backup, failover, restore, etc.) activities in a piece of software running inside your Kubernetes cluster, by integrating natively with Kubernetes concepts and APIs. We call this a Kubernetes-native application. With Operators you can stop treating an application as a collectionof primitives like Pods, Deployments, Services or ConfigMaps, but instead as a single object that only exposes the knobs that make sense for the application.


#### 2. Custom Resource Definitions

As we have learned decoupled nature of Kubernetes depends on a collection of watcher loops, or controllers, interrogating
the kube-apiserver to determine if a particular configuration is true. If the current state does not match the declared state the
controller makes API calls to modify the state until they do match.

If you add a new API object and controller you can use the
existing kube-apiserver to monitor and control the object. The addition of a Custom Resource Definition will be added to
the cluster API path, currently under `apiextensions.k8s.io/v1`.

While this is the easiest way to add a new object to the cluster it may not be flexible enough for your needs. Only the existing
API functionality can be used. Objects must respond to REST requests and have their configuration state validated and stored
in the same manner as built-in objects. They would also need to exist with the protection rules of built-in objects.
A CRD allows the resource to be deployed in a namespace or available in the entire cluster. The YAML file sets this with the
scope: parameter which can be set to Namespaced or Cluster.


Configuration Example:
```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
 name: backups.stable.linux.com
spec:
 group: stable.linux.com
 version: v1
 scope: Namespaced
 names:
  plural: backups
  singular: backup
  shortNames:
  - bks
  kind: BackUp
```

- `apiVersion`: Should match the current level of stability, currently apiextensions.k8s.io/v1
- `kind: CustomResourceDefinition`: The object type being inserted by the kube-apiserver.
- `name: backups.stable.linux.com` The name must match the spec field declared later. The syntax must be ``<plural name>.<group>.``
- `group: stable.linux.com`  The group name will become part of the REST API under /apis/<group>/<version> or /apis/stable/v1 in this case with
the version set to v1.
- `scope`: Determines if the object exists in a single namespace or is cluster-wide.
- `plural`: Defines the last part of the API URL such as apis/stable/v1/backups.
- `singular` and `shortNames`: represent the name with displayed and make CLI usage easier.
- `kind`: A CamelCased singular type used in resource manifests.



New Object Configuration:
```
apiVersion: "stable.linux.com/v1"
kind: BackUp
metadata:
 name: a-backup-object
spec:
 timeSpec: "* * * * */5"
 image: linux-backup-image
 replicas: 5
```

Note that the `apiVersion` and `kind` match the CRD we created in a previous step. The spec parameters depend on the controller.
The object will be evaluated by the controller. If the syntax, such as timeSpec does not match the expected value you
will receive and error, should validation be configured. Without validation only the existence of the variable is checked, not its
details.


#### 3. Aggregated APIs
The use of `Aggregated APIs` allows adding additional Kubernetes-stype API servers to the cluster. The added server acts
as a subordinate to kube-apiserver which as of v1.7 runs the aggregation layer in-process. When an extension resource is
registered the aggregation layer watches a passed URL path and proxy any requests to the newly registered API service.
The aggregation layer is easy to enable. Edit the flags passed during startup of the kube-apiserver to include
--enable-aggregator-routing=true. Some vendors enable this feature by default.
The creation of the exterior can be done via YAML configuration files or APIs. Configuring TLS auth between components
and RBAC rules for various new objects is also required. A sample API server is available on github here: https://github.com/
kubernetes/sample-apiserver. A project currently in incubation stage is an API server builder which should handle much of
the security and connection configuration. It can be found here: https://github.com/kubernetes-incubator/apiserver-builder
V 1.20
