# Chapter 8 Kubernetes - Scheduling

#### 1. Overview
The larger and more diverse a Kubernetes deployment becomes the more administration of scheduling can be important. The
`kube-scheduler` determines which nodes will run a Pod.
Users can set the priority of a pod, which will allow preemption of lower priority pods. The eviction of lower priority pods would
then allow the higher priority pod to be scheduled.

The scheduler tracks the set of nodes in your cluster, filters them based on a set of `predicates`, then uses `priority`
functions to determine on which node each Pod should be scheduled.

The Pod spec as part of a request is sent to the
kubelet on the node for creation.

The default scheduling decision can be affected through the use of `Labels` on nodes or Pods. Labels of `podAffinity`, `taints`,
and `pod bindings` allow for configuration from the Pod or the node perspective. Some like `tolerations` allow a Pod to work
with a Node, even when the Node has a taint that would otherwise preclude a Pod being scheduled.
Not all labels are drastic. `Affinity` settings may encourage a Pod to be deployed on a node, but would deploy the Pod elsewhere
if the node was not available. Sometimes documentation may use the term require, but practice shows the setting to be more
of a request. As beta features expect the specifics to change. Some settings will evict Pods from a node should the required
condition no longer be true such as requiredDuringSchedulingRequiredDuringExecution.
Others options, like a custom scheduler, need to be programmed and deployed into your Kubernetes cluster.


#### 2. Scheduler Settings
The scheduler goes through a set of filters, or `predicates` to find available nodes, then ranks each node using
`priority` functions. The node with the highest rank is selected to run the Pod.

The predicates, such as `PodFitsHost` or `NoDiskConflict` are evaluated in a particular and configurable order. In this way
a node has the least amount of checks for new Pod deployment, which can be useful to exclude a node from unnecessary
checks if node is not in the proper condition.

For example, there is a filter called `HostNamePred`, which is also known as HostName, which filters out nodes that do not match
the node name specified in the pod specification. Another predicate for `PodFitsResources` to make sure that the available
CPU and memory can fit the resources required by the Pod.

The scheduler can be updated by passing a configuration of kind: `Policy` which can order predicates, give special weights
to priorities and even `hardPodAffinitySymmetricWeight` which deploys Pods such that if we set Pod A to run with Pod B,
then Pod B should automatically be run with Pod A.


##### Priorities
Unless Pod and node affinity has been configured, the `SelectorSpreadPriority` setting, which ranks nodes based on the
number of existing running pods, will select the node with the least amount of Pods. This is a basic way to spreading Pods
across the cluster.

Other priorities can be used for particular cluster needs. The `ImageLocalityPriorityMap` favors nodes which already have
downloaded container images. The total sum of image size is compared with the largest having highest priority, but does not
check the image about to be used.

There currently are more than ten included priorities which range from checking the existence of a label to choosing a node
with the most requested CPU and memory usage.
A feature stable as of v1.14 allows the setting of a PriorityClass and assigning pods via the use of PriorityClassName
settings. This allows user to preempt, or evict, lower priority pods so that their higher priority pods can be scheduled. The
`kube-scheduler` determines a node where the pending pod could run if one or more existing pods were evicted. If a node is
found the low priority pod(s) are evicted and the higher priority pod scheduled. The use of a Pod Disruption Budget (PDB)
is a way to limit the number of pods preemption evicts to ensure enough pods remain running. The scheduler will remove pods
even if the PDB is violated if no other options are available



#### 3. Policies
The default scheduler contains a number of predicates and priorities; however, these can be changed via a scheduler policy
file.
Typically, you will configure a scheduler with this policy using the `--policy-config-file` parameter and define a name for
this scheduler using the `--scheduler-name` parameter. You will then have two schedulers running and will be able to specify
which scheduler to use in the Pod specification.
With multiple schedulers there could be conflict in Pod allocation. Each Pod should declare which scheduler should be used.
But if separate schedulers determine a node is eligible because of available resources and both attempt to deploy, causing
the resource to no longer be available, a conflict would occur. The current solution is for the local `kubelet` to return the Pods
to the scheduler for reassignment. Eventually one Pod will succeed and the other be scheduled elsewhere.


##### Pod Specification
Most scheduling decisions can be made as part of the the Pod spec. The `nodeName` and `nodeSelector` options allow a Pod
to be assigned to a single node or a group of nodes with particular labels.

