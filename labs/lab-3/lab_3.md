# K8s Lab 3
Related to Chapter #4 - APIs and Access (p.79)

Using the Kubernetes API, `kubectl` makes API calls for you. With the appropriate TLS keys you could run curl as well
use a `golang` client. Calls to the kube-apiserver get or set a PodSpec, or desired state. If the request represents
a new state the Kubernetes Control Plane will update the cluster until the current state matches the specified state.
Some end states may require multiple requests. For example, to delete a ReplicaSet, you would first set the number
of replicas to zero, then delete the ReplicaSet.
An API request must pass information as JSON. `kubectl` converts .yaml to JSON when making an API request on
your behalf. The API request has many settings, but must include apiVersion, kind and metadata, and spec settings
to declare what kind of container to deploy. The spec fields depend on the object being created.
We will begin by configuring remote access to the kube-apiserver then explore more of the API.

#### Exercise 4.1: Configuring TLS Access

1. Begin by reviewing the kubectl configuration file. We will use the three certificates and the API server address.
```
student@master:˜$ less $HOME/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdw...
```

2. We will create a variables using certificate information. You may want to double-check each parameter as you set it.
Begin with setting the client-certificate-data key.
```
student@master:˜$ export client=$(grep client-cert $HOME/.kube/config |cut -d" " -f 6)
student@master:˜$ echo $client
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURFekNDQWZ1Z0F3SUJBZ0lJUDBxQ3NGaERLTFF3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TVRBME1qWXhPRFV5TXpWYUZ3MHlNakEwTWpZeE9EVXlNemhhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXN3RGRNbVZWVzY5QmppYjIKVnFnV2thUk5OenJaNENhNnJmVnQ1WkN2MEdlbllMVEdOOGNibUp3c1FHNlZvV0JlaENoTzFWZ3pPVnl4c0FvYgpGRWdQMVdiYVFTc2t2K05GbVNnMWVDWmZRQnlIN0wzMkNRNDhRa1AydFlIL0l6cFVGRFRzKzBwYjQ4TVZjYmVsCk1ZUE5mb3oxYkNlYmxxVlpRa0hTb3NTODIwZ2FkNzZ1Ky9qNStrZE9LM2s1bkNaMnZQbGFLTlpuNjlvNG9vUEgKbVBMTHVBeVV0U2lweU50UGJpWGh3NmlUR3VSeWI1dW44QmFUNlNHOHFzdzRGa0x4aThOb3V6Zm1KbXdvRDJWaApjUU85NWU4bE55dXlDRkZsbmxvVXg2dmhMZ1JzL0ptRzh3NTFia01DOGNhSHVuKzZVTGxHdS9lYVJaM3hYaWxXCkE3eWNid0lEQVFBQm8wZ3dSakFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0h3WURWUjBqQkJnd0ZvQVVadkdTVUg2dURRZVI1bUQxbGVORzBsZEEwMHd3RFFZSktvWklodmNOQVFFTApCUUFEZ2dFQkFDQjRPbVVvVHAxZXhpRWRzdk54alRoQ0xQYmJWMUdqN3RYQldzNDFoQ3FXbXVJWWxkc1hzY0FXClhUb3FSVHpBMGNvZXZNSmxiRVcwOGwxQkJKc1k4UzliNzd0Q3J1UFZUMkxHZytyWXVCQVhoOXZQT1orNnNEN3oKMjh5STE0UHZpT1hoUTNKRXlaMGF4c2lsYWpkazRaaVlUUmJrNWZiNFZXUXoySkVydHA1MGRwVDNpdVMrWVp6NgptZ1RMdTh1Nm12NnZCdDBTb3Bvdnl0RUhwbk8vcGFPOEJjbFNVYWNKS1lYaVdWdkN0a3B1VW9lQnhieU1CUjZ0CkxWcVk4Q0k4dXYyZTVWZ1MyYlA1Rk1ISUNaQm1iQ3N4cVFIM2xwd3F0bHpKTnRsYmJ3aXpIcTJOTy9CMlVabTUKNkJXa3ZjYzlIV29zbmw5TWxQMTU3OThJU0lRYjc3cz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
```

