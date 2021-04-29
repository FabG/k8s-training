# K8s Lab 9

This lab is related to Chapter #10 - Security (p.194)

#### Exercise 10.1: Set SecurityContext for a Pod and Container
In this lab we will implement security features for new applications, as the simpleapp YAML file is getting long and more
difficult to read. Kubernetes architecture favors smaller, decoupled, and transient applications working together. We’ll
continue to emulate that in our exercises.
In this exercise we will create two new applications. One will be limited in its access to the host node, but have access
to encoded data. The second will use a network security policy to move from the default all-access Kubernetes
policies to a mostly closed network. First we will set security contexts for pods and containers, then create and
consume secrets, then finish with configuring a network security policy.

1. Begin by making a new directory for our second application. Change into that directory.
```
student@master:˜$ mkdir $HOME/app2
student@master:˜$ cd $HOME/app2/
```

2. Create a YAML file for the second application. In the example below we are using a simple image, busybox, which allows
access to a shell, but not much more. We will add a `runAsUser` to both the pod as well as the container.
```
student@master:˜/app2$ vim second.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secondapp
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: busy
    image: busybox
    command:
      - sleep
      - "3600"
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
```


3. Create the secondapp pod and verify it’s running. Unlike the previous deployment this application is running as a pod.
Look at the YAML output, to compare and contrast with what a deployment looks like. The status section probably has
the largest contrast.
```
student@master:˜/app2$ kubectl create -f second.yaml
pod/secondapp created
student@master:˜/app2$ kubectl get pod secondapp
NAME        READY   STATUS    RESTARTS   AGE
secondapp   1/1     Running   0          33s
```

```
student@master:˜/app2$ kubectl get pod secondapp -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/podIP: 192.168.162.1/32
    cni.projectcalico.org/podIPs: 192.168.162.1/32
  creationTimestamp: "2021-04-29T17:30:04Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
...
```

4. Execute a Bourne shell within the Pod. Check the user ID of the shell and other processes. It should show the container
setting, not the pod. This allows for multiple containers within a pod to customize their UID if desired. As there is only
one container in the pod we do not need to use the -c busy option.
```
student@master:˜/app2$ kubectl exec -it secondapp -- sh
```

On the container where we now hav access:
```
/ $ ps aux
PID   USER     TIME  COMMAND
    1 2000      0:00 sleep 3600
    8 2000      0:00 sh
   15 2000      0:00 ps aux
```

5. While here check the capabilities of the kernel. In upcoming steps we will modify these values.
```
/ $ grep Cap /proc/1/status
CapInh:	00000000a80425fb
CapPrm:	0000000000000000
CapEff:	0000000000000000
CapBnd:	00000000a80425fb
CapAmb:	0000000000000000
/ $ exit
```

6. Use the capability shell wrapper tool, the capsh command, to decode the output. We will view and compare the output
in a few steps. Note that there are 14 comma separated capabilities listed.
```
student@master:˜/app2$ capsh --decode=00000000a80425fb
0x00000000a80425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
```

7. Edit the YAML file to include new capabilities for the container. A capability allows granting of specific, elevated privileges
without granting full root access. We will be setting `NET ADMIN` to allow interface, routing, and other network
configuration. We’ll also set `SYS TIME`, which allows system clock configuration. More on kernel capabilities can be
read here: https://github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h
It can take up to a minute for the pod to fully terminate, allowing the future pod to be created.
```
student@master:˜/app2$ kubectl delete pod secondapp
```

```
student@master:˜/app2$ vim second.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secondapp
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: busy
    image: busybox
    command:
      - sleep
      - "3600"
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities: #<-- Add this and following line
       add: ["NET_ADMIN", "SYS_TIME"]
```

8. Create the pod again. Execute a shell within the container and review the Cap settings under `/proc/1/status`. They should be different from the previous instance.
```
student@master:˜/app2$ kubectl create -f second.yaml
pod/secondapp created
student@master:˜/app2$ kubectl exec -it secondapp -- sh
```

on the container:
```
/ $ grep Cap /proc/1/status
CapInh:	00000000aa0435fb
CapPrm:	0000000000000000
CapEff:	0000000000000000
CapBnd:	00000000aa0435fb
CapAmb:	0000000000000000
/ $ exit
```

9. Decode the output again. Note that the instance now has 16 comma delimited capabilities listed. cap net admin is
listed as well as cap sys time.
```
student@master:˜/app2$ capsh --decode=00000000aa0435fb
0x00000000aa0435fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_admin,cap_net_raw,cap_sys_chroot,cap_sys_time,cap_mknod,cap_audit_write,cap_setfcap
```


#### Exercise 10.2: Create and consume Secrets
Secrets are consumed in a manner similar to ConfigMaps, covered in an earlier lab. While at-rest encryption is just now
enabled, historically a secret was just base64 encoded. There are three types of encryption which can be configured.

1. Begin by generating an encoded password.
```
student@master:˜/app2$ echo LFTr@1n | base64
TEZUckAxbgo=
```

2. Create a YAML file for the object with an API object kind set to Secret. Use the encoded key as a password parameter.
```
student@master:˜/app2$ vim secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: lfsecret
data:
  password: TEZUckAxbgo=
```

3. Ingest the new object into the cluster.
```
student@master:˜/app2$ kubectl create -f secret.yaml
secret/lfsecret created
```

4. Edit secondapp YAML file to use the secret as a volume mounted under `/mysqlpassword`.
`volumeMounts:` lines up with the container `name:` and `volumes:` lines up with `containers:` Note the pod will restart when the sleep command
finishes every 3600 seconds, or every hour.
```
student@master:˜/app2$ vim second.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secondapp
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: busy
    image: busybox
    command:
      - sleep
      - "3600"
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities: #<-- Add this and following line
       add: ["NET_ADMIN", "SYS_TIME"]
  volumeMounts: #<-- Add this and six following lines
  - name: mysql
   mountPath: /mysqlpassword
  volumes:
  - name: mysql
   secret:
    secretName: lfsecret
```

```
student@master:˜/app2$ kubectl delete pod secondapp
pod "secondapp" deleted
student@master:˜/app2$ kubectl create -f second.yaml
pod/secondapp created
```

5. Verify the pod is running, then check if the password is mounted where expected. We will find that the password is
available in its clear-text, decoded state.
```
student@master:˜/app2$ kubectl get pod secondapp
NAME READY STATUS RESTARTS AGE
secondapp 1/1 Running 0 34s
student@master:˜/app2$ kubectl exec -ti secondapp -- /bin/sh
```

On container:
```
/ $ cat /mysqlpassword/password
LFTr@1n
```


6. View the location of the directory. Note it is a symbolic link to ..data which is also a symbolic link to another directory.
After taking a look at the filesystem within the container, exit back to the node.
On Container:
```
/ $ cd /mysqlpassword/
/mysqlpassword $ ls
password
/mysqlpassword $ ls -al
total 4
drwxrwxrwt 3 root root 100 Apr 11 07:24 .
drwxr-xr-x 21 root root 4096 Apr 11 22:30 ..
drwxr-xr-x 2 root root 60 Apr 11 07:24 ..4984_11_04_07_24_47.831222818
lrwxrwxrwx 1 root root 31 Apr 11 07:24 ..data -> ..4984_11_04_07_24_47.831222818
6lrwxrwxrwx 1 root root 15 Apr 11 07:24 password -> ..data/password
/mysqlpassword $ exit
```

See the reaminder of the exercises in the PDF 
