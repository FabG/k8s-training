# Chapter 3 Kubernetes - APIs and Access


### 1 - API Access
Everything is an API call, going to `kube-apiserver`

`kubectl` makes API calls on your behalf. You can also make calls externally, using curl or other program. With the appropriate
certs and keys, you can make requests or pass json files to make configuration changes.
```
$ curl --cert userbob.pem --key userBob-key.pem \
--cacert /path/to/ca.pem \
https://k8sServer:6443/api/v1/pods
```
The ability to impersonate other users or groups, subject to RBAC configuration, allows a manual override authentication. This
can be helpful for debugging authorization policies of other users.

*Notes*: the `.kube/config` file has all the information related to the cluster, IP, certificate autohrity,...

To note, the APIs are under:
```
ls .kube/cache/discovery/10.128.0.45_6443/
admissionregistration.k8s.io  autoscaling            events.k8s.io                 rbac.authorization.k8s.io
apiextensions.k8s.io          batch                  extensions                    scheduling.k8s.io
apiregistration.k8s.io        certificates.k8s.io    flowcontrol.apiserver.k8s.io  servergroups.json
apps                          coordination.k8s.io    networking.k8s.io             storage.k8s.io
authentication.k8s.io         crd.projectcalico.org  node.k8s.io                   v1
authorization.k8s.io          discovery.k8s.io       policy
```

We can see the verbs:
```
$ python3 -m json.tool .kube/cache/discovery/10.128.0.45_6443/v1/serverresources.json
{
    "kind": "APIResourceList",
    "apiVersion": "v1",
    "groupVersion": "v1",
    "resources": [
        {
            "name": "bindings",
            "singularName": "",
            "namespaced": true,
            "kind": "Binding",
            "verbs": [
                "create"
            ]
        },
        {
            "name": "componentstatuses",
            "singularName": "",
            "namespaced": false,
            "kind": "ComponentStatus",
            "verbs": [
                "get",
                "list"
            ],
            "shortNames": [
                "cs"
            ]
        },
        {
            "name": "configmaps",
            "singularName": "",
            "namespaced": true,
            "kind": "ConfigMap",
            "verbs": [
                "create",
                "delete",
                "deletecollection",
                "get",
                "list",
                "patch",
                "update",
                "watch"
            ],
            "shortNames": [
                "cm"
            ],
            "storageVersionHash": "qFsyl6wFWjQ="
...
```


While there is more detail on security in a later chapter, it is helpful to check the current authorizations, both as an admin, as
well as another user, using `auth can-i`. The following shows what user bob could do in the default namespace and the developer namespace:
```
$ kubectl auth can-i create deployments
yes
$ kubectl auth can-i create deployments --as bob
no
$ kubectl auth can-i create deployments --as bob --namespace developer
yes
```


The default serialization for API calls must be `JSON`. There is an effort to use Google’s protobuf serialization, but this remains experimental. While we may work with files in YAML format, they are converted to and from JSON.
Kubernetes uses the resourceVersion value to determine API updates and implement optimistic concurrency. In other words, an object is not locked from the time it has been read until the object is written.
Instead, upon an updated call to an object, the resourceVersion is checked and a `409 CONFLICT` is returned should the
number have changed. The resourceVersion is currently backed via the modifedIndex parameter in the etcd database,
and is unique to the namespace, kind and server. Operations which do not change an object such as WATCH or GET do not update this value.
This means Clients must handle 409 CONFLICT Errors.


### 2. Annotations
`Labels` are used to work with objects or collections of objects; `annotations` are not.

Instead `annotations` allow for metadata to be included with an object that may be helpful outside of Kubernetes object interaction. Similar to labels, they are key to value maps. They also are able to hold more information, and more human readable information than labels.
Having this kind of metadata can be used to track information such as a timestamp, pointers to related objects from other
ecosystems, or even an email from the developer responsible for that object’s creation.
The annotation data could otherwise be held in an exterior database, but that would limit the flexibility of the data. The more
this metadata is included, the easier to integrate management and deployment tools or shared client libraries.
For example, to annotate only Pods within a namespace, then overwrite the annotation and finally delete it:
```
$ kubectl annotate pods --all description='Production Pods' -n prod
$ kubectl annotate --overwrite pod webpod description="Old Production Pods" -n prod
$ kubectl -n prod annotate pod webpod description-
```