3. Almost the same command, but this time collect the client-key-data as the key variable.
```
student@master:˜$ export key=$(grep client-key-data $HOME/.kube/config |cut -d " " -f 6)
student@master:˜$ echo $key
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBc3dEZE1tVlZXNjlCamliMlZxZ1drYVJOTnpyWjRDYTZyZlZ0NVpDdjBHZW5ZTFRHCk44Y2JtSndzUUc2Vm9XQmVoQ2hPMVZnek9WeXhzQW9iRkVnUDFXYmFRU3NrditORm1TZzFlQ1pmUUJ5SDdMMzIKQ1E0OFFrUDJ0WUgvSXpwVUZEVHMrMHBiNDhNVmNiZWxNWVBOZm96MWJDZWJscVZaUWtIU29zUzgyMGdhZDc2dQorL2o1K2tkT0szazVuQ1oydlBsYUtOWm42OW80b29QSG1QTEx1QXlVdFNpcHlOdFBiaVhodzZpVEd1UnliNXVuCjhCYVQ2U0c4cXN3NEZrTHhpOE5vdXpmbUptd29EMlZoY1FPOTVlOGxOeXV5Q0ZGbG5sb1V4NnZoTGdScy9KbUcKOHc1MWJrTUM4Y2FIdW4rNlVMbEd1L2VhUlozeFhpbFdBN3ljYndJREFRQUJBb0lCQUNvMFhmTHY2WHhBaWhoVwpIMmEzSXZzYjNnalRtMk02UG5HZG1GZTBFWC8xc0lVZ21rdTBhbEZGSVRuZjJPKy9wUWxMNTZwdHJVWXRFTWdNCjJlMmlQL2wwMHFqaTN6dE8vcTNweVJHWEdvWW5NL1VocE16bHlsZGxadG5NMkxjWm5aRldHVG4yZ2t2bFJ6MFUKcTZKTkRUcDFTYmhDYm5ES2M0MG5yOTRvdk02R2dLRTVvL1BUT0x4cDM1NDRyTk02TUtjZXlvUXloeU1QWHpzYwpjSC92MzdneTN3R2djK0dhcGY2cUI5andFSGVRNHBLcTJCU01IaWlLNmJIRkNleHlWbGlGaCt4cEc3QlQwREN5CnRGbkV4M0kvQW5RYndvT2Z1WElFTEJPM2NiSG5NQ2ptQ1dvcDkwK21DZ0IzK2tpYktBbzUxNVJtd0dPVFNJVTgKNlRiUUZjRUNnWUVBMG1IRkw0eEo2b1JBTGZRQVhkNmJIcDlkNFRnRDlqdmsreFlBL2VqeERPSHhVbjVzQ0xlZwpjK1kxaUc1Q3BiVmdsK3ZFQVJMbXVUK0VubjRzQnhhZ2J5L1dXamtnbjNvMFVibzVKUTcxZDdoeC9QTXBKbHZ5CkdpcmtrUGFaVFE3SHUwU1ppT0swR2dkYXFzcEhtWG5qT2t4elRGL25MU1dNeFNBVzFxQzY3UWNDZ1lFQTJkRkkKRXY2a0x0QUxrTDI2MEFNazA2L2Z0QUQ1Zk8xSVBRczVYZzhwK2hTT1VPcmQ4QmZRb2tDUTRRRThyeXpsOXEvagpadjEwL3JQTk1WQ2hscERDM20wNWRWT2VaZjZaZ3FBa1pnOFJQSFBDcWFPSXc3OXJ2RmlHMThoRVFlemJnak93CmJqQmZHcUV6M1ZDZDFVSlBlK0tUdjZ4NkhRaHFDZmp0NEVjVjQxa0NnWUVBaXlWaldDbC9rZjdhdkFvUVhBV3AKcnoxVXlHdFdJM3hGM05RVzM5azc1WHRqTTE2dWNhMlNFRllJdmkyL0l2NnQzbzF2WEE1MlQ3djFLU2JtVStIaApSdWpxNjAyeGlBazVwWHgzNDB6YmljMlNodTBGSUh2Ynl2Ym5xZ0dRTDJsQkxWa1grM09HRDFraDNLaGhadDg1CkxRdjNqbUkzOHFKTlV1d0Fid0JyNGUwQ2dZRUFpR1BJbXNSQ2NHR3JiRVd4allENmRXY2lUNmR3a2E1TzFwS0oKcUlFY1N0REdVSnJRdi85Wmo4K1lLSnhLc0hJUHlVbFh2bXlrN3J3TmwzeWl3aElCUTUvbkk3VnBjUHBjaTNQVgpkdWFtWjFFaEtuSVJPR2xMZjlON0UvRDk5TDNvc1gzR1V5a00xREx1cy8wLzU0S3ZIS2JDMTNtYnVVUkVpZHdLCnI2NHpZWGtDZ1lFQWl4eFlKYjlyVVRyWFRqaTdRakZ3S1FMTzRFOUJ4aTVSSUpOVENiR1QzWllJd0ZDdVJSaVQKaGNPSFUwVDRJNE1McDk3a1ArQmozNkc5c0tnM2NiUmhGcUlnTkFjR2hpVFBDalN0YVBkWlJ3RXVwdzl5c2FqWgpyditwZURPZkxyQ3NpNGw5L2ppSkJFZHdUMmNjMVVtMHh3a3JYREdWdDQ3WDBDUThvcDlaUnZrPQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```

