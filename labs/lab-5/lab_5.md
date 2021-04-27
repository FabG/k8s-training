# K8s Lab 5
This lab is related to the API Objects K8s course (p.119).
We will skip the first 3 exercises as they are duplicate with previous lab.


### Exercise 6.4: Using Labels
Create and work with labels. We will understand how the deployment, replicaSet, and pod labels interact.
1. Create a new deployment called design2
```
student@master:˜$ kubectl create deployment design2 --image=nginx
deployment.apps/design2 created
```

2. View the wide kubectl get output for the design2 deployment and make note of the `SELECTOR`
```
student@master:˜$ kubectl get deployments.apps design2 -o wide
NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
design2   1/1     1            1           26s   nginx        nginx    app=design2
```

3. Use the `-l` option to use the selector to list the pods running inside the deployment. There should be only one pod
running.
```
student@master:˜$ kubectl get -l app=design2 pod
NAME                      READY   STATUS    RESTARTS   AGE
design2-ddc7b59cb-j7p6t   1/1     Running   0          59s
```

4. View the pod details in YAML format using the deployment selector. This time use the –selector option. Find the pod
label in the output. It should match that of the deployment.
```
student@master:˜$ kubectl get --selector app=design2 pod -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      cni.projectcalico.org/podIP: 192.168.162.48/32
      cni.projectcalico.org/podIPs: 192.168.162.48/32
    creationTimestamp: "2021-04-27T20:28:50Z"
    generateName: design2-ddc7b59cb-
    labels:
      app: design2
      pod-template-hash: ddc7b59cb
      ...
```

5. Edit the pod label to be your favorite color.
```
student@master:˜$ kubectl edit pod design2-ddc7b59cb-j7p6t
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/podIP: 192.168.162.48/32
    cni.projectcalico.org/podIPs: 192.168.162.48/32
  creationTimestamp: "2021-04-27T20:28:50Z"
  generateName: design2-ddc7b59cb-
  labels:
    app: green                       #<<-- Edit this line
    pod-template-hash: ddc7b59cb
  name: design2-ddc7b59cb-j7p6t
  namespace: default
  ownerReferences:
  ...
```

6. Now view how many pods are in the deployment. Then how many have design2 in their name. Note the AGE of the
pods.
```
student@master:˜$ kubectl get deployments.apps design2 -o wide
NAME      READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES   SELECTOR
design2   0/1     1            0           4m31s   nginx        nginx    app=design2
```

```
student@master:˜$ kubectl get pods | grep design2
design2-ddc7b59cb-j7p6t    1/1     Running   0          5m3s
design2-ddc7b59cb-s46t4    1/1     Running   0          34s
```

7. Delete the design2 deployment.
```
student@master:˜$ kubectl delete deploy design2
deployment.apps "design2" deleted
```

8. Check again for pods with design2 in their names. You should find one pod, with an AGE of when you first created the
deployment. Once the label was edited the deployment created a new pod in order that the status matches the spec
and there be a replica running with the intended label.
```
student@master:˜$ kubectl get pods | grep design2
design2-ddc7b59cb-j7p6t    1/1     Running   0          5m45s
```

9. Delete the pod using the -l and the label you edited to be your favorite color in a previous step. The command details
have been omitted. Use previous steps to figure out these commands.


#### Exercise 6.5: Setting Pod Resource Limits and Requirements
1. Create a new pod running the vish/stress image. A YAML stress.yaml file has been included in the course tarball.
```
sudo find / -name stress.yaml
cp /home/student/LFD5461/SOLUTIONS/s_06/stress.yaml .
kubectl create -f stress.yaml
deployment.apps/stressmeout created
```

2. Run the top command on the master and worker nodes. You should find a stress command consuming the majority of
the CPU on one node, the worker. Use ctrl-c to exit from top. Delete the deployment.
```
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
9065 root      20   0 1169936 1.112g   3120 R 172.2 15.2   0:28.47 stress
11453 root      20   0 1168440 435564  71024 S   4.0  5.7  20:28.23 kube-apiserver
20763 root      20   0 1949056 110332  67452 S   4.0  1.4  52:26.18 kubelet
10383 root      20   0 1458120 124992  53340 S   2.0  1.6   8:16.74 dockerd
11027 root      20   0 10.121g  75048  23628 S   1.7  1.0   4:32.19 etcd
12224 root      20   0 1507016  56956  39392 S   1.3  0.7   5:41.37 calico-node
11997 root      20   0  816712 110380  61120 S   1.0  1.4   5:34.01 kube-controller
  1 root      20   0  225820   9824   6932 S   0.7  0.1   2:56.63 systemd
```

3. Edit the stress.yaml file add in the following limits and requests.
student@master:˜$
```
student@master:~$ vim stress.yaml
...
containers:
- image: vish/stress
  imagePullPolicy: Always
  name: stressmeout
  resources:          #<<-- Add this and following six lines
    limits:
      cpu: "1"
      memory: "1Gi"
    requests:
      cpu: "0.5"
      memory: "500Mi"
...
```

4. Create the deployment again. Check the status of the pod. You should see that it shows an `OOMKilled` status and a
growing number of restarts. You may see a status of Running if you catch the pod in early in a restart. If you wait long
enough you may see `CrashLoopBackOff`.
```
$ kubectl delete deploy stressmeout
deployment.apps "stressmeout" deleted
$ kubectl get pods
stressmeout-57b9dc9477-26fqs   0/1     Terminating   0          6m29s
```

```
$ kubectl create -f stress.yaml
deployment.apps/stressmeout created
```

```
$ kubectl get pod stressmeout-7dc667854-bmcbc
NAME                          READY   STATUS    RESTARTS   AGE
stressmeout-7dc667854-bmcbc   1/1     Running   1          20s
```

After a while:
```
$ kubectl get pod stressmeout-7dc667854-bmcbc
NAME                          READY   STATUS             RESTARTS   AGE
stressmeout-7dc667854-bmcbc   0/1     CrashLoopBackOff   4          3m1s
```

```
$ kubectl delete deploy stressmeout
deployment.apps "stressmeout" deleted
```
