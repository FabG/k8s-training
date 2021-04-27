# K8s Lab 1
This lab is related to Chapter 2 - Kubernetes Architecture 

This covers Exercise 2.1 to 2.6 (p29 - p45)
We will create a two-node Ubuntu 18.04 cluster. Using two nodes allows an understanding of some issues and configurations found in a production environment. Currently 2 vCPU and 8G of memory allows for quick labs. Other Linux distributions should work in a very similar manner, but have not been tested.

*Important Notes*: Regardless of the platform used (VirtualBox, VMWare, AWS, GCE or even bare metal) please remember that security software like SELinux, AppArmor, and firewall configurations can prevent the labs from working. While not something to do in production consider disabling the firewall and security software.
GCE requires a new VPC to be created and a rule allowing all traffic to be included. The use of wireshark can be a
helpful place to start with troubleshooting network and connectivity issues if you’re unable to open all ports.
The `kubeadm` utility currently requires that swap be turned off on every node. The `swapoff -a` command will do this until the next reboot, with various methods to disable swap persistently. Cloud providers typically deploy instances with swap disabled.

### SSH into each node
Using the pem key provided before the training, run:
```
$ chmod 400 ../keys/LF-Class.pem
$ ssh -i ../keys/LF-Class.pem student@34.72.190.13
```
From another terminal, run:
```
 ssh -i ../keys/LF-Class.pem student@35.202.199.29
```

I am now connected to each node:
```
student@master-73kx:~$
```
and
```
student@worker-8f1k:~$
```

### Download shell scripts and YAML files
To assist with setting up your cluster please download the tarball of shell scripts and YAML files. The `k8sMaster.sh` and `k8sSecond.sh` scripts deploy a Kubernetes cluster using kubeadm and use Project Calico for networking.

Should the file not be found you can always use a browser to investigate the parent directory.
```
wget https://training.linuxfoundation.org/cm/LFD5461/LFD5461_V1.20_SOLUTIONS.tar.xz --user=LFtraining --password=Penguin2014
```
Check you have the file:
```
$ ls -ltr
total 68
-rw-rw-r-- 1 student student 62192 Mar 26 21:19 LFD5461_V1.20_SOLUTIONS.tar.xz
```