4. Finally set the auth variable with the certificate-authority-data key.
```
student@master:˜$ export auth=$(grep certificate-authority-data $HOME/.kube/config |cut -d " " -f 6)
student@master:˜$ echo $auth
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1EUXlOakU0TlRJek5Wb1hEVE14TURReU5ERTROVEl6TlZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTWMrCmE3TzNPZkJ5emNJMlV0QkJJc1JLRnhnUjU0RjhPMWc0WkFKMjVaNUtkaGtxM0JZeFIyMnlCTUpjUjczRHpvYWIKUTlrbUIwL3dhSWU1MmpZTGYyeWYwWVI5bXQ5QlpTNTZrVWYzaCtYbDJqUlRnc0JlYVZoTmFMRHZpcmxaNzJXbgpFbUFJWG1SWTRPNDBMdmVSSTRjS1I2dnZOQ2tvTVZkcjBERUpSTlRtcERZVzFsUFNjNFo3UU9MMVRJbkFiMjJVCkcydXpVdzl6OTRlWDloMFErb2xUd09YWlBzV3ozeUlCeW9VdTVIVHE5WHBNcVNSQmFENktxUGRIWXhkbTJ1YlEKZlZYVTd3dHZmY0lqVU5wVnRLN1RGMTRtZG5sN3JaSEFsdjMxcE9PdituVmhTSGdVQnBOTUh0Vm9uUjJQc3VtdwpjdUZDdlcyZG5DVWgvTHJ1YkJrQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZHYnhrbEIrcmcwSGtlWmc5WlhqUnRKWFFOTk1NQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFDUDJlSVByMnJhV3RIcTZHcnBnSit3TkNqVjZZR0FCYUJjN0NaWU44YVM2Mkp2UnVpdApGdWZ4NTJNQ1RXK2VTeWhrSTBJZCtOWkR6Ny9WWlpZRFJ6dVAyUVNpOVdGUjNTTVFPbk43WnFoc0k0VFlYNHoxClpwL3BNL2N5Tk04dUxyQmRuNWZCa2ErWmFpOFVsckcyc0pnak5DdllidnZHNkFJb05PZG5SRGYxUnl4UTRrT24KUm1WOENETnZiem4wMU1yVXdCNi9tUFJHc2ltaWtkTjc2ZTlHTGVHYnhBT2xJQlFCZFducWZ6aHQ4WG5jek8xeApmU1BZbGRSTElKMW5wL2tlMXZvM0dlb3dVYjJKcVdHSHh5aDlqY3oxQVFFN0NLRGE0ay8xbHdUQXpuWDlFZHJ6CnNJczFuZkd4c0wzQlZNazJPZFdPcUdHQTdZait1R0d6TUYwTAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
```