### 3. Working with A Simple Pod
As discussed earlier, a Pod is the lowest compute unit and individual object we can work with in Kubernetes. It can be a single
container, but often it will consist of a primary application container and one or more supporting containers.
Below is an example of a simple pod manifest in YAML format. You can see the apiVersion, the kind, the metadata, and its
spec, which define the container that actually runs in this pod:
```
apiVersion: v1
kind: Pod
metadata:
 name: firstpod
spec:
 containers:
 - image: nginx
  name: stan
```
Few required, many optional:
- `apiVersion`: Must match existing API group
- `kind`: The type of object to create
- `metadata`: At least a name
- `spec` What to create and parameters

You can use the `kubectl creat`e command to create this pod in Kubernetes. Once it is created, you can check its status
with `kubectl get pods`. Output is omitted to save space:
```
$ kubectl create -f simple.yaml
$ kubectl get pods
$ kubectl get pod firstpod -o yaml
$ kubectl get pod firstpod -o json
```

### 4. kubectl and API

Kubernetes exposes resources via RESTful API calls, which allows all resources to be managed via HTTP, JSON or even XML. the typical
protocol being HTTP. The state of the resources can be changed using standard HTTP verbs (e.g. GET, POST, PATCH, DELETE, etc.).

`kubectl` has a verbose mode argument which shows details from where the command gets and updates information. Other output includes
curl commands you could use to obtain the same result. While the verbosity accepts levels from zero to any number, there is currently no
verbosity value greater than ten. You can check this out for kubectl get. The output below has been formatted for clarity:
```
$ kubectl --v=10 get pods firstpod
1 ....
2 I1215 17:46:47.860958 29909 round_trippers.go:417]
3 curl -k -v -XGET -H "Accept: application/json"
4 -H "User-Agent: kubectl/v1.8.5 (linux/amd64) kubernetes/cce11c6"
5 https://10.128.0.3:6443/api/v1/namespaces/default/pods/firstpod
6 ....
```
If you delete this pod, you will see that the HTTP method changes from XGET to XDELETE.
```
$ kubectl --v=10 delete pods firstpod
1 ....
2 I1215 17:49:32.166115 30452 round_trippers.go:417]
3 curl -k -v -XDELETE -H "Accept: application/json, */*"
4 -H "User-Agent: kubectl/v1.8.5 (linux/amd64) kubernetes/cce11c6"
5 https://10.128.0.3:6443/api/v1/namespaces/default/pods/firstpod
6 ....
```

The primary tool used from the command line will be `kubectl`, which calls curl on your behalf. You can also use the curl command from outside the cluster to view or make changes.
The basic server information, with redacted TLS certificate information can be found in the output of
```
$ kubectl config view
```
If you view verbose output from a previous page, you will note that the first line references a config file where this information
is pulled from, `˜/.kube/config .`


The term `namespace` is used to both reference the Kernel feature as well as the segregation of API objects by Kubernetes.
Both are means to keep resources distinct.
Every API call includes a namespace, using default if not otherwise declared: https://10.128.0.3:6443/api/v1/namespaces/
default/pods.
Namespaces are intended to isolate multiple groups and the resources they have access to work with via quotas. Eventually,
access control policies will work on namespace boundaries as well. One could use Labels to group resources for
administrative reasons.
There are four namespaces when a cluster is first created:
- `default`: This is where all resources are assumed unless set otherwise.
- `kube-node-lease`: The namespace where worker node lease information is kept.
- `kube-public`: A namespace readable by all, even those not authenticated. General information is often included in this namespaces.
- `kube-system`: Contains infrastructure pods.

Should you want to see all resources on a system you must pass the `--all-namespaces` option to the kubectl command.


#### API Resources with kubectl
- All available via kubectl
- `kubectl [command] [type] [Name] [flag]``
- `kubectl help` for more information
- Abbreviated names

#### 5. Swagger and OpenAPI
The entire Kubernetes API uses a Swagger specification. This is evolving towards the OpenAPI initiative. It is extremely
useful, as it allows, for example, to auto-generate client code. All the stable resources definitions are available on the documentation
site.
You can browse some of the API groups via a Swagger UI at https://swagger.io/specification/.