Unzip and untar it:
```
student@worker-8f1k:~$ tar -xvf LFD5461_V1.20_SOLUTIONS.tar.xz
LFD5461/SOLUTIONS/
LFD5461/SOLUTIONS/s_08/
LFD5461/SOLUTIONS/s_08/EXAMPLES/
LFD5461/SOLUTIONS/s_08/EXAMPLES/resources/
LFD5461/SOLUTIONS/s_08/EXAMPLES/resources/labs/
LFD5461/SOLUTIONS/s_08/EXAMPLES/resources/labs/new-crontab.yaml
LFD5461/SOLUTIONS/s_08/EXAMPLES/resources/labs/crd.yaml
LFD5461/SOLUTIONS/s_08/EXAMPLES/resources/index.tex
LFD5461/SOLUTIONS/s_08/EXAMPLES/resources/overview.tex
LFD5461/SOLUTIONS/s_08/EXAMPLES/resources/aggregated.tex
LFD5461/SOLUTIONS/s_08/EXAMPLES/resources/labs.tex
LFD5461/SOLUTIONS/Makefile
LFD5461/SOLUTIONS/s_10/
LFD5461/SOLUTIONS/s_10/clusterrole.yaml
LFD5461/SOLUTIONS/s_10/rolebinding.yaml
LFD5461/SOLUTIONS/s_10/second.yaml
LFD5461/SOLUTIONS/s_10/allclosed.yaml
LFD5461/SOLUTIONS/s_10/secret.yaml
LFD5461/SOLUTIONS/s_10/security-review1.yaml
LFD5461/SOLUTIONS/s_10/serviceaccount.yaml
LFD5461/SOLUTIONS/s_14/
LFD5461/SOLUTIONS/s_14/EXAMPLES/
LFD5461/SOLUTIONS/s_14/EXAMPLES/helm/
LFD5461/SOLUTIONS/s_14/EXAMPLES/helm/index.tex
LFD5461/SOLUTIONS/s_14/EXAMPLES/helm/helm.tex
LFD5461/SOLUTIONS/s_14/EXAMPLES/helm/overview.tex
LFD5461/SOLUTIONS/s_14/EXAMPLES/helm/usinghelm.tex
LFD5461/SOLUTIONS/s_14/EXAMPLES/helm/labs.tex
LFD5461/SOLUTIONS/s_04/
LFD5461/SOLUTIONS/s_04/EXAMPLES/
LFD5461/SOLUTIONS/s_04/EXAMPLES/access/
LFD5461/SOLUTIONS/s_04/EXAMPLES/access/api.tex
LFD5461/SOLUTIONS/s_04/EXAMPLES/access/kubectl.tex
LFD5461/SOLUTIONS/s_04/EXAMPLES/access/labs/
LFD5461/SOLUTIONS/s_04/EXAMPLES/access/labs/curlpod.json
LFD5461/SOLUTIONS/s_04/EXAMPLES/access/index.tex
LFD5461/SOLUTIONS/s_04/EXAMPLES/access/swagger.tex
LFD5461/SOLUTIONS/s_04/EXAMPLES/access/annotations.tex
LFD5461/SOLUTIONS/s_04/EXAMPLES/access/firstpod.tex
LFD5461/SOLUTIONS/s_04/EXAMPLES/access/labs.tex
LFD5461/SOLUTIONS/s_12/
LFD5461/SOLUTIONS/s_12/troubleshoot-review1.yaml
LFD5461/SOLUTIONS/s_02/
LFD5461/SOLUTIONS/s_02/basic-later.yaml
LFD5461/SOLUTIONS/s_02/k8sSecond.sh
LFD5461/SOLUTIONS/s_02/architecture-review1.yaml
LFD5461/SOLUTIONS/s_02/k8sMaster.sh
LFD5461/SOLUTIONS/s_02/basic.yaml
LFD5461/SOLUTIONS/s_02/basicservice.yaml
LFD5461/SOLUTIONS/s_02/basicservice-AllDone.yaml
LFD5461/SOLUTIONS/s_11/
LFD5461/SOLUTIONS/s_11/service_first.yaml
LFD5461/SOLUTIONS/s_11/service.yaml
LFD5461/SOLUTIONS/s_11/ingress.rule.yaml
LFD5461/SOLUTIONS/s_11/traefik-ds.yaml
LFD5461/SOLUTIONS/s_11/ingress.rbac.yaml
LFD5461/SOLUTIONS/s_06/
LFD5461/SOLUTIONS/s_06/edited-stress.yaml
LFD5461/SOLUTIONS/s_06/cronjob.yaml
LFD5461/SOLUTIONS/s_06/stress.yaml
LFD5461/SOLUTIONS/s_06/cron-job.yaml
LFD5461/SOLUTIONS/s_06/design-review1.yaml
LFD5461/SOLUTIONS/s_06/design-review2.yaml
LFD5461/SOLUTIONS/s_06/job.yaml
LFD5461/SOLUTIONS/s_07/
LFD5461/SOLUTIONS/s_07/PVol.yaml
LFD5461/SOLUTIONS/s_07/basic.yaml-with-edits
LFD5461/SOLUTIONS/s_07/CreateNFS.sh
LFD5461/SOLUTIONS/s_07/weblog-pv.yaml
LFD5461/SOLUTIONS/s_07/car-map.yaml
LFD5461/SOLUTIONS/s_07/simpleapp.yaml-with-edits
LFD5461/SOLUTIONS/s_07/pvc.yaml
LFD5461/SOLUTIONS/s_07/weblog-pvc.yaml
LFD5461/SOLUTIONS/s_07/weblog-configmap.yaml
LFD5461/SOLUTIONS/s_09/
LFD5461/SOLUTIONS/s_09/EXAMPLES/
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/labs/
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/labs/vip.yaml
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/labs/taint.yaml
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/index.tex
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/taints.tex
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/policies.tex
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/scheduler.tex
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/affinityrules.tex
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/overview.tex
LFD5461/SOLUTIONS/s_09/EXAMPLES/scheduling/labs.tex
LFD5461/SOLUTIONS/s_05/
LFD5461/SOLUTIONS/s_05/EXAMPLES/
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/sets.tex
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/rbac.tex
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/labs/
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/labs/cronjob.yaml
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/labs/cron-job.yaml
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/labs/job.yaml
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/index.tex
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/overview.tex
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/versionone.tex
LFD5461/SOLUTIONS/s_05/EXAMPLES/APIobjects/labs.tex
LFD5461/SOLUTIONS/s_03/
LFD5461/SOLUTIONS/s_03/edited-later-simpleapp.yaml
LFD5461/SOLUTIONS/s_03/vol2.yaml
LFD5461/SOLUTIONS/s_03/docker-compose.yaml
LFD5461/SOLUTIONS/s_03/SimpleHarbor.sh
LFD5461/SOLUTIONS/s_03/build-review1.yaml
LFD5461/SOLUTIONS/s_03/edited-localregistry.yaml
LFD5461/SOLUTIONS/s_03/Dockerfile
LFD5461/SOLUTIONS/s_03/simple.py
LFD5461/SOLUTIONS/s_03/vol1.yaml
LFD5461/SOLUTIONS/s_03/edited-simpleapp.yaml
LFD5461/SOLUTIONS/s_13/
LFD5461/SOLUTIONS/s_13/EXAMPLES/
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/monitoring.tex
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/index.tex
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/bootseqence.tex
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/podlevel.tex
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/overview.tex
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/troubleshooting.tex
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/logging.tex
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/plugins.tex
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/labs.tex
LFD5461/SOLUTIONS/s_13/EXAMPLES/LFS458_troubleshooting/HA-labs.tex
LFD5461/SOLUTIONS/LICENSE
LFD5461/SOLUTIONS/genmake
```