`Affinity` and `anti-affinity` can be used to require or prefer which node is used by the scheduler. If using a preference instead a
matching node is chosen first, but other nodes would be used if no match is present.

The use of `taints` allows a node to be labeled such that Pods would not be scheduled for some reason, such as the master
node after initialization. A `toleration` allows a Pod to ignore the taint and be scheduled assuming other requirements are met.
Should none of these options meet the needs of the cluster there is also the ability to deploy a custom scheduler. Each Pod could then include a schedulerName to choose which schedule to use.

The pod would remain `Pending` until a node is found with the matching labels.


#### 4. Affinity Rules
Pods which may communicate a lot or share data may operate best if co-located, which would be a form of `affinity`.
For greater fault tolerance you may want Pods to be as separate as possible, which would be `anti-affinity`.

These settings are used by the
scheduler based on labels of Pods already running. As a result the scheduler must interrogate each node and track the labels
of running Pods. Clusters larger than several hundred nodes may see significant performance loss.

The use of `requiredDuringSchedulingIgnoredDuringExecution` means that the Pod will not be scheduled on a node
unless the following operator is true. If the operator changes to become false in the future the Pod will continue to run. This
could be seen as a hard rule.

Similar is `preferredDuringSchedulingIgnoredDuringExecution` which will choose a node with the desired setting before
those without. Should no properly labeled nodes be available the Pod will execute anyway. This is more of a soft settings
which declares a preference instead of a requirement.

With the use of podAffinity the scheduler will try to schedule Pods together. The use of podAntiAffinity would cause the scheduler to keep Pods on different nodes.
The topologyKey allows a general grouping of Pod deployment. Affinity (or the inverse anti-affinity) will try to run on nodes
with the declared topology key and running Pods with a particular label.

The `topologyKey` could be any legal key, but there some important considerations:
• If using `requiredDuringScheduling` and the admission controller LimitPodHardAntiAffinityTopology setting the
topologyKey must be set to kubernetes.io/hostname.
• If using `PreferredDuringScheduling` an empty topologyKey is assumed to be all, or the combination of
kubernetes.io/hostname, topology.kubernetes.io/zone
and topology.kubernetes.io/region.

Example of podAffinity:
```
spec:
 affinity:
  podAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:
   - labelSelector:
    matchExpressions:
    - key: security
     operator: In
     values:
     - S1
    topologyKey: topology.kubernetes.io/zone
```

#### 5 Taints and Tolerations
- Expressed as key=value:effect
- key and value value created by admin
- Effect must be `NoSchedule`, `PreferNoSchedule`, or `NoExecute`
- Only nodes can be tainted currently
- Multiple `taints` possible on a node


A node with a particular `taint` will repel Pods without `tolerations` for that taint.

The key and value used can be any legal string, while allows flexibility to prevent Pods from running on nodes based off of
any need. If a Pod does not have an existing toleration the scheduler will not consider the tainted node.
There are three effects, or ways to handle Pod scheduling.
- `NoSchedule` The scheduler will not schedule a Pod on this node unless the Pod has this toleration. Existing Pods
continue to run, regardless of toleration.
- `PreferNoSchedule` The scheduler will avoid using this node unless there are no untainted nodes for the Pods
toleration. Existing Pods are unaffected.
- `NoExecute` This taint will cause existing Pods to be evacuated and no future Pods scheduled. Should an existing Pod
have a toleration it will continue to run. If the Pod tolerationSeconds value is set the Pod will remain for that many
seconds then be evicted. Certain node issues will cause kubelet to add 300 second tolerations to avoid unnecessary
evictions.

If a node has multiple taints the scheduler ignores those with matching tolerations. The remaining un-ignored taints
have their typical effect.
The use of TaintBasedEvictions is still an alpha feature. The kubelet uses taints to rate-limit evictions when the node
has problems.

*Notes*: asking the instructor about our use case, to segregate compute for internal vs external jobs, he agreed that taints & tolerations is a good use.
He also mentioned that if we had anything more complex (more envts to seggregate) to use affinity.
*Notes*: `affinity` deals with `labels`, `taints` only deals with `toleration`. We could use affinity to seggregate the jobs running across our customers, each having its dedicated note. `requirednodeaffinity` is the label we would use,
