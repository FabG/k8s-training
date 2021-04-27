# K8s Lab 4
This lab is related to the API Objects K8s course (p.95)

#### Exercise 5.1: RESTful API Access
We will continue to explore ways of accessing the control plane of our cluster. In the security chapter we will discuss
there are several authentication methods, one of which is use of a Bearer token We will work with one then deploy a
local proxy server for application-level access to the Kubernetes API.

We will use the curl command to make API requests to the cluster, in an insecure manner. Once we know the IP address
and port, then the token we can retrieve cluster data in a RESTful manner. By default most of the information is restricted, but
changes to authentication policy could allow more access.


1. First we need to know the IP and port of a node running a replica of the API server. The master system will typically have one running. Use kubectl config view to get overall cluster configuration, and find the server entry. This will give us both the IP and the port.
```
student@master:˜$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.128.0.45:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

2. Next we need to find the bearer token. This is part of a default token. Look at a list of tokens, first all on the cluster, then
just those in the default namespace. There will be a secret for each of the controllers of the cluster.
```
student@master:˜$ kubectl get secrets --all-namespaces
...
kube-system       replicaset-controller-token-ddhgf                kubernetes.io/service-account-token   3      23h
kube-system       replication-controller-token-5jkhl               kubernetes.io/service-account-token   3      23h
kube-system       resourcequota-controller-token-v6tp5             kubernetes.io/service-account-token   3      23h
kube-system       root-ca-cert-publisher-token-m4c8k               kubernetes.io/service-account-token   3      23h
kube-system       service-account-controller-token-pxzxt           kubernetes.io/service-account-token   3      23h
kube-system       service-controller-token-gjrgk                   kubernetes.io/service-account-token   3      23h
kube-system       statefulset-controller-token-q5xkg               kubernetes.io/service-account-token   3      23h
kube-system       token-cleaner-token-9b4g8                        kubernetes.io/service-account-token   3      23h
kube-system       ttl-controller-token-xdfcl                       kubernetes.io/service-account-token   3      23h
```

```
student@master:˜$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-8t7gb   kubernetes.io/service-account-token   3      23h
```


3. Look at the details of the secret. We will need the token: information from the output.
```
student@master:˜$ kubectl describe secret default-token-8t7gb
Name:         default-token-8t7gb
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 49f0198f-8b6f-44f5-9e61-1327dda09add

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImdoVzBQWnc3ZG8zOUJDSnk5ZVBfM1pDYVBiZ3p0cTVJaTVpLVpFVG9UOUkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tOHQ3Z2IiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjQ5ZjAxOThmLThiNmYtNDRmNS05ZTYxLTEzMjdkZGEwOWFkZCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.Pcp2WQJl_8RikpDFiXOAAtNwFlS0tWr8nqgNB71Ba_PgT5Nsi8C0AqAT2ZtdTolB5xUFr1h_5C5brhcto7aeQHSD1_Osr8g0r3pBv96Qw5qwAIBqJfN-n6oS07me6jkQRmxaNE1pBXJjmVfZX8_qcB-2vezFwLaNLjYCv5fSJv5Jx-ba9B5hwgpVFfrQfDYissj7Gn9YNerJBnVXtMO5u_v4Ilx38gX7XceqC_ypF7h-sPojDTOBKWIo1rg4X6cJYuEt6OJ5DCk_Xy-7-XpbO-OSpKVZb955E-Y9-dv6sm3QAECi9QG8sdEOTLQ765DFRTJFuoMPNqhCZpzvAGcP3Q
```

4. Using your mouse to cut and paste, or cut, or awk to save the data, from the first character eyJh to the last, to a variable named `token`. Your token data will be different.

```
student@master:˜$ export token=$(kubectl describe \
secret default-token-8t7gb |grep token |cut -f7 -d ' ')
```

```
student@master:˜$ echo $token
eyJhbGciOiJSUzI1NiIsImtpZCI6ImdoVzBQWnc3ZG8zOUJDSnk5ZVBfM1pDYVBiZ3p0cTVJaTVpLVpFVG9UOUkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tOHQ3Z2IiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjQ5ZjAxOThmLThiNmYtNDRmNS05ZTYxLTEzMjdkZGEwOWFkZCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.Pcp2WQJl_8RikpDFiXOAAtNwFlS0tWr8nqgNB71Ba_PgT5Nsi8C0AqAT2ZtdTolB5xUFr1h_5C5brhcto7aeQHSD1_Osr8g0r3pBv96Qw5qwAIBqJfN-n6oS07me6jkQRmxaNE1pBXJjmVfZX8_qcB-2vezFwLaNLjYCv5fSJv5Jx-ba9B5hwgpVFfrQfDYissj7Gn9YNerJBnVXtMO5u_v4Ilx38gX7XceqC_ypF7h-sPojDTOBKWIo1rg4X6cJYuEt6OJ5DCk_Xy-7-XpbO-OSpKVZb955E-Y9-dv6sm3QAECi9QG8sdEOTLQ765DFRTJFuoMPNqhCZpzvAGcP3Q
```

5. Test to see if you can get basic API information from your cluster. We will pass it the server name and port, the token
and use the `-k` option to avoid using a cert.
```
student@master:˜$ curl https://10.128.0.45:6443/apis --header "Authorization: Bearer $token" -k
```

6. Try the same command, but look at API v1. Note that the path has changed to api.
```
student@master:˜$ curl https://10.128.0.45:6443/api/v1
--header "Authorization: Bearer $token" -k
```

7. Now try to get a list of namespaces. This should return an error. It shows our request is being seen as
systemserviceaccount:, which does not have the RBAC authorization to list all namespaces in the cluster.
```
student@master:˜$ curl \
https://10.128.0.45:6443/api/v1/namespaces \
--header "Authorization: Bearer $token" -k
```

8. Pods can also make use of included certificates to use the API. The certificates are automatically made available to
a pod under the /var/run/secrets/kubernetes.io/serviceaccount/. We will deploy a simple Pod and view the
resources. If you view the token file you will find it is the same value we put into the $token variable. The -i will request
a -t terminal session of the busybox container. Once you exit the container will not restart and the pod will show as
completed.
```
student@master:˜$ kubectl run -i -t busybox --image=busybox --restart=Never
```


9. Clean up by deleting the busybox container.
```
student@master:˜$ kubectl delete pod busybox
```



#### Exercise 5.2: Using the Proxy
Another way to interact with the API is via a proxy. The proxy can be run from a node or from within a Pod through the use of
a sidecar. In the following steps we will deploy a proxy listening to the loopback address. We will use curl to access the API
server. If the curl request works, but does not from outside the cluster, we have narrowed down the issue to authentication
and authorization instead of issues further along the API ingestion process.

1. Begin by starting the proxy. It will start in the foreground by default. There are several options you could pass. Begin by
reviewing the help output.
```
student@master:˜$ kubectl proxy -h
```

2. Start the proxy while setting the API prefix, and put it in the background. You may need to use enter to view the prompt.
Take note of the process ID, 225000 in the example below, we’ll use it to kill the process when we are done.
```
student@master:˜$ kubectl proxy --api-prefix=/ &
[1] 4450
student@master:~$ Starting to serve on 127.0.0.1:8001
```

3. Now use the same curl command, but point toward the IP and port shown by the proxy. The output should be the same
as without the proxy, but may be formatted differently.
```
student@master:˜$ curl http://127.0.0.1:8001/api/
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "10.128.0.45:6443"
    }
  ]
}
```


4. Make an API call to retrieve the namespaces. The command did not work in the previous section due to permissions,
but should work now as the proxy is making the request on your behalf.
```
student@master:˜$ curl http://127.0.0.1:8001/api/v1/namespaces
{
  "kind": "NamespaceList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "117598"
  },
  "items": [
    {
      "metadata": {
        "name": "default",
        "uid": "f23a1939-31d3-4a17-9d06-36d3e6571dca",
        "resourceVersion": "199",
        "creationTimestamp": "2021-04-26T18:52:53Z",
        "managedFields": [
          {
            "manager": "kube-apiserver",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2021-04-26T18:52:53Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {"f:status":{"f:phase":{}}}
          }
        ]
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
...
```

5. Stop the proxy service as we won’t need it any more. Use the process ID from a previous step. Your process ID may be
different.
```
student@master:˜$ kill 4450
[1]+  Terminated              kubectl proxy --api-prefix=/
```


#### Exercise 5.3: Working with Jobs
While most API objects are deployed such that they continue to be available there are some which we may want to run a
particular number of times called a Job, and others on a regular basis called a CronJob

1. Create a job which will run a container which sleeps for three seconds then stops.
```
student@master:˜$ vim job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleepy
spec:
  template:
    spec:
      containers:
      - name: resting
        image: busybox
        command: ["/bin/sleep"]
        args: ["3"]
      restartPolicy: Never
```

2. Create the job, then verify and view the details. The example shows checking the job three seconds in and then again
after it has completed. You may see different output depending on how fast you type.
```
student@master:˜$ kubectl create -f job.yaml
job.batch/sleepy created
```

```
student@master:˜$ kubectl get job
NAME     COMPLETIONS   DURATION   AGE
sleepy   1/1           6s         24s
```

```
student@master:˜$ kubectl describe jobs.batch sleepy
Name:           sleepy
Namespace:      default
Selector:       controller-uid=657b16e1-5630-42ab-b698-6e3dbef1e469
Labels:         controller-uid=657b16e1-5630-42ab-b698-6e3dbef1e469
                job-name=sleepy
Annotations:    <none>
Parallelism:    1
Completions:    1
Start Time:     Tue, 27 Apr 2021 19:37:13 +0000
Completed At:   Tue, 27 Apr 2021 19:37:19 +0000
Duration:       6s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=657b16e1-5630-42ab-b698-6e3dbef1e469
           job-name=sleepy
  Containers:
   resting:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      /bin/sleep
    Args:
      3
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  48s   job-controller  Created pod: sleepy-lfw8c
  Normal  Completed         42s   job-controller  Job completed
```

```
student@master:˜$ kubectl get job
NAME     COMPLETIONS   DURATION   AGE
sleepy   1/1           6s         88s
```


3. View the configuration information of the job. There are three parameters we can use to affect how the job runs. Use `-o yaml` to see these parameters. We can see that backoffLimit, completions, and the parallelism. We’ll add these
parameters next.
```
student@master:˜$ kubectl get jobs.batch sleepy -o yaml
...
name: sleepy
namespace: default
resourceVersion: "117851"
uid: 657b16e1-5630-42ab-b698-6e3dbef1e469
spec:
backoffLimit: 6
completions: 1
parallelism: 1
...
```

4. As the job continues to AGE in a completion state, delete the job.
```
student@master:˜$ kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted
```

5. Edit the YAML and add the completions: parameter and set it to 5.
```
student@master:˜$ vim job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleepy
spec:
  completions: 5 #<--Add this line
  template:
    spec:
      containers:
      - name: resting
        image: busybox
        command: ["/bin/sleep"]
        args: ["3"]
      restartPolicy: Never
```

6. Create the job again. As you view the job note that COMPLETIONS begins as zero of 5.
```
student@master:˜$ kubectl create -f job.yaml
job.batch/sleepy created
```

```
student@master:˜$ kubectl get jobs.batch
NAME     COMPLETIONS   DURATION   AGE
sleepy   4/5           25s        25s
```

7. View the pods that running. Again the output may be different depending on the speed of typing.
```
student@master:˜$ kubectl get pods
NAME                       READY   STATUS      RESTARTS   AGE
nginx-79f868896b-svbqp     1/1     Running     1          4h27m
registry-6b5bb79c4-ddsqm   1/1     Running     1          4h27m
sleepy-4z22r               0/1     Completed   0          59s
sleepy-9jfx8               0/1     Completed   0          64s
sleepy-qhnm9               0/1     Completed   0          70s
sleepy-r997t               0/1     Completed   0          80s
sleepy-t999v               0/1     Completed   0          75s
try1-6bc6f69476-4j9d6      2/2     Running     0          3h44m
try1-6bc6f69476-fttlq      2/2     Running     0          3h44m
try1-6bc6f69476-k6hmx      2/2     Running     0          3h44m
try1-6bc6f69476-n7mk4      2/2     Running     0          3h44m
try1-6bc6f69476-q49gt      2/2     Running     0          3h44m
try1-6bc6f69476-xhpxf      2/2     Running     0          3h44m
```

8. Eventually all the jobs will have completed. Verify then delete the job.
```
student@master:˜$ kubectl get jobs
NAME     COMPLETIONS   DURATION   AGE
sleepy   5/5           26s        109s
```

```
student@master:˜$ kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted
```

9. Edit the YAML again. This time add in the parallelism: parameter. Set it to 2 such that two pods at a time will be
deployed.
```
student@master:˜$ vim job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleepy
spec:
  completions: 5
  parallelism: 2 #<-- Add this line
  template:
    spec:
      containers:
      - name: resting
        image: busybox
        command: ["/bin/sleep"]
        args: ["3"]
      restartPolicy: Never
```

10. Create the job again. You should see the pods deployed two at a time until all five have completed.
```
student@master:˜$ kubectl create -f job.yaml
job.batch/sleepy created
```

```
student@master:˜$ kubectl get pods
NAME                       READY   STATUS      RESTARTS   AGE
nginx-79f868896b-svbqp     1/1     Running     1          4h30m
registry-6b5bb79c4-ddsqm   1/1     Running     1          4h30m
sleepy-4wqnx               0/1     Completed   0          10s
sleepy-dbdqt               1/1     Running     0          5s
sleepy-m7k4c               0/1     Completed   0          10s
sleepy-wtqkd               1/1     Running     0          3s
try1-6bc6f69476-4j9d6      2/2     Running     0          3h47m
try1-6bc6f69476-fttlq      2/2     Running     0          3h47m
try1-6bc6f69476-k6hmx      2/2     Running     0          3h47m
try1-6bc6f69476-n7mk4      2/2     Running     0          3h47m
try1-6bc6f69476-q49gt      2/2     Running     0          3h47m
try1-6bc6f69476-xhpxf      2/2     Running     0          3h47m
```

```
student@master:˜$ kubectl get jobs
NAME     COMPLETIONS   DURATION   AGE
sleepy   5/5           16s        26s
```


11. Add a parameter which will stop the job after a certain number of seconds. Set the activeDeadlineSeconds: to 15.
The job and all pods will end once it runs for 15 seconds. We will also increase the sleep argument to five, just to be
sure does not expire by itself.
```
student@master:˜$ vim job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleepy
spec:
  completions: 5
  parallelism: 2
  activeDeadlineSeconds: 15 #<-- Add this line
  template:
    spec:
      containers:
      - name: resting
        image: busybox
        command: ["/bin/sleep"]
        args: ["5"]   #<-- Edit this line
      restartPolicy: Never
```


12. Delete and recreate the job again. It should run for 15 seconds, usually 3/5, then continue to age without further
completions.
```
student@master:˜$ kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted
```

```
student@master:˜$ kubectl create -f job.yaml
job.batch/sleepy created
```

```
student@master:˜$ kubectl get jobs
NAME     COMPLETIONS   DURATION   AGE
sleepy   2/5           11s        11s
```

```
student@master:˜$ kubectl get jobs
NAME     COMPLETIONS   DURATION   AGE
sleepy   3/5           23s        23s
```


13. View the message: entry in the `Status` section of the object YAML output.
```
student@master:˜$ kubectl get job sleepy -o yaml
...
status:
  conditions:
  - lastProbeTime: "2021-04-27T19:47:53Z"
    lastTransitionTime: "2021-04-27T19:47:53Z"
    message: Job was active longer than specified deadline
    reason: DeadlineExceeded
    status: "True"
    type: Failed
  failed: 1
  startTime: "2021-04-27T19:47:38Z"
  succeeded: 3
```

14. Delete the job.
```
student@master:˜$ kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted
```



#### Create a CronJob
A CronJob creates a watch loop which will create a batch job on your behalf when the time becomes true. We Will use our
existing Job file to start.

1. Copy the Job file to a new file.
```
student@master:˜$ cp job.yaml cronjob.yaml
```

2. Edit the file to look like the annotated file shown below. Edit the lines mentioned below. The three parameters we added
will need to be removed. Other lines will need to be further indented.
```
student@master:˜$ vim cronjob.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: sleepy
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: resting
            image: busybox
            command: ["/bin/sleep"]
            args: ["5"]
          restartPolicy: Never
```


3. Create the new CronJob. View the jobs. It will take two minutes for the CronJob to run and generate a new batch Job.
```
student@master:˜$ kubectl create -f cronjob.yaml
cronjob.batch/sleepy created
```

```
student@master:˜$ kubectl get cronjobs.batch
NAME SCHEDULE SUSPEND ACTIVE LAST SCHEDULE AGE
sleepy */2 * * * * False 0 <none> 8s
```

```
student@master:˜$ kubectl get jobs.batch
AME                COMPLETIONS   DURATION   AGE
sleepy-1619553120   1/1           7s         11s
```

4. After two minutes you should see jobs start to run.
```
student@master:˜$ kubectl get cronjobs.batch
NAME     SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
sleepy   */2 * * * *   False     1        8s              2m18s
```

```
student@master:˜$ kubectl get jobs.batch
NAME                COMPLETIONS   DURATION   AGE
sleepy-1619553120   1/1           7s         2m17s
sleepy-1619553240   1/1           7s         17s
```

```
student@master:˜$ kubectl get jobs.batch
NAME                COMPLETIONS   DURATION   AGE
sleepy-1619553120   1/1           7s         4m33s
sleepy-1619553240   1/1           7s         2m33s
sleepy-1619553360   1/1           7s         32s
```