#### Exercise 2.2 - deploy a new cluster
Deploy a Master Node using Kubeadm
Review the script to install and begin the configuration of the master kubernetes server. Y

```
$ find $HOME -name k8sMaster.sh
/home/student/LFD5461/SOLUTIONS/s_02/k8sMaster.sh
```

Open the file
```
student@master:˜$ more LFD5461/SOLUTIONS/s_02/k8sMaster.sh
k8sMaster.sh
```


3. Run the script as an argument to the `bash` shell. You will need the kubeadm join command shown near the end of the output when you add the worker/minion node in a future step. Use the `tee` command to save the output of the script, in case you cannot scroll back to find the kubeadm join in the script output. Please note the following is one command and then its output.

Using `Ubuntu 18` you will be asked questions during the installation. Allow restarts and use the local, installed software
if asked during the update, usually option 2.
Copy files to your home directory first.
```
student@master:~$ cp /home/student/LFD5461/SOLUTIONS/s_02/k8sMaster.sh .
student@master:~$ ls -ltr
total 76
-rw-rw-r-- 1 student student 62192 Mar 26 21:19 LFD5461_V1.20_SOLUTIONS.tar.xz
drwxrwxr-x 3 student student  4096 Apr 26 18:40 LFD5461
-rwxrwxr-x 1 student student  2223 Apr 26 18:45 k8sMaster.sh
```
```
student@master:˜$ bash k8sMaster.sh | tee $HOME/master.out
```
<output_omitted>



Deploy a Minion Node
4. Open a separate terminal into your second node. Having both terminal sessions allows you to monitor the status of the cluster while adding the second node. Change the color or other characteristic of the second terminal to make it visually distinct from the first. This will keep you from running commands on the incorrect instance, which probably won’t work.
Use the previous wget command to download the tarball to the second node. Extract the files with tar as before. Find and copy the `k8sSecond.sh` file to student’s home directory then view it. You should see the same early steps as on
the master system.
```
student@worker:˜$ find $HOME -name k8sSecond.sh
student@worker:˜$ find $HOME -name k8sSecond.sh
/home/student/LFD5461/SOLUTIONS/s_02/k8sSecond.sh
```
```
student@worker:˜$ cp /home/student/LFD5461/SOLUTIONS/s_02/k8sSecond.sh .
```

5. Run the script on the second node. Again please note you may have questions during the update. Allow daemons to restart and use the local installed version, usually option 2.

```
bash k8sSecond.sh | tee $HOME/second.out
```

At the end we can see:
```
Script finished. You now need the kubeadm join command
from the output on the master node
```

6. When the script is done the minion node is ready to join the cluster.
The kubeadm join statement can be found near the end of the kubeadm init output on the master node. It should also be in the file master.out as well. Your nodes
will use a different IP address and hashes than the example below. You’ll need to pre-pend sudo to run the script copied from the master node. Also note that some non-Linux operating systems and tools insert extra characters when
multi-line samples are copied and pasted. Copying one line at a time solves this issue.
On master node:
```
~$ grep -A3 join master.out
```
Then you can join any number of worker nodes by running the following on each as root:
```
kubeadm join 10.128.0.45:6443 --token ywxslj.h3ewhwugabfrswyu \
    --discovery-token-ca-cert-hash sha256:1bbaebaa9c61f21464debf69612d947781173e7b8a4d030fc0ed91432be1b415
Running the steps explained at the end of the init output for you
Apply Calico network plugin from ProjectCalico.org
```

Run the `kubeadm join` command from the second node to connect to master.

```
student@worker:˜$ kubeadm join 10.128.0.45:6443 --token ywxslj.h3ewhwugabfrswyu \
    --discovery-token-ca-cert-hash sha256:1bbaebaa9c61f21464debf69612d947781173e7b8a4d030fc0ed91432be1b415
```

It now shows the node has joined the cluster in the stdout:
```
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```


7. Configure Master Node
Install vi
```
sudo apt-get install bash-completion vim -y
```

8. We will configure command line completion and verify both nodes have been added to the cluster. The first command will configure completion in the current shell. The second command will ensure future shells have completion. You may need to exit the shell and log back in for command completion to work without error.

