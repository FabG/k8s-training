# Cheat Sheet / Misc notes



Pods: there are several pod types such as:
 - All calls go through the `kube-api-server`. It is not the thinker
 - another pod is `kube-controler-manager` is the "thinker"
 - `scheduler` - decides where a pod gets placed
 - `etcd` keeps the persistence of the cluster
 - `kubeproxy` - handling the networking, running on every single node and work with `kubelet`.

 What is the job of `kubelet`?
 It is not a pod. It's a systemctl linux service that runs on everynode, and talks to `docker`. It mounts stuff, make sure we have an IP address,...


Nodes:
 - A `node` is a worker machine. It may be virtual or a physical machine.
 - each node is managed by the Master.


To deploy / interact with the cluster:
 - we can run `curl` request of `kubectl` that runs curl behind the scene.
 - `kupe-apiserver` will handle 3 things:
  - will check if we are `authenticated`.
  - it will then check if we are `autorized`, via rbac
  - then the 3rd phase called `admission controllers` validates or change/mutates
  - Example: calling a company and asking to talk to the CEO.
   - who is calling => authenticated
   - admission controller: not allowed to talk to CEO but let me connect you to someone that can help.
 - `kube-apiserver` will persist any info about what needs to be on the cluster in `etcd`.
 - Moments later (in micro-seconds), the `kube-controller-manager` makes a request of the `kube-apiserver` and asks: are there any changes to the spec? At this point, yes. There is a change - there is a "deployment" called "tim2". It asks back - is there a deployment with a label "tim2". If not, it makes a deployment. Then it creates a replicaset and so on.


 Once pod is assigned to a node, via the `kube-scheduler`, a container run a pod.

 To scale the `tim2` pod to scale to 2:
 `kubectl scale deployment tim2 --replicas=2`

 the new pod will then be assigned to the node, based on what `kube-scheduler` says. Once `kubelet` running on the target node has all the resources it needs (IP address for the new pod, filesystem mounter,...), it tells `docker` to start a pod called "tim2". This starts a container. We now have a replica count of 2.
 kubelet gets volume, secrets, configmap and IP address. It tells docker to run the container. Docker then download the container image.


IP Addresses:
 - `Pod IP` is ephemeral, as opposed to `cluster IP` that will remain over time. the `cluster IP` is tied to a service.
 - we can connect to the service, hitting from outside the public IP of the master node, and adding the port where the service is exposed, to end up reaching the pod in one of the worker nodes.


Security - check OPA
https://www.openpolicyagent.org/