5. Now encode the keys for use with curl.
```
student@master:˜$ echo $client | base64 -d - > ./client.pem
student@master:˜$ echo $key | base64 -d - > ./client-key.pem
student@master:˜$ echo $auth | base64 -d - > ./ca.pem
```

6. Pull the API server URL from the config file. Your hostname or IP address may be different.
```
student@master:˜$ kubectl config view |grep server
    server: https://10.128.0.45:6443
```

7. Use curl command and the encoded keys to connect to the API server. Use your hostname, or IP, found in the previous
command, which may be different than the example below.
`
student@master:˜$ curl --cert ./client.pem
 --key ./client-key.pem
 --cacert ./ca.pem
 https://10.128.0.45:6443/api/v1/pods
`

8. If the previous command was successful, create a JSON file to create a new pod. Remember to use find and search
for this file in the tarball output, it can save you some typing.
```
student@master:˜$ vim curlpod.json
{
    "kind": "Pod",
    "apiVersion": "v1",
    "metadata":{
        "name": "curlpod",
        "namespace": "default",
        "labels": {
            "name": "examplepod"
        }
    },
    "spec": {
        "containers": [{
            "name": "nginx",
            "image": "nginx",
            "ports": [{"containerPort": 80}]
        }]
    }
}
```


9. The previous curl command can be used to build a XPOST API call. There will be a lot of output, including the scheduler
and taints involved. Read through the output. In the last few lines the phase will probably show Pending, as it’s near the
beginning of the creation process.
`
student@master:˜$ curl --cert ./client.pem
--key ./client-key.pem --cacert ./ca.pem
https://10.128.0.45:6443/api/v1/namespaces/default/pods
-XPOST -H'Content-Type: application/json'
-d@curlpod.json
`


10. Verify the new pod exists and shows a Running status.

```
student@master:˜$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
curlpod                    1/1     Running   0          41s
nginx-79f868896b-svbqp     1/1     Running   1          165m
registry-6b5bb79c4-ddsqm   1/1     Running   1          165m
try1-6bc6f69476-4j9d6      2/2     Running   0          123m
try1-6bc6f69476-fttlq      2/2     Running   0          123m
try1-6bc6f69476-k6hmx      2/2     Running   0          123m
try1-6bc6f69476-n7mk4      2/2     Running   0          123m
try1-6bc6f69476-q49gt      2/2     Running   0          123m
try1-6bc6f69476-xhpxf      2/2     Running   0          123m
```

#### Exercise 4.2: Explore API Calls
1. One way to view what a command does on your behalf is to use strace. In this case, we will look for the current
endpoints, or targets of our API calls. Install the tool, if not present.
```
student@master:˜$ sudo apt-get install -y strace
student@master:˜$ kubectl get endpoints
NAME         ENDPOINTS             AGE
kubernetes   10.128.0.45:6443      23h
nginx        192.168.162.9:443     170m
registry     192.168.162.10:5000   170m
```

2. Run this command again, preceded by strace. You will get a lot of output. Near the end you will note several `openat`
functions to a local directory, /home/student/.kube/cache/discovery/k8smaster 6443. If you cannot find the lines,
you may want to redirect all output to a file and grep for them. This information is cached, so you may see some
differences should you run the command multiple times. As well your IP address may be different.
```
student@master:˜$ strace kubectl get endpoints &> endpoints.out
student@master:˜$ grep openat endpoints.out
...
openat(AT_FDCWD, "/home/student/.kube/cache/discovery/10.128.0.45_6443/apps/v1/serverresources.json", O_RDONLY|O_CLOEXEC) = 6
openat(AT_FDCWD, "/home/student/.kube/cache/discovery/10.128.0.45_6443/events.k8s.io/v1/serverresources.json", O_RDONLY|O_CLOEXEC) = 6
openat(AT_FDCWD, "/home/student/.kube/cache/discovery/10.128.0.45_6443/events.k8s.io/v1beta1/serverresources.json", O_RDONLY|O_CLOEXEC) = 6
openat(AT_FDCWD, "/home/student/.kube/cache/discovery/10.128.0.45_6443/autoscaling/v2beta2/serverresources.json", O_RDONLY|O_CLOEXEC) = 7
openat(AT_FDCWD, "/home/student/.kube/cache/discovery/10.128.0.45_6443/certificates.k8s.io/v1/serverresources.json", O_RDONLY|O_CLOEXEC) = 6
openat(AT_FDCWD, "/home/student/.kube/cache/discovery/10.128.0.45_6443/certificates.k8s.io/v1beta1/serverresources.json", O_RDONLY|O_CLOEXEC) = 6
openat(AT_FDCWD, "/home/student/.kube/cache/discovery/10.128.0.45_6443/batch/v1/serverresources.json", O_RDONLY|O_CLOEXEC) = 6
...
```

