# Chapter 4 Kubernetes - API Objects

#### 1. API objects
This chapter is about additional API resources or objects. We will learn about resources in the v1 API group, among others.
Stability increases and code becomes more stable as objects move from alpha versions, to beta, then v1 indicating stability.
`DaemonSets`, which ensure a Pod on every node, and `StatefulSets`, which stick a container to a node and otherwise act like
a deployment, have progressed to apps/v1 stability.
Role-based Access Control (`RBAC`), essential to security has made the leap from v1alpha1 to the stable v1 status.

As a fast moving project keeping track of changes, and possible changes can be an important part of ongoing system administration. Release notes, as well as discussions to release notes can be found in version-dependent subdirectories
at: https://github.com/kubernetes/enhancements/. For example, the release feature status can be found here: https://kubernetes.io/docs/setup/release/notes/.
Starting with v1.16 deprecated API object versions will respond with an error instead of being accepted. This is an important
change from historic behavior.


#### 2. The v1 Group
The v1 API is no longer a single group, but rather a collection of groups for each main object category. For example there is a
v1 group, a storage.k8s.io/v1 group, and rbac.authorization.k8s.io/v1 etc... Currently there are eight v1 groups.
We have touched on several objects in lab exercises. Here are details for some of them:
- `Node`
Represents a machine (physical or virtual) that is part of your Kubernetes cluster. You can get more information
about nodes with the kubectl get nodes command. You can turn on and off the scheduling to a node with the
kubectl cordon/uncordon commands.
- `Service Account`
Provides an identifier for processes running in a pod to access the API server and performs actions that it is authorized
to do.
- `Resource Quota`
It is an extremely useful tool, allowing you to define quotas per namespace. For example, if you want to limit a specific
namespace to only run a given number of pods, you can write a resourcequota manifest, create it with kubectl and
the quota will be enforced.
- `Endpoint`
Generally, you do not manage endpoints. They represent the set of IPs for pods that match a particular service. They
are handy when you want to check that a service actually matches some running pods. If an endpoint is empty, then it
means that there are no matching pods and something is most likely wrong with your service definition.


#### 3. API Resources
Using the `kubectl` create command we can quickly deploy an application. We have looked at the Pods created running the application, like nginx. Looking closer you will find that a Deployment was created which manages a ReplicaSet which then deploys the Pod. Lets take a closer look at each object.
- `Deployment` - A controller which manages the state of ReplicaSets and the pods within. The higher level control allows
for more flexibility with upgrades and administration. Unless you have a good reason, use a deployment.
- `ReplicaSet` - Orchestrates individual Pod life cycle and updates. These are newer versions of Replication Controllers
which differ only in selector support.
- `Pod` - As we’ve mentioned the lowest unit we can manage, runs the application container, possibly support containers

**DaemonSets**:
- Ensures every node runs a single pod
- Similar to ReplicaSet
- Often used for logging, metrics and security pods.
- Can be configured to avoid nodes
Should you want to have a logging application on every node a DaemonSet may be a good choice. The controller ensures that
a single pod, of the same type, runs on every node in the cluster. When a new node is added to the cluster a Pod, same as
deployed on the other nodes, is started. When the node is removed the DaemonSet makes sure the local Pod is deleted.
As usual, you get all the CRUD operations via kubectl:
```
$ kubectl get daemonsets
$ kubectl get ds
```

**StatefulSet**:
- Similar to Deployment
- Ensures unique pods
- Guarantee ordering
- Stable in v1.9


Pods deployed using a `StatefulSet` use the same Pod specification. How this is different than a `Deployment` is that a
StatefulSet considers each Pod as unique and provides ordering to Pod deployment.

In order to track each Pod as a unique object the controller uses identity composed of stable storage, stable network identity,
and an ordinal. This identity remains with the node regardless to which node the Pod is running on at any one time.
The default deployment scheme is sequential starting with 0, such as app-0, app-1, app-2 etc. A following Pod will not launch
until the current Pod reaches a running and ready state. They are not deployed in parallel. V 1.20


**Autoscaling**:
- Agents which add or remove resources from the cluster.
- Horizontal Pod Autoscaling (HPA)
 - Scale based on current CPU usage, or custom metric
 - Must have Metrics Server or custom component running
- Vertical Pod Autoscaler (under development)
- Cluster Autoscaler (CA)
 - Add or remove nodes based on utilization
 - Makes request to cloud provider
 - Pods which cannot be evicted prevent scale-down

In the autoscaling group we find the `Horizontal Pod Autoscalers (HPA)`. This is a stable resource. HPAs automatically scale Replication Controllers, ReplicaSets, or Deployments based on a target of 50% CPU usage by default. The
usage is checked by kubelet every 30 seconds and retrieved by Metrics Server API call every minute. HPA checks with Metrics Server every 30 seconds. Should a Pod be added or removed HPA waits 180 seconds before further action.
Other metrics can be used and queried via REST. The autoscaler does not collect the metrics, it only makes a request for the
aggregated information and increases or decreases the number of replicas to match the configuration.

The `Cluster Autoscaler (CA)` adds or removes nodes to the cluster based off of inability to deploy a Pod or having nodes
with low utilization for at least 10 minutes. This allows dynamic requests of resources from the cloud provider and minimizes
expense for unused nodes. If you are using CA nodes should be added and removed through cluster-autoscaler- commands.
Scale-up and down of nodes is checked every 10 seconds, but decisions are made on a node every 10 minutes.
Should a scale-down fail the group will be rechecked in 3 minutes, with the failing node being eligible in five minutes. The total
time to allocate a new node is largely dependent on the cloud provider.

Another project still under development is the Vertical Pod Autoscaler. This component will adjust the amount of CPU and memory requested by Pods.


**Jobs**
- Part of `Batch` API group
- `Jobs` run Pod until number of completions reached
 - Batch processing or one-off Pods
 - Ensure specified number of pods successfully terminate
 - Can run multiple Pods in parallel
- `Cronjob` to run Pod on regular basis
 - Creates a Pod about once per executing time
 - Some issues, job should be idempotent
 - Can run in serial or parallel
 - Same time syntax as Linux cron job


#### 4. RBAC API
The last API resources that we will look at are in the rbac.authorization.k8s.io group. We actually have four resources:
`ClusterRole`, `Role`, `ClusterRoleBinding`, and `RoleBinding`.
They are used for Role Based Access Control (RBAC) to Kubernetes.
```
$ curl localhost:8080/apis/rbac.authorization.k8s.io/v1
```

These resources allow us to define Roles within a cluster and associate users to these Roles. For example, we can define a Role for someone who can only read pods in a specific namespace, or a Role that can create deployments, but no services.
Note: More on RBAC is covered later in the Security chapter.
