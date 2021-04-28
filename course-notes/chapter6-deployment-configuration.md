# Chapter 6 Kubernetes - Deployment configuration

In this chapter, we are going to cover some of the typical tasks necessary for full application deployment inside of Kubernetes.
We will begin by talking about attaching storage to our containers. By default, storage would only last as long as the container
using it. Through the use of Persistent Volumes and Persistent Volume Claims, we can attach storage such that it lives longer
than the container and/or the pod the container runs inside of. We’ll cover adding dynamic storage, which might be made
available from a cloud provider, and we will talk about ConfigMaps and secrets. These are ways of attaching configuration
information into our containers in a flexible manner. They are ingested in almost the exact same way. The difference between
them, currently, is a secret is encoded. There are plans for it to be encrypted in the future. We will also talk about how do we
update our applications. We can do a rolling update, where there’s always some containers available for response to client
requests, or update all of them at once. We’ll also cover how to roll back an application to a previous version, looking at our
entire history of updates, and choosing a particular one.

#### 1. Volumes Overview
- Container engines have traditionally not offered storage that outlives the container. As containers are considered transient, this could lead to a loss of data, or complex exterior storage options. By default, this storage is `ephemeral` - it lives as long as the container, not the pod.
- A Kubernetes `volume` shares at least the Pod lifetime, not the containers within. Should a container terminate, the data would continue to be available to the new container.
A volume can persist longer than a pod, and can be accessed by multiple pods, using `PersistenVolumeClaims`, also called `pv`. This allows for **state persistence**.
- A volume is a directory, possibly pre-populated, made available to containers in a Pod. The creation of the directory, the backend storage of the data and the contents depend on the volume type. As of v1.14, there are 28 different volume types ranging from rbd to for Ceph, to NFS, to dynamic volumes from a cloud provider like Google’s gcePersistentDisk. Each has particular configuration options and dependencies.
- Adoption of `Container Storage Interface (CSI)` enables the goal of an industry standard interface for container orchestration
to allow access to arbitrary storage systems. Currently, volume plugins are ”in-tree”, meaning they are compiled and built with the core Kubernetes binaries. This “out-of-tree” object will allow storage vendors to develop a single driver
and allow the plugin to be containerized. This will replace the existing Flex plugin which requires elevated access to the
host node, a large security concern.
- Should you want your storage lifetime to be distinct from a Pod, you can use `Persistent Volumes`. These allow for empty or pre-populated volumes to be claimed by a Pod using a `Persistent Volume Claim`, then outlive the Pod. Data inside the volume could then be used by another Pod, or as a means of retrieving data
- There are two API Objects which exist to provide data to a Pod already. `Encoded` data can be passed using a Secret and `non-encoded` data can be passed with a ConfigMap. These can be used to pass important data like SSH keys, passwords, or even a configuration file like /etc/hosts.
- `dynamic storage` - An api call goes to a provider which makes the volume at that moment.


#### 2. Introducing Volumes
- A Pod specification can declare one or more volumes and where they are made available. Each requires a name, a
type, and a mount point.
- Keeping acquired data or ingesting it into other containers is a common task, typically requiring the use of a `Persistent Volume Claim` (PVC)
- The same volume can be made available to multiple containers within a Pod, which can be a method of container-to-container
communication. A volume can be made available to multiple Pods, with each given an `access mode` to write. There is no concurrency checking, which means data corruption is probable, unless outside locking takes place. So don't write to the same file or use some lock.
- *Notes*: Kubernetes does not order writes or check coherency

- Access Modes
 - `RWO` (ReadWriteOnce) - All pods on first node to mount get write. All pods on other nodes only have read access
 - `ROX` (ReadOnlyMany)
 - `RWX` (ReadWriteMany)


- *kubelet* - first gets all resouces in the podspec, then asks container engine to start containers
 - IP - via pause container
 - mounts all of the storage, creates symbolic link to ephemeral directory
    downloads configMaps and secrets.
 - Only if all resources are available does it tell container engine to start containers for pod
 - Stays in `pending` state if unable to mount or download resources.


#### 3. Volume Spec
- One of the many types of storage available is an `emptyDir`. The kubelet will create the directory in the container, but not mount any storage. Any data created is written to the shared container space. As a result, it would not be persistent storage. When the Pod is destroyed, the directory would be deleted along with the container.
 - More info at: https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