3. Change to the parent directory and explore. Your endpoint IP will be different, so replace the following with one suited to your system.

```
student@master:˜$ cd /home/student/.kube/cache/discovery/
student@master:˜/.kube/cache/discovery$ ls
10.128.0.45_6443
```

```
student@master:˜/.kube/cache/discovery$ cd 10.128.0.45_6443/
```

4. View the contents. You will find there are directories with various configuration information for kubernetes.

```
student@master:˜/.kube/cache/discovery/10.128.0.45_6443$ ls
admissionregistration.k8s.io  authorization.k8s.io  crd.projectcalico.org         networking.k8s.io          servergroups.json
apiextensions.k8s.io          autoscaling           discovery.k8s.io              node.k8s.io                storage.k8s.io
apiregistration.k8s.io        batch                 events.k8s.io                 policy                     v1
apps                          certificates.k8s.io   extensions                    rbac.authorization.k8s.io
authentication.k8s.io         coordination.k8s.io   flowcontrol.apiserver.k8s.io  scheduling.k8s.io
```

5. Use the find command to list out the subfiles. The prompt has been modified to look better on this page.
```
student@master:./k8smaster_6443$ find .
...
./scheduling.k8s.io
./scheduling.k8s.io/v1
./scheduling.k8s.io/v1/serverresources.json
./scheduling.k8s.io/v1beta1
./scheduling.k8s.io/v1beta1/serverresources.json
./discovery.k8s.io
./discovery.k8s.io/v1beta1
./discovery.k8s.io/v1beta1/serverresources.json
...
```

6. View the objects available in version 1 of the API. For each object, or kind:, you can view the verbs or actions for that
object, such as create seen in the following example. Note the prompt has been truncated for the command to fit on one
line. Some are HTTP verbs, such as GET, others are product specific options, not standard HTTP verbs. The command
may be python, depending on what version is installed.
```
student@master:.$ python3 -m json.tool v1/serverresources.json
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
...
```


7. Some of the objects have shortNames, which makes using them on the command line much easier. Locate the shortName for endpoints.
```
student@master:.$ python3 -m json.tool v1/serverresources.json | less
...
"name": "endpoints",
"singularName": "",
"namespaced": true,
"kind": "Endpoints",
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
        "ep"
    ],
    "storageVersionHash": "fWeeMqaN/OA="
},
...
```

8. Use the shortName to view the endpoints. It should match the output from the previous command.
```
student@master:.$ kubectl get ep
NAME         ENDPOINTS             AGE
kubernetes   10.128.0.45:6443      23h
nginx        192.168.162.9:443     3h4m
registry     192.168.162.10:5000   3h4m
```

9. We can see there are 37 objects in version 1 file.
```
student@master:.$ python3 -m json.tool v1/serverresources.json | grep kind
38
```

10. Looking at another file we find nine more.
```
student@master:$ python3 -m json.tool \
apps/v1/serverresources.json | grep kind
"kind": "APIResourceList",
        "kind": "ControllerRevision",
        "kind": "DaemonSet",
        "kind": "DaemonSet",
        "kind": "Deployment",
        "kind": "Scale",
        "kind": "Deployment",
        "kind": "ReplicaSet",
        "kind": "Scale",
        "kind": "ReplicaSet",
        "kind": "StatefulSet",
        "kind": "Scale",
        "kind": "StatefulSet",
```

11. Delete the curlpod to recoup system resources.a
```
student@master:$ kubectl delete po curlpod
pod "curlpod" deleted
```