```
student@master-73kx:~$ source <(kubectl completion bash)
student@master-73kx:~$ echo "source <(kubectl completion bash)" >> $HOME/.bashrc
```

9. Verify that both nodes are part of the cluster. And show a `Ready` state
```
student@master-73kx:~$ kubectl get node
NAME          STATUS   ROLES                  AGE     VERSION
master-73kx   Ready    control-plane,master   17m     v1.20.1
worker-8f1k   Ready    <none>                 2m35s   v1.20.1
```

10. We will use the `kubectl` command for the majority of work with Kubernetes. Review the help output to become familiar with commands options and arguments.
`student@master:˜$ kubectl --help`
```
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create        Create a resource from a file or from stdin.
  expose        Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service
  run           Run a particular image on the cluster
  set           Set specific features on objects

Basic Commands (Intermediate):
  explain       Documentation of resources
  get           Display one or many resources
  edit          Edit a resource on the server
  delete        Delete resources by filenames, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout       Manage the rollout of a resource
  scale         Set a new size for a Deployment, ReplicaSet or Replication Controller
  autoscale     Auto-scale a Deployment, ReplicaSet, or ReplicationController

Cluster Management Commands:
  certificate   Modify certificate resources.
  cluster-info  Display cluster info
  top           Display Resource (CPU/Memory/Storage) usage.
  cordon        Mark node as unschedulable
  uncordon      Mark node as schedulable
  drain         Drain node in preparation for maintenance
  taint         Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe      Show details of a specific resource or group of resources
  logs          Print the logs for a container in a pod
  attach        Attach to a running container
  exec          Execute a command in a container
  port-forward  Forward one or more local ports to a pod
  proxy         Run a proxy to the Kubernetes API server
  cp            Copy files and directories to and from containers.
  auth          Inspect authorization
  debug         Create debugging sessions for troubleshooting workloads and nodes

Advanced Commands:
  diff          Diff live version against would-be applied version
  apply         Apply a configuration to a resource by filename or stdin
  patch         Update field(s) of a resource
  replace       Replace a resource by filename or stdin
  wait          Experimental: Wait for a specific condition on one or many resources.
  kustomize     Build a kustomization target from a directory or a remote url.

Settings Commands:
  label         Update the labels on a resource
  annotate      Update the annotations on a resource
  completion    Output shell completion code for the specified shell (bash or zsh)

Other Commands:
  api-resources Print the supported API resources on the server
  api-versions  Print the supported API versions on the server, in the form of "group/version"
  config        Modify kubeconfig files
  plugin        Provides utilities for interacting with plugins.
  version       Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

11. With more than 40 arguments, you can explore each also using the `--help` option. Take a closer look at a few, starting with `taint` for example.

`student@master:˜$ kubectl taint --help`
```
Update the taints on one or more nodes.

  *  A taint consists of a key, value, and effect. As an argument here, it is expressed as key=value:effect.
  *  The key must begin with a letter or number, and may contain letters, numbers, hyphens, dots, and underscores, up to
253 characters.
  *  Optionally, the key can begin with a DNS subdomain prefix and a single '/', like example.com/my-app
  *  The value is optional. If given, it must begin with a letter or number, and may contain letters, numbers, hyphens,
dots, and underscores, up to  63 characters.
  *  The effect must be NoSchedule, PreferNoSchedule or NoExecute.
  *  Currently taint can only apply to node.
...
```

12. By default the master node will not allow general containers to be deployed for security reasons. This is via a `taint`.
Only containers which tolerate this taint will be scheduled on this node. As we only have two nodes in our cluster we will remove the taint, allowing containers to be deployed on both nodes. This is not typically done in a production
environment for security and resource contention reasons. The following command will remove the taint from all nodes, so you should see one success and one not found error. The worker/minion node does not have the taint to begin with. Note the minus sign at the end of the command, which removes the preceding value.

`student@master:˜$ kubectl describe nodes | grep -i Taint`

We can see current Taints:
```
Taints:             node-role.kubernetes.io/master:NoSchedule
Taints:             <none>
```

Now run:

`student@master:˜$ kubectl taint nodes --all node-role.kubernetes.io/master-`

I get:
```
node/master-73kx untainted
error: taint "node-role.kubernetes.io/master" not found
```

Check that both nodes are without a Taint. If they both are without taint the nodes should now show as Ready. It may take a minute or two for all infrastructure pods to enter Ready state, such that the nodes will show a Ready state.

`student@master:˜$ kubectl describe nodes | grep -i taint`

`student@master:˜$ kubectl get nodes`
```
NAME          STATUS   ROLES                  AGE   VERSION
master-73kx   Ready    control-plane,master   28m   v1.20.1
worker-8f1k   Ready    <none>                 13m   v1.20.1
```


#### Exercise 2.3: Create a basic Pod
1. The smallest unit we directly control with Kubernetes is the pod. We will create a pod by creating a minimal YAML file. First we will get a list of current API objects and their APIGROUP. If value is not shown it may not exist, as with SHORTNAMES. Note that pods does not declare an APIGROUP. At the moment this indicates it is part of the stable v1 group.

`student@master:˜$ kubectl api-resources`

2. From the output we see most are v1 which is used to denote a stable object. With that information we will add the other
three required sections for pods such as metadata, with a name, and spec which declares which container image to use and a name for the container. We will create an eight line YAML file. White space and indentation matters. Don’t use Tabs. There is a basic.yaml file available in the tarball, as well as basic-later.yaml which shows what the file will become and can be helpful for figuring out indentation.
```
$ find $HOME -name basic.yaml
/home/student/LFD5461/SOLUTIONS/s_02/basic.yaml
$ cp /home/student/LFD5461/SOLUTIONS/s_02/basic.yaml .
$ more basic.yaml
apiVersion: v1
kind: Pod
metadata:
  name: basicpod