- Another is `hostpath`. A hostPath volume mounts a file or directory from the host node's filesystem into your Pod. This is not something that most Pods will need, but it offers a powerful escape hatch for some applications.
 - More info at: https://kubernetes.io/docs/concepts/storage/volumes/#hostpath


#### 4 Volume Types
- There are several types that you can use to define volumes, each with their pros and cons. Some are local, and many
make use of network-based resources.
- In GCE or AWS, you can use volumes of type `GCEpersistentDisk` or `awsElasticBlockStore`, which allows you to mount GCE and EBS disks in your Pods, assuming you have already set up accounts and privileges.
- `emptyDir` and `hostPath` volumes are easy to use. As mentioned, emptyDir is an empty directory that gets erased when the Pod dies, but is recreated when the container restarts. The hostPath volume mounts a resource from the
host node filesystem. The resource could be a directory, file socket, character, or block device. These resources must
already exist on the host to be used. There are two types, DirectoryOrCreate and FileOrCreate, which create the
resources on the host, and use them if they don’t already exist.
- `NFS` (Network File System) and `iSCSI` (Internet Small Computer System Interface) are straightforward choices for multiple
readers scenarios.
• `rbd` for block storage or CephFS and GlusterFS, if available in your Kubernetes cluster, can be a good choice for multiple
writer needs.

#### 5. Persistent Volumes and Claims
- A persistent volume (`pv`) is a storage abstraction used to retain data longer than the Pod using it. Pods define a volume of type persistentVolumeClaim (pvc) with various parameters for size and possibly the type of backend storage known
as its StorageClass. The cluster then attaches the persistentVolume.
- Kubernetes will dynamically use volumes that are available, irrespective of its storage type, allowing claims to any
backend storage.
- There are several phases to persistent storage:
 - `Provisioning` can be from pvs created in advance by the cluster administrator, or requested from a dynamic source,
such as the cloud provider.
 - `Binding` occurs when a control loop on the master notices the PVC, containing an amount of storage, access
request, and optionally, a particular StorageClass. The watcher locates a matching PV or waits for the
StorageClass provisioner to create one. The pv must match at least the storage amount requested, but may
provide more.
 -  The `use` phase begins when the bound volume is mounted for the Pod to use, which continues as long as the Pod
requires.
 -  `Releasing` happens when the Pod is done with the volume and an API request is sent, deleting the PVC. The
volume remains in the state from when the claim is deleted until available to a new claim. The resident data
remains depending on the persistentVolumeReclaimPolicy.
 - The `reclaim` phase has three options:
 - `Retain`, which keeps the data intact, allowing for an administrator to handle the storage and data.
  - `Delete` tells the volume plugin to delete the API object, as well as the storage behind it.
  - The `Recycle` option runs an rm -rf /mountpoint and then makes it available to a new claim. With the
stability of dynamic provisioning, the Recycle option is planned to be deprecated.
```
$ kubectl get pv
$ kubectl get pvc
```

Check `rook.io`
- https://rook.io/
- https://www.linux.com/training-tutorials/kubernetes-getting-started-with-rook/ceph.com


#### 6. Secrets

- Pods can access local data using volumes, but there is some data you don’t want readable to the naked eye. Passwords may be an example. Someone reading through a YAML file may read a password and remember it. Using the Secret
API resource, the same password could be encoded. A casual reading would not give away the password. You can create, get, or delete secrets:
```
$ kubectl get secrets
```
- Secrets can be manually encoded with kubectl create secret:
```
$ kubectl create secret generic --help
$ kubectl create secret generic mysql --from-literal=password=root
```
- A secret is **not encrypted by default**, only `base64-encoded`. You can see the encoded string inside the secret with `kubectl`. The secret will be decoded and be presented as a string saved to a file. The file can be used as an environmental variable or in a new directory, similar to the presentation of a volume.
- In order to encrypt secrets one must create a `EncryptionConfiguration` object with a key and proper identity. Then the `kube-apiserver` needs the --encryption-provider-config flag set to a previously configured provider such as
aescbc or ksm . Once this is enabled you need to recreate every secret as they are encrypted upon write. Multiple keys are possible. Each key for a provider is tried during decryption. The first key of the first provider is used for encryption.
- To rotate keys first create a new key, restart (all) kube-apiserver processes, then recreate every secret.

