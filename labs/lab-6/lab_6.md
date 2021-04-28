# K8s Lab 6

This lab covers the K8s course about Deployment and Configuration (starting page 135)
In this lab we will add resources to our deployment with further configuration you may need for production.
There are three different ways a `ConfigMap` can ingest data, from a literal value, from a file, or from a directory of files.

#### Exercise 7.1: Configure the Deployment: Secrets and ConfigMap
Notes:
Save a copy of your $HOME/app1/simpleapp.yaml file, in case you would like to repeat portions of the labs, or you
find your file difficult to use due to typos and whitespace issues.
```
student@master:˜$ cp $HOME/app1/simpleapp.yaml $HOME/beforeLab5.yaml
```


1. Create a ConfigMap containing primary colors. We will create a series of files to ingest into the ConfigMap. First create
a directory primary and populate it with four files. Then we create a file in our home directory with our favorite color.
```
student@master:˜/app1$ cd
student@master:˜$ mkdir primary
student@master:˜$ echo c > primary/cyan
student@master:˜$ echo m > primary/magenta
student@master:˜$ echo y > primary/yellow
student@master:˜$ echo k > primary/black
student@master:˜$ echo "known as key" >> primary/black
student@master:˜$ echo blue > favorite
```

```
$ ls -l primary/
total 16
-rw-rw-r-- 1 student student 15 Apr 28 15:00 black
-rw-rw-r-- 1 student student  2 Apr 28 15:00 cyan
-rw-rw-r-- 1 student student  2 Apr 28 15:00 magenta
-rw-rw-r-- 1 student student  2 Apr 28 15:00 yellow
$ more primary/black
k
known as key
```


2. Generate a configMap using each of the three methods.
```
student@master:˜$ kubectl create configmap colors \
--from-literal=text=black \
--from-file=./favorite \
--from-file=./primary/
configmap/colors created
```

3. View the newly created configMap. Note the way the ingested data is presented.
```
student@master:˜$ kubectl get configmap colors
NAME     DATA   AGE
colors   6      25s
```

```
student@master:˜$ kubectl get configmap colors -o yaml
apiVersion: v1
data:
  black: |
    k
    known as key
  cyan: |
    c
  favorite: |
    blue
  magenta: |
    m
  text: black
  yellow: |
    y
kind: ConfigMap
metadata:
  creationTimestamp: "2021-04-28T15:02:09Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:black: {}
        f:cyan: {}
        f:favorite: {}
        f:magenta: {}
        f:text: {}
        f:yellow: {}
    manager: kubectl-create
    operation: Update
    time: "2021-04-28T15:02:09Z"
  name: colors
  namespace: default
  resourceVersion: "209451"
  uid: c4fb9ecd-3c3b-433e-9225-454fde01352a
```


4. Update the YAML file of the application to make use of the configMap as an environmental parameter. Add the six lines
from the env: line to key:favorite.
```
student@master:˜$ vim $HOME/app1/simpleapp.yaml
 ....
 spec:
 containers:
 - image: 10.105.119.236:5000/simpleapp
 env: #<-- Add from here
 - name: ilike
 valueFrom:
 configMapKeyRef:
 name: colors
 key: favorite #<-- to here
 imagePullPolicy: Always
 ....

```

5. Delete and re-create the deployment with the new parameters.
```
student@master-lab-7xtx:˜$ kubectl delete deployment try1
deployment.apps "try1" deleted
student@master-lab-7xtx:˜$ kubectl create -f $HOME/app1/simpleapp.yaml
deployment.apps/try1 created
```

6. Even though the try1 pod is not in a fully ready state, it is running and useful. Use kubectl exec to view a variable’s value. View the pod state then verify you can see the ilike value within the simpleapp container. Note that the use of double dash (- -) tells the shell to pass the following as standard in.
```
student@master:˜$ kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
design2-ddc7b59cb-j7p6t    1/1     Running   0          18h
nginx-79f868896b-svbqp     1/1     Running   1          23h
registry-6b5bb79c4-ddsqm   1/1     Running   1          23h
try1-6dc98655fc-22vmg      1/2     Running   0          75s
try1-6dc98655fc-gsqck      1/2     Running   0          75s
try1-6dc98655fc-h7rpq      1/2     Running   0          75s
try1-6dc98655fc-m9s44      1/2     Running   0          75s
try1-6dc98655fc-slrqw      1/2     Running   0          75s
try1-6dc98655fc-spb5h      1/2     Running   0          75s
```

```
student@master:˜$ kubectl exec -c simpleapp -it try1-6dc98655fc-spb5h \
-- /bin/bash -c 'echo $ilike'
blue
```

...