spec:
  containers:
  - name: webcont
    image: nginx
```

3. Create the new pod using the recently created YAML file.
```
student@master:˜$ kubectl create -f basic.yaml
pod/basicpod created
```

4. Make sure the pod has been created then use the describe sub-command to view the details. Among other values in the output you should be about to find the image and the container name

```
student@master:˜$ kubectl get pod
NAME       READY   STATUS    RESTARTS   AGE
basicpod   1/1     Running   0          43s
```

```
student@master:˜$ kubectl describe pod basicpod
Name:         basicpod
Namespace:    default
Priority:     0
Node:         worker-8f1k/10.128.0.126
Start Time:   Mon, 26 Apr 2021 19:25:52 +0000
Labels:       <none>
Annotations:  cni.projectcalico.org/podIP: 192.168.162.1/32
              cni.projectcalico.org/podIPs: 192.168.162.1/32
Status:       Running
IP:           192.168.162.1
IPs:
  IP:  192.168.162.1
Containers:
  webcont:
    Container ID:   docker://25c7ad9c2b5764ac5c771f6233d78ed035fba05763d69bf424dead44a7d341be
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:75a55d33ecc73c2a242450a9f1cc858499d468f077ea942867e662c247b5e412
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 26 Apr 2021 19:26:01 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8t7gb (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-8t7gb:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-8t7gb
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  73s   default-scheduler  Successfully assigned default/basicpod to worker-8f1k
  Normal  Pulling    72s   kubelet            Pulling image "nginx"
  Normal  Pulled     66s   kubelet            Successfully pulled image "nginx" in 6.087144232s
  Normal  Created    64s   kubelet            Created container webcont
  Normal  Started    64s   kubelet            Started container webcont
```

5. Shut down the pod and verify it is no longer running.
```
student@master:˜$ kubectl delete pod basicpod
pod "basicpod" deleted
```
```
student@master:˜$ kubectl get pod
No resources found in default namespace.
```

6. We will now configure the pod to expose port 80. This configuration does not interact with the container to determine what port to open. We have to know what port the process inside the container is using, in this case port 80 as a web
server. Add two lines to the end of the file. Line up the indentation with the image declaration.

```
student@master:˜$ vim basic.yaml
```

Add:
```
apiVersion: v1
kind: Pod
metadata:
 name: basicpod
spec:
 containers:
 - name: webcont
   image: nginx
   ports: #<--Add this and following line
   - containerPort: 80
```

7. Create the pod and verify it is running. Use the -o wide option to see the internal IP assigned to the pod, as well
as NOMINATED NODE, which is used by the scheduler and READINESS GATES which show if experimental features are
enabled. Using curl and the pods IP address you should get the default nginx welcome web page.
```
student@master:˜$ kubectl create -f basic.yaml
pod/basicpod created
```

```
student@master:˜$ kubectl get pod -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
basicpod   1/1     Running   0          30s   192.168.162.2   worker-8f1k   <none>           <none>
```

```
student@master:˜$ curl http://192.168.162.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
...
```

```
student@master:˜$ kubectl delete pod basicpod
pod "basicpod" deleted
```


8. We will now create a simple service to expose the pod to other nodes and pods in the cluster. The service YAML will have the same four sections as a pod, but different spec configuration and the addition of a selector.

```
student@master:˜$ vim basicservice.yaml
```

here is the yaml file:
```
apiVersion: v1
kind: Service
metadata:
 name: basicservice
spec:
 selector:
  type: webserver
 ports:
 - protocol: TCP
   port: 80
```

We will also add a label to the pod and a selector to the service so it knows which object to communicate with.

basic.yaml
```
apiVersion: v1
kind: Pod
metadata:
 name: basicpod
 labels: #<-- Add this line
  type: webserver #<-- and this line which matches selector
 spec:
 ...
```

10. Create the new pod and service. Verify both have been created.
```
student@master:˜$ kubectl create -f basic.yaml
pod/basicpod created
```

```
student@master:˜$ kubectl create -f basicservice.yaml
service/basicservice created
```

```
student@master:˜$ kubectl get pod
NAME       READY   STATUS    RESTARTS   AGE
basicpod   1/1     Running   0          64s
```

```
student@master:˜$ kubectl get svc
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
basicservice   ClusterIP   10.104.187.78   <none>        80/TCP    66s
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP   48m
```

11. Test access to the web server using the CLUSTER-IP for the basicservice.

```
student@master:˜$ curl http://10.104.187.78
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
...
```

12. We will now expose the service to outside the cluster as well. Delete the service, edit the file and add a type declaration.
```
student@master:˜$ kubectl delete svc basicservice
service "basicservice" deleted
```

```
student@master:˜$ vim basicservice.yaml
```

Add this line:
```
apiVersion: v1
kind: Service
metadata:
 name: basicservice
spec:
 selector:
  type: webserver
 type: NodePort  #<--Add this line
 ports:
 - protocol: TCP
   port: 80
```


13. Create the service again. Note there is a different TYPE and CLUSTER-IP and also a high-numbered port.
```
student@master:˜$ kubectl create -f basicservice.yaml
service/basicservice created
```

```
student@master:˜$ kubectl get svc
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
basicservice   NodePort    10.101.81.153   <none>        80:31394/TCP   29s
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP        52m
```

14. Using the public IP address of the node and the high port you should be able to test access to the webserver. The high port will also probably be different. Note that testing from within a GCE or AWS node will not work. Use a local to you terminal or web browser to test.
The `35.238.3.83` is the public IP from my node
```
local$ curl http://35.238.3.83:31394
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
```

#### Exercise 2.4: Multi-Container Pods
Using a single container per pod allows for the most granularity and decoupling. There are still some reasons to deploy multiple containers, sometimes called composite containers, in a single pod. The secondary containers can handle logging or enhance the primary, the sidecar concept, or acting as a proxy to the outside, the ambassador concept, or modifying data to meet an external format such as an adapter. All three concepts are secondary containers to perform a function the primary container does not.

1. We will add a second container to the pod to handle logging. Without going into details of how to use fluentd we will
add a logging container to the exiting pod from its own repository. The second container would act as a sidecar. At
this state we will just add the second container and verify it is running. In the Deployment Configuration chapter we
will continue to work on this pod by adding persistent storage and configure fluentd via a configMap.
Edit the YAML file and add a fluentd container. The dash should line up with the previous container dash. At this point
a name and image should be enough to start the second container.
```
student@master:˜$ vim basic.yaml
```

Add:
```
....
containers:
 - name: webcont
  image: nginx
  ports:
  - containerPort: 80
 - name: fdlogger
  image: fluent/fluentd
```

2. Delete and create the pod again. The commands can be typed on a single line, separated by a semicolon. This time
you should see 2/2 under the READY column. You should also find information on the fluentd container inside of the
kubectl describe output.
```
student@master:˜$ kubectl delete pod basicpod ; kubectl create -f basic.yaml
pod "basicpod" deleted
pod/basicpod created
```

```
student@master:˜$ kubectl get pod
NAME       READY   STATUS    RESTARTS   AGE
basicpod   2/2     Running   0          37s
```

```
student@master:˜$ kubectl describe pod basicpod
Name:         basicpod
Namespace:    default
Priority:     0
Node:         worker-8f1k/10.128.0.126
Start Time:   Mon, 26 Apr 2021 19:53:02 +0000
Labels:       type=webserver
...
```

3. For now shut down the pod. We will use it again in a future exercise.
```
student@master:˜$ kubectl delete pod basicpod
pod "basicpod" deleted
```


#### Exercise 2.5: Create a Simple Deployment
Creating a pod does not take advantage of orchestration abilities of Kubernetes. We will now create a Deployment
which gives us scalability, reliability, and updates.

1. Now run a containerized webserver nginx. Use kubectl create to create a simple, single replica deployment running
the nginx web server. It will create a single pod as we did previously but with new controllers to ensure it runs as well as
other features
```
student@master:˜$ kubectl create deployment firstpod --image=nginx
deployment.apps/firstpod created
```

2. Verify the new deployment exists and the desired number of pods matches the current number. Using a comma, you
can request two resource types at once. The Tab key can be helpful. Type enough of the word to be unique and press
the Tab key, it should complete the word. The deployment should show a number 1 for each value, such that the desired
number of pods matches the up-to-date and running number. The pod should show zero restarts.

```
student@master:˜$ kubectl get deployment,pod
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/firstpod   1/1     1            1           53s

NAME                            READY   STATUS    RESTARTS   AGE
pod/firstpod-7c4477dd55-czpxf   1/1     Running   0          53s
```


3. View the details of the deployment, then the pod. Work through the output slowly. Knowing what a healthy deployment
and looks like can be helpful when troubleshooting issues. Again the Tab key can be helpful when using long autogenerated
object names. You should be able to type firstpodTab and the name will complete when viewing the pod.
```
student@master:˜$ kubectl describe deployment firstpod
Name:                   firstpod
Namespace:              default
CreationTimestamp:      Mon, 26 Apr 2021 19:55:43 +0000
Labels:                 app=firstpod
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=firstpod
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
...
NewReplicaSet:   firstpod-7c4477dd55 (1/1 replicas created)

```

```
student@master:˜$ kubectl describe pod firstpod-7c4477dd55
Name:         firstpod-7c4477dd55-czpxf
Namespace:    default
Priority:     0
Node:         worker-8f1k/10.128.0.126
Start Time:   Mon, 26 Apr 2021 19:55:43 +0000
Labels:       app=firstpod
              pod-template-hash=7c4477dd55
Annotations:  cni.projectcalico.org/podIP: 192.168.162.5/32
              cni.projectcalico.org/podIPs: 192.168.162.5/32
Status:       Running
IP:           192.168.162.5
IPs:
  IP:           192.168.162.5
Controlled By:  ReplicaSet/firstpod-7c4477dd55
Containers:
  nginx:
  ...
```

4. Note that the resources are in the default namespace. Get a list of available namespaces.
```
student@master:˜$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   67m
kube-node-lease   Active   67m
kube-public       Active   67m
kube-system       Active   67m
```

5. There are four default namespaces. Look at the pods in the kube-system namespace.
```
student@master:˜$ kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-69496d8b75-wlnxm   1/1     Running   0          67m
calico-node-nczzd                          1/1     Running   0          52m
calico-node-nfs62                          1/1     Running   0          67m
coredns-74ff55c5b-h4v9m                    1/1     Running   0          67m
coredns-74ff55c5b-mw5nv                    1/1     Running   0          67m
etcd-master-73kx                           1/1     Running   0          67m
kube-apiserver-master-73kx                 1/1     Running   0          67m
kube-controller-manager-master-73kx        1/1     Running   0          67m
kube-proxy-bqh7h                           1/1     Running   0          52m
kube-proxy-dq8v9                           1/1     Running   0          67m
kube-scheduler-master-73kx                 1/1     Running   0          67m
```


6. Now look at the pods in a namespace that does not exist. Note you do not receive an error.
```
student@master:˜$ kubectl get pod -n fakenamespace
No resources found in fakenamespace namespace.
```

7. You can also view resources in all namespaces at once. Use the --all-namespaces options to select objects in all
namespaces at once.
```
student@master:˜$ kubectl get pod --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
default       firstpod-7c4477dd55-czpxf                  1/1     Running   0          6m
kube-system   calico-kube-controllers-69496d8b75-wlnxm   1/1     Running   0          68m
kube-system   calico-node-nczzd                          1/1     Running   0          53m
kube-system   calico-node-nfs62                          1/1     Running   0          68m
kube-system   coredns-74ff55c5b-h4v9m                    1/1     Running   0          68m
kube-system   coredns-74ff55c5b-mw5nv                    1/1     Running   0          68m
kube-system   etcd-master-73kx                           1/1     Running   0          68m
kube-system   kube-apiserver-master-73kx                 1/1     Running   0          68m
kube-system   kube-controller-manager-master-73kx        1/1     Running   0          68m
kube-system   kube-proxy-bqh7h                           1/1     Running   0          53m
kube-system   kube-proxy-dq8v9                           1/1     Running   0          68m
kube-system   kube-scheduler-master-73kx                 1/1     Running   0          68m
```


8. View several resources at once. Note that most resources have a short name such as rs for ReplicaSet, po for Pod,
svc for Service, and ep for endpoint. Note the endpoint still exists after we deleted the pod.

```
student@master:˜$ kubectl get deploy,rs,po,svc,ep
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/firstpod   1/1     1            1           6m34s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/firstpod-7c4477dd55   1         1         1       6m34s

NAME                            READY   STATUS    RESTARTS   AGE
pod/firstpod-7c4477dd55-czpxf   1/1     Running   0          6m34s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/basicservice   NodePort    10.101.81.153   <none>        80:31394/TCP   17m
service/kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP        69m

NAME                     ENDPOINTS          AGE
endpoints/basicservice   <none>             17m
endpoints/kubernetes     10.128.0.45:6443   69m
```

9. Delete the ReplicaSet and view the resources again. Note that the age on the ReplicaSet and the pod it controls
is now less than a minute of age. The deployment operator started a new ReplicaSet operator when we deleted the
existing one. The new ReplicaSet started another pod when the desired spec did not match the current status.
```
student@master:˜$ kubectl delete rs firstpod-7c4477dd55
replicaset.apps "firstpod-7c4477dd55" deleted
```

```
student@master:˜$ kubectl get deployment,rs,po,svc,ep
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/firstpod   1/1     1            1           8m4s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/firstpod-7c4477dd55   1         1         1       23s

NAME                            READY   STATUS    RESTARTS   AGE
pod/firstpod-7c4477dd55-wbb27   1/1     Running   0          23s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/basicservice   NodePort    10.101.81.153   <none>        80:31394/TCP   18m
service/kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP        70m

NAME                     ENDPOINTS          AGE
endpoints/basicservice   <none>             18m
endpoints/kubernetes     10.128.0.45:6443   70m
```

10. This time delete the top-level controller. After about 30 seconds for everything to shut down you should only see the
cluster service and endpoint remain for the cluster and the service we created.
```
student@master:˜$ kubectl delete deployment firstpod
deployment.apps "firstpod" deleted
```

```
student@master:˜$ kubectl get deployment,rs,po,svc,ep
NAME                            READY   STATUS        RESTARTS   AGE
pod/firstpod-7c4477dd55-wbb27   0/1     Terminating   0          72s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/basicservice   NodePort    10.101.81.153   <none>        80:31394/TCP   19m
service/kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP        71m

NAME                     ENDPOINTS          AGE
endpoints/basicservice   <none>             19m
endpoints/kubernetes     10.128.0.45:6443   71m
```

11. As we won’t need it for a while, delete the basicservice service as well.
```
student@master:˜$ kubectl delete svc basicservice
service "basicservice" deleted
```



#### Exercise 2.6: Domain Review
1. Using a browser go to https://www.cncf.io/certification/ckad/ and read through the program description.

2. In the Exam Resources section open the Curriculum Overview and Candidate-handbook in new tabs. Both of these should be read and understood prior to sitting for the exam.

3. Navigate to the Curriculum Overview tab. You should see links for domain information for various versions of the exam.
Select the latest version, such as CKAD Curriculum V1.20.1.pdf. The versions you see may be different. You should see a new page showing a PDF.

4. Read through the document. Be aware that the term Understand, such as Understand Services, is more than just knowing they exist. In this case expect it to also mean create, update, and troubleshoot.

5. Locate the Core Concepts section. If you review the lab, you will see we have covered these steps. Again, please note
this document will change, distinct from this book. It remains your responsibility to check for changes in the online
document. They may change on an irregular and unannounced basis.

6. Navigate to the Candidate-handbook tab. You are strongly encourage to read and understand this entire document prior
to taking the exam. Again, please note this document will change, distinct from this book. It remains your responsibility
to check for changes in the online document. They may change on an irregular and unannounced basis.

7. Find the Guidelines and Tips for Use of the Linux server terminal” section in the document.

8. Among other points you will note the current exam version and three (at the time this was written) domains and subdomains
you can use, with some stated conditions.

9. Using only the allowed browser, URLs, and subdomains search for and bookmark a YAML example to create and
configure a basic pod. Ensure it works for the version of the exam you are taking. URLs may change, plan on checking
each book mark prior to taking the exam.

10. Using a timer and bookmarked YAML files see how long it takes you to create and verify. Try it again and see how much
faster you can complete and test each step:
• A new pod with the nginx image. Showing all containers running and a Ready status.
• A new service exposing the pod as a nodePort, which presents a working webserver configured in the previous
step.
• Update the pod to run the nginx:1.11-alpine image and re-verify you can view the webserver via a nodePort.

11. Find and use the architecture-review1.yaml file included in the course tarball. Your path, such as course number,
may be different than the one in the example below. Use the find output. Determine if the pod is running. Fix any errors
you may encounter. The use of kubectl describe may be helpful.
```
student@master:˜$ find $HOME -name architecture-review1.yaml
1 /home/student/LFD259/SOLUTIONS/s_02/architecture-review1.yaml
```
```
student@master:˜$ cp <copy-paste-from-above> .
student@master:˜$ kubectl create -f architecture-review1.yaml
```
12. Remove any pods or services you may have created as part of the review before moving on to the next section. For
example:
```
student@master:˜$ kubectl delete -f architecture-review1.yaml
```
