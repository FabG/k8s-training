# K8s Lab 8

This lab is related to Chapter #9 - Scheduling (p.180)


#### Exercise 9.1: Assign Pods Using Labels
While allowing the system to distribute Pods on your behalf is typically the best route, you may want to determine which
nodes a Pod will use.

For example you may have particular hardware requirements to meet for the workload. You may
want to assign VIP Pods to new, faster hardware and everyone else to older hardware.
In this exercise we will use `labels` to schedule Pods to a particular node. Then we will explore `taints` to have more flexible deployment in a large environment.

1. Begin by getting a list of the nodes. They should be in the ready state and without added labels or taints.
```
student@master:˜$ kubectl get nodes
NAME          STATUS   ROLES                  AGE   VERSION
master-73kx   Ready    control-plane,master   2d    v1.20.1
worker-8f1k   Ready    <none>                 47h   v1.20.1
```

2. View the current labels and taints for the nodes.
```
student@master:˜$ kubectl describe nodes |grep -A5 -i label
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=master-73kx
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
--
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=worker-8f1k
                    kubernetes.io/os=linux
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
```

```
student@master:˜$ kubectl describe nodes |grep -i taint
Taints:             <none>
Taints:             <none>
```

3. Get a count of how many containers are running on both the master and worker nodes.

```
student@master:˜$ kubectl get deployments --all-namespaces
NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
default       nginx                     1/1     1            1           27h
default       registry                  1/1     1            1           27h
default       try1                      0/6     6            0           3h56m
kube-system   calico-kube-controllers   1/1     1            1           2d
kube-system   coredns                   2/2     2            2           2d
```

```
student@master:˜$ sudo docker ps |wc -l
22
```

```
student@worker:˜$ sudo docker ps |wc -l
26
```

4. For the purpose of the exercise we will assign the master node to be VIP hardware and the secondary node to be for
others.
```
student@master:˜$ kubectl label nodes master<tab> status=vip
kubectl label nodes master-73kx status=vip
node/master-73kx labeled
```

```
student@master:˜$ kubectl label nodes worker<tab> status=other
kubectl label nodes worker-8f1k status=other
node/worker-8f1k labeled
```


5. Verify your settings. You will also find there are some built in labels such as hostname, os and architecture type. The
output below appears on multiple lines for readability.
```
student@master:˜$ kubectl get nodes --show-labels
student@master-73kx:~$ kubectl get nodes --show-labels
NAME          STATUS   ROLES                  AGE   VERSION   LABELS
master-73kx   Ready    control-plane,master   2d    v1.20.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master-73kx,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,status=vip
worker-8f1k   Ready    <none>                 2d    v1.20.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker-8f1k,kubernetes.io/os=linux,status=other
```

6. Create vip.yaml to spawn four busybox containers which sleep the whole time. Include the nodeSelector entry.
```
student@master:˜$ vim vip.yaml
apiVersion: v1
kind: Pod
metadata:
  name: vip
spec:
  containers:
  - name: vip1
    image: busybox
    args:
    - sleep
    - "1000000"
  - name: vip2
    image: busybox
    args:
    - sleep
    - "1000000"
  - name: vip3
    image: busybox
    args:
    - sleep
    - "1000000"
    - name: vip4
      image: busybox
      args:
      - sleep
      - "1000000"
    nodeSelector:
      status: vip    
```


7. Deploy the new pod. Verify the containers have been created on the master node. It may take a few seconds for all the
containers to spawn. Check both the master and the secondary nodes.
```
student@master:˜$ kubectl create -f vip.yaml
pod/vip created
student@master:˜$ sudo docker ps |wc -l
27
```

```
student@worker:˜$ sudo docker ps |wc -l
26
```

8. Delete the pod then edit the file, commenting out the nodeSelector lines. It may take a while for the containers to fully
terminate.
```
student@master:˜$ kubectl delete pod vip
pod "vip" deleted
```

```
student@master:˜$ vim vip.yaml
# nodeSelector:
 # status: vip
```

9. Create the pod again. Containers should now be spawning on either node. You may see pods for the daemonsets as well.
```
student@master:˜$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
design2-ddc7b59cb-j7p6t    1/1     Running   0          22h
nginx-79f868896b-svbqp     1/1     Running   1          27h
registry-6b5bb79c4-ddsqm   1/1     Running   1          27h
try1-6dc98655fc-22vmg      1/2     Running   0          4h6m
try1-6dc98655fc-gsqck      1/2     Running   0          4h6m
try1-6dc98655fc-h7rpq      1/2     Running   0          4h6m
try1-6dc98655fc-m9s44      1/2     Running   0          4h6m
try1-6dc98655fc-slrqw      1/2     Running   0          4h6m
try1-6dc98655fc-spb5h      1/2     Running   0          4h6m
```

```
student@master:˜$ kubectl create -f vip.yaml
pod/vip created
```

10. Determine where the new containers have been deployed. They should be more evenly spread this time.
```
student@master:˜$ sudo docker ps |wc -l
27
```

```
student@worker:˜$ sudo docker ps |wc -l
26
```

11. Create another file for other users. Change the names from vip to others, and uncomment the nodeSelector lines.
```
student@master:˜$ cp vip.yaml other.yaml
student@master:˜$ sed -i s/vip/other/g other.yaml
student@master:˜$ vim other.yaml
```

12. Create the other containers. Determine where they deploy.
```
student@master:˜$ kubectl create -f other.yaml
pod/other created
```

```
student@master:˜$ sudo docker ps |wc -l
27
```

```
student@worker:˜$ sudo docker ps |wc -l
31
```

13. Shut down both pods and verify they terminated. Only our previous pods should be found.
```
student@master:˜$ kubectl delete pods vip other
pod "vip" deleted
pod "other" deleted
student@master:˜$ kubectl get pods
```


#### Exercise 9.2: Using Taints to Control Pod Deployment
Use taints to manage where Pods are deployed or allowed to run. In addition to assigning a Pod to a group of nodes,
you may also want to limit usage on a node or fully evacuate Pods. Using taints is one way to achieve this. You may
remember that the master node begins with a NoSchedule taint. We will work with three taints to limit or remove
running pods.
1. Verify that the master and secondary node have the minimal number of containers running.
```
student@master:˜$ kubectl delete deployment try1
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
design2-ddc7b59cb-j7p6t    1/1     Running   0          23h
nginx-79f868896b-svbqp     1/1     Running   1          28h
registry-6b5bb79c4-ddsqm   1/1     Running   1          28h
```

2. Create a deployment which will deploy eight nginx containers. Begin by creating a YAML file.
```
student@master:˜$ vim taint.yaml
```
3. Apply the file to create the deployment.
```
student@master:˜$ kubectl apply -f taint.yaml
deployment.apps/taint-deployment created
```

4. Determine where the containers are running.
```
student@master:˜$ sudo docker ps |grep nginx
...
student@master:˜$ sudo docker ps |wc -l
19
student@worker:˜$ sudo docker ps |wc -l
27
```

5. Delete the deployment. Verify the containers are gone.
```
student@master:˜$ kubectl delete deployment taint-deployment
deployment.apps "taint-deployment" deleted
student@master:˜$ sudo docker ps |wc -l
19
student@worker:˜$ sudo docker ps |wc -l
11
```