#### 7. Using Secrets via Environment Variables
- A secret can be used as an environmental variable in a Pod.
- There is no limit to the number of Secrets used, but there is a `1MB` limit to their size. Each secret occupies memory,
along with other API objects, so very large numbers of secrets could deplete memory on a host.
- They are stored in the `tmpfs` storage on the host node, and are only sent to the host running Pod. All volumes requested
by a Pod must be mounted before the containers within the Pod are started. So, a secret must exist prior to being
requested.

#### 8. Mounting Secrets as Volumes
- You can also mount secrets as files using a volume definition in a pod manifest. The mount path will contain a file whose
name will be the key of the secret created with the kubectl create secret step earlier.
- • Once the pod is running, you can verify that the secret is indeed accessible in the container:
```
$ kubectl exec -ti busybox -- cat /mysqlpassword/password
LFTr@1n
```

#### 9. Portable Data with ConfigMaps
- A similar API resource to Secrets is the `ConfigMap`, except the data is not encoded. In keeping with the concept of decoupling in Kubernetes, using a ConfigMap decouples a container image from configuration artifacts.
- They store data as sets of key-value pairs or plain configuration files in any format. The data can come from a collection
of files or all files in a directory. It can also be populated from a literal value.
- A ConfigMap can be used in several different ways. A Pod can use the data as environmental variables from one or
more sources. The values contained inside can be passed to commands inside the pod. A Volume or a file in a Volume
can be created, including different names and particular access modes. In addition, cluster components like controllers
can use the data.

#### 10. Using ConfigMaps
- Like secrets, you can use ConfigMaps as environment variables or using a volume mount. They must exist prior to
being used by a Pod, unless marked as optional. They also reside in a specific namespace.
- In the case of environment variables, your pod manifest will use the valueFrom key and the configMapKeyRef value to
read the values


#### 11. Scaling and Rolling Updates
- The API server allows for the configurations settings to be updated for most values. There are some immutable values,
which may be different depending on the version of Kubernetes you have deployed.
- A common update is to change the number of replicas running. If this number is set to zero, there would be no containers,
but there would still be a `ReplicaSet` and `Deployment`. This is the backend process when a Deployment is deleted.
```
$ kubectl scale deploy/dev-web --replicas=4
deployment "dev-web" scaled
$ kubectl get deployments
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
dev-web 4 4 4 1 12m
```
- Non-immutable values can be edited via a text editor, as well. Use edit to trigger an update. For example, to change the deployed version of the nginx web server to an older version:

```
$ kubectl edit deployment nginx
....
containers:
- image: nginx:1.8 #<<---Set to an older version
imagePullPolicy: IfNotPresent
name: dev-web
...
```
- This would trigger a rolling update of the deployment. While the deployment would show an older age, a review of the Pods would show a recent update and older version of the web server application deployed


#### 12. Deployment Rollbacks
- With all the ReplicaSets of a Deployment being kept, you can also roll back to a previous revision by scaling up and
down the ReplicaSets the other way. Next, we will have a closer look at rollbacks, using the --record option of the
kubectl command, which allows annotation in the resource definition. The create generator does not have a record
function.
```
$ kubectl set image deployment ghost --image=ghost:0.9 --record
$ kubectl get deployments ghost -o yaml
metadata:
 annotations:
  deployment.kubernetes.io/revision: "1"
  kubernetes.io/change-cause: kubectl set image deployment ghost --image=ghost0.9 --record
```

- hould an update fail, due to an improper image version, for example, you can roll back the change to a working version with `kubectl rollout` undo:
```
$ kubectl set image deployment/ghost ghost=ghost:0.9 --all
$ kubectl rollout history deployment/ghost deployments "ghost":
 REVISION CHANGE-CAUSE
 1 <none>
 2 kubectl set image deployment/ghost ghost=ghost:09 --all```
```

```
$ kubectl get pods
 NAME READY STATUS RESTARTS AGE
 ghost-2141819201-tcths 0/1 ImagePullBackOff 0 1m
$ kubectl rollout undo deployment/ghost
$ kubectl get pods
 NAME READY STATUS RESTARTS AGE
 ghost-3378155678-eq5i6 1/1 Running 0 7s
```

- You can roll back to a specific revision with the `--to-revision=2` option. You can also edit a Deployment using the
kubectl edit command. One can also pause a Deployment, and then resume.
```
$ kubectl rollout pause deployment/ghost
$ kubectl rollout resume deployment/ghost
```
