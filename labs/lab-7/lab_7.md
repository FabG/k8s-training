# K8s Lab 7
Related to Chapter #8 - Custom Resource Definition (p.162)


#### Exercise 8.1: Create a Custom Resource Definition
Overview: The use of CustomResourceDefinitions (CRD), has become a common manner to deploy new objects and operators.
Creation of a new operator is beyond the scope of this course, basically it is a watch-loop comparing a spec to the current status, and making changes until the states match.

A good discussion of creating a controller can be found here:
- https://coreos.com/blog/introducing-operators.html.

First we will examine an existing CRD, then make a simple CRD, but without any particular action. It will be enough to find the
object ingested into the API and responding to commands.

1. View the existing CRDs.
```
student@master:˜$ kubectl get crd --all-namespaces
NAME                                                  CREATED AT
bgpconfigurations.crd.projectcalico.org               2021-04-26T18:53:05Z
bgppeers.crd.projectcalico.org                        2021-04-26T18:53:05Z
blockaffinities.crd.projectcalico.org                 2021-04-26T18:53:05Z
clusterinformations.crd.projectcalico.org             2021-04-26T18:53:05Z
felixconfigurations.crd.projectcalico.org             2021-04-26T18:53:05Z
globalnetworkpolicies.crd.projectcalico.org           2021-04-26T18:53:05Z
globalnetworksets.crd.projectcalico.org               2021-04-26T18:53:05Z
hostendpoints.crd.projectcalico.org                   2021-04-26T18:53:05Z
ipamblocks.crd.projectcalico.org                      2021-04-26T18:53:05Z
ipamconfigs.crd.projectcalico.org                     2021-04-26T18:53:05Z
ipamhandles.crd.projectcalico.org                     2021-04-26T18:53:05Z
ippools.crd.projectcalico.org                         2021-04-26T18:53:05Z
kubecontrollersconfigurations.crd.projectcalico.org   2021-04-26T18:53:05Z
networkpolicies.crd.projectcalico.org                 2021-04-26T18:53:05Z
networksets.crd.projectcalico.org                     2021-04-26T18:53:05Z
```

2. We can see from the names that these CRDs are all working on Calico, out network plugin. View the `calico.yaml` file
we used when we initialized the cluster to see how these objects were created, and some CRD templates to review.
```
student@master:˜$ grep calico k8sMaster.sh
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
student@master:~$ wget https://docs.projectcalico.org/manifests/calico.yaml
student@master:˜$ less calico.yaml
# Source: calico/templates/calico-config.yaml
# This ConfigMap is used to configure a self-hosted Calico installation.
kind: ConfigMap
apiVersion: v1
metadata:
  name: calico-config
  namespace: kube-system
data:
  # Typha is disabled.
  typha_service_name: "none"
  # Configure the backend to use.
  calico_backend: "bird"

  # Configure the MTU to use for workload interfaces and tunnels.
  # By default, MTU is auto-detected, and explicitly setting this field should not be required.
  # You can override auto-detection by providing a non-zero value.
  veth_mtu: "0"
...
"name": "k8s-pod-network",
"cniVersion": "0.3.1",
"plugins": [
  {
    "type": "calico",
    "log_level": "info",
    "log_file_path": "/var/log/calico/cni/cni.log",
    "datastore_type": "kubernetes",
    "nodename": "__KUBERNETES_NODE_NAME__",
    "mtu": __CNI_MTU__,
    "ipam": {
        "type": "calico-ipam"
    },
    "policy": {
        "type": "k8s"
    },
...
```

3. Now that we have seen some examples, we will create a new YAML file.

```
student@master:˜$ vim crd.yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: crontabs.training.lfs458.com
         # This name must match names below.
         # <plural>.<group> syntax
spec:
  scope: Cluster    #Could also be Namespaced
  group: training.lfs458.com
  version: v1
  names:
    kind: CronTab      #Typically CamelCased for resource manifest
    plural: crontabs   #Shown in URL
    singular: crontab  #Short name for CLI alias
    shortNames:
    - ct               #CLI short name
```

4. Add the new resource to the cluster.
```
student@master:˜$ kubectl create -f crd.yaml
Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io/crontabs.training.lfs458.com created
```

5. View and describe the resource. The new line may be in the middle of the output. You’ll note the describe output is
unlike other objects we have seen so far.
```
student@master:˜$ kubectl get crd | grep lfs458
crontabs.training.lfs458.com                          2021-04-28T18:14:01Z
```

```
student@master:˜$ kubectl describe crd crontab<Tab>
student@master:˜$ kubectl describe crd crontabs.training.lfs458.com
Name:         crontabs.training.lfs458.com
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  apiextensions.k8s.io/v1
Kind:         CustomResourceDefinition
Metadata:
  Creation Timestamp:  2021-04-28T18:14:01Z
  Generation:          1
  Managed Fields:
    API Version:  apiextensions.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        f:acceptedNames:
...
Spec:
  Conversion:
    Strategy:  None
  Group:       training.lfs458.com
  Names:
    Kind:       CronTab
    List Kind:  CronTabList
    Plural:     crontabs
    Short Names:
      ct
    Singular:               crontab
  Preserve Unknown Fields:  true
  Scope:                    Cluster
  Versions:
    Name:     v1
    Served:   true
    Storage:  true
```


6. Now that we have a new API resource we can create a new object of that type. In this case it will be a crontab-like
image, which does not actually exist, but is being used for demonstration.
```
student@master:˜$ vim new-crontab.yaml
apiVersion: "training.lfs458.com/v1"
    # This is from the group and version of new CRD
kind: CronTab
    # The kind from the new CRD
metadata:
  name: new-cron-object
spec:
  cronSpec: "*/5 * * * *"
  image: some-cron-image
    #Does not exist
```

7. Create the new object and view the resource using short and long name.
```
student@master:˜$ kubectl create -f new-crontab.yaml
crontab.training.lfs458.com/new-cron-object created
student@master:˜$ kubectl get ct
NAME              AGE
new-cron-object   17s
```

```
student@master:˜$ kubectl describe ct
Name:         new-cron-object
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  training.lfs458.com/v1
Kind:         CronTab
Metadata:
  Creation Timestamp:  2021-04-28T18:17:44Z
  Generation:          1
  Managed Fields:
    API Version:  training.lfs458.com/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:cronSpec:
        f:image:
    Manager:         kubectl-create
    Operation:       Update
    Time:            2021-04-28T18:17:44Z
  Resource Version:  225162
  UID:               20fbbc16-b532-4d9c-9bf7-dc6da01ec92d
Spec:
  Cron Spec:  */5 * * * *
  Image:      some-cron-image
Events:       <none>
```

8. To clean up the resources we will delete the CRD. This should delete all of the endpoints and objects using it as well.
```
student@master:˜$ kubectl delete -f crd.yaml
customresourcedefinition.apiextensions.k8s.io
"crontabs.training.lfs458.com" deleted
student@master:˜$ kubectl get ct
Error from server (NotFound): Unable to list "training.lfs458.com/v1, Resource=crontabs": the server could not find the requested resource (get crontabs.training.lfs458.com)
```
