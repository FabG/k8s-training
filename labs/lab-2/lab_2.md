# lab 2
This lab is related to the K8s course chapter 3 - Build (starting page 39)

*Notes*: you can use the `--dry-un` parameter when running `kubectl` commands to see what would the command return. This is particularly convenient when running `delete` command.
Example:
```
kubectl delete deplyment -l app=tim1 --dry-run=client
```

*Notes*: you can use the `--v=X` parameter where `X` is an integer that control the verobisty of the output.
Example:
```
kubectl get node --v=10
```



#### Exercise 3.1: Deploy a New Application
In this lab we will deploy a very simple Python application, test it using Docker, ingest it into Kubernetes and configure
probes to ensure it continues to run. This lab requires the completion of the previous lab, the installation and configuration of a Kubernetes cluster.

1. Install python on your master node. It may already be installed, as is shown in the output below.
```
student@master:˜$ sudo apt-get -y install python
```

2. Locate the python binary on your system.
```
student@master:˜$ which python
/usr/bin/python
```

3. Create and change into a new directory. The Docker build process pulls everything from the current directory into the
image file by default. Make sure the chosen directory is empty.
```
student@master:˜$ mkdir app1
student@master:˜$ cd app1
student@master:˜/app1$ ls -l
total 0
```

4. Create a simple python script which prints the time and hostname every 5 seconds. There are six commented parts to
this script, which should explain what each part is meant to do. The script is included with others in the course tar file,
though you are encouraged to create the file by hand if not already familiar with the process. While the command shows
vim as an example other text editors such as nano work just as well.
```
student@master:˜/app1$ vim simple.py
```

Content of the `simple.py`:
```python
#!/usr/bin/python
## Import the necessary modules
import time
import socket

## Use an ongoing while loop to generate output
while True :

## Set the hostname and the current date
  host = socket.gethostname()
  date = time.strftime("%Y-%m-%d %H:%M:%S")

## Convert the date output to a string
  now = str(date)

## Open the file named date in append mode
## Append the output of hostname and time
  f = open("date.out", "a" )
  f.write(now + "\n")
  f.write(host + "\n")
  f.close()

## Sleep for five seconds then continue the loop
  time.sleep(5)
```


5. Make the file executable and test that it works. Use Ctrl-C to interrupt the while loop after 20 or 30 seconds. The
output will be sent to a newly created file in your current directory called date.out.
```
student@master:˜/app1$ chmod +x simple.py
student@master:˜/app1$ ./simple.py
^CTraceback (most recent call last):
  File "./simple.py", line 24, in <module>
    time.sleep(5)
KeyboardInterrupt
```

6. and timedate stamps.
```
student@master:˜/app1$ cat date.out
2021-04-27 14:36:36
master-73kx
2021-04-27 14:36:41
master-73kx
2021-04-27 14:36:46
master-73kx
2021-04-27 14:36:51
master-73kx
2021-04-27 14:36:56
master-73kx
```

7. Create a text file named `Dockerfile`.
*Notes*: The name is important: it cannot have a suffix.

We will use three statements, `FROM` to declare which version of Python to use, `ADD` to include our script and `CMD` to indicate the action of the container. Should you be including more complex tasks you may need to install extra libraries, shown commented out as RUN pip install in the following example.
```
student@master:˜/app1$ vim Dockerfile
```

Here is the content of `Dockerfile`:
```
FROM python:2
ADD simple.py /
## RUN pip install pystrich
CMD [ "python", "./simple.py" ]
```

8. Build the container.
The output below shows mid-build as necessary software is downloaded. You will need to use sudo in order to run this command. After the three step process completes the last line of output should indicate success.
Note the dot (.) at the end of the command indicates the current directory.
```
student@master:˜/app1$ sudo docker build -t simpleapp .
Sending build context to Docker daemon  4.608kB
Step 1/3 : FROM python:2
2: Pulling from library/python
7e2b2a5af8f6: Pull complete
09b6f03ffac4: Pull complete
dc3f0c679f0f: Pull complete
fd4b47407fc3: Pull complete
b32f6bf7d96d: Pull complete
6f4489a7e4cf: Pull complete
af4b99ad9ef0: Pull complete
39db0bc48c26: Pull complete
acb4a89489fc: Pull complete
Digest: sha256:cfa62318c459b1fde9e0841c619906d15ada5910d625176e24bf692cf8a2601d
Status: Downloaded newer image for python:2
 ---> 68e7be49c28c
Step 2/3 : ADD simple.py /
 ---> cea7eb08eb37
Step 3/3 : CMD [ "python", "./simple.py" ]
 ---> Running in e3a264b2fd2f
Removing intermediate container e3a264b2fd2f
 ---> a7f6982fcec0
Successfully built a7f6982fcec0
Successfully tagged simpleapp:latest
```

9. Verify you can see the new image among others downloaded during the build process, installed to support the cluster,
or you may have already worked with. The newly created simpleapp image should be listed first.
```
student@master:˜/app1$ sudo docker images
REPOSITORY                           TAG        IMAGE ID       CREATED          SIZE
simpleapp                            latest     a7f6982fcec0   44 seconds ago   902MB
calico/node                          v3.18.1    50b52cdadbcf   6 weeks ago      172MB
calico/pod2daemon-flexvol            v3.18.1    3994c62982cc   6 weeks ago      21.7MB
calico/cni                           v3.18.1    21fdaa2fccee   6 weeks ago      131MB
...
```

10. Use `sudo docker` to run a container using the new image. While the script is running you won’t see any output and the shell will be occupied running the image in the background. After 30 seconds use ctrl-c to interrupt. The local `date.out` file will not be updated with new times, instead that output will be a file of the container image.
```
student@master:˜$ sudo docker run simpleapp
^CTraceback (most recent call last):
  File "./simple.py", line 24, in <module>
    time.sleep(5)
KeyboardInterrupt
```

11. Locate the newly created `date.out` file. The following command should show two files of this name, the one created when we ran simple.py and another under /var/lib/docker when run via a Docker container.
```
student@master:˜/app1$ sudo find / -name date.out
/home/student/app1/date.out
/var/lib/docker/overlay2/459f2bbcbaa4456090ff6ddf2e2637c4b8ae9b394c5a41c0abea09b47a2c06f0/diff/date.out
```

12. View the contents of the `date.out` file created via Docker. Note the need for sudo as Docker created the file this time, and the owner is root. The long name is shown on several lines in the example, but would be a single line when typed or copied.
```
student@master:˜/app1$ sudo tail /var/lib/docker/overlay2/459f2bbcbaa4456090ff6ddf2e2637c4b8ae9b394c5a41c0abea09b47a2c06f0/diff/date.out
2021-04-27 14:43:17
f56404f74fa5
2021-04-27 14:43:22
f56404f74fa5
2021-04-27 14:43:27
f56404f74fa5
2021-04-27 14:43:32
f56404f74fa5
2021-04-27 14:43:37
f56404f74fa5
```


#### Exercise 3.2: Configure A Local Docker Repo
While we could create an account and upload our application to hub.docker.com, thus sharing it with the world, we will
instead create a local repository and make it available to the nodes of our cluster

1. We’ll need to complete a few steps with special permissions, for ease of use we’ll become root using sudo.
```
student@master:˜/app1$ cd
student@master:˜$ sudo -i
root@master-73kx:~#
```

2. Install the docker-compose software and utilities to work with the nginx server which will be deployed with the registry.
```
root@master:˜# apt-get install -y docker-compose apache2-utils
```

3. Create a new directory for configuration information. We’ll be placing the repository in the root filesystem. A better location may be chosen in a production environment.
```
root@master:˜# mkdir -p /localdocker/data
root@master:˜# cd /localdocker/
```

4. Create a Docker compose file. Inside is an entry for the nginx web server to handle outside traffic and a registry entry listening to loopback port 5000 for running a local Docker registry.
```
root@master:/localdocker# vim docker-compose.yaml
```

Here is the content of `docker-compose.yaml`
```
nginx:
  image: "nginx:1.17"
  ports:
    - 443:443
  links:
    - registry:registry
  volumes:
    - /localdocker/nginx/:/etc/nginx/conf.d
registry:
  image: registry:2
  ports:
    - 127.0.0.1:5000:5000
  environment:
    REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
  volumes:
    - /localdocker/data:/data
```


5. Use the docker-compose up command to create the containers declared in the previous step YAML file. This will
capture the terminal and run until you use ctrl-c to interrupt. There should be five registry_1 entries with info messages
about memory and which port is being listened to. Once we’re sure the Docker file works we’ll convert to a Kubernetes
tool. Let it run. You will use ctrl-c in a few steps.
```
root@master:/localdocker# docker-compose up
Pulling registry (registry:2)...
2: Pulling from library/registry
ddad3d7c1e96: Pull complete
6eda6749503f: Pull complete
363ab70c2143: Pull complete
5b94580856e6: Pull complete
12008541203a: Pull complete
Digest: sha256:bac2d7050dc4826516650267fe7dc6627e9e11ad653daca0641437abdf18df27
Status: Downloaded newer image for registry:2
Pulling nginx (nginx:1.17)...
1.17: Pulling from library/nginx
afb6ec6fdc1c: Pull complete
b90c53a0b692: Pull complete
11fa52a0fdc0: Pull complete
Digest: sha256:6fff55753e3b34e36e24e37039ee9eae1fe38a6420d8ae16ef37c92d1eb26699
Status: Downloaded newer image for nginx:1.17
Creating localdocker_registry_1 ...
Creating localdocker_registry_1 ... done
Creating localdocker_nginx_1 ...
Creating localdocker_nginx_1 ... done
Attaching to localdocker_registry_1, localdocker_nginx_1
registry_1  | time="2021-04-27T14:49:39.020453227Z" level=warning msg="No HTTP secret provided - generated random secret. This may cause problems with uploads if multiple registries are behind a load-balancer. To provide a shared secret, fill in http.secret in the configuration file or set the REGISTRY_HTTP_SECRET environment variable." go.version=go1.11.2 instance.id=bfb185ff-bd38-48a5-9250-29f70091719c service=registry version=v2.7.1
registry_1  | time="2021-04-27T14:49:39.020689299Z" level=info msg="redis not configured" go.version=go1.11.2 instance.id=bfb185ff-bd38-48a5-9250-29f70091719c service=registry version=v2.7.1
registry_1  | time="2021-04-27T14:49:39.024526321Z" level=info msg="Starting upload purge in 16m0s" go.version=go1.11.2 instance.id=bfb185ff-bd38-48a5-9250-29f70091719c service=registry version=v2.7.1
registry_1  | time="2021-04-27T14:49:39.036890947Z" level=info msg="using inmemory blob descriptor cache" go.version=go1.11.2 instance.id=bfb185ff-bd38-48a5-9250-29f70091719c service=registry version=v2.7.1
registry_1  | time="2021-04-27T14:49:39.037180479Z" level=info msg="listening on [::]:5000" go.version=go1.11.2 instance.id=bfb185ff-bd38-48a5-9250-29f70091719c service=registry version=v2.7.1
```

6. Test that you can access the repository. Open a second terminal to the master node. Use the curl command to test the
repository. It should return {}, but does not have a carriage-return so will be on the same line as the following prompt.
You should also see the GET request in the first, captured terminal, without error. Don’t forget the trailing slash. You’ll
see a “Moved Permanently” message if the path does not match exactly.

In the new terminal, I get a `{}`
```
student@master-73kx:~$ curl http://127.0.0.1:5000/v2/
{}student@master-73kx:~$
```

I can see in the first terminal a new registry:
```
registry_1  | 172.17.0.1 - - [27/Apr/2021:14:52:16 +0000] "GET /v2/ HTTP/1.1" 200 2 "" "curl/7.58.0"
```

7. Now that we know docker-compose format is working, ingest the file into Kubernetes using kompose. Use ctrl-c to stop the previous docker-compose command.
```
^CGracefully stopping... (press Ctrl+C again to force)
Stopping localdocker_nginx_1    ... done
Stopping localdocker_registry_1 ... done
```

8. Download the kompose binary and make it executable. The command can run on a single line. Note that the option
following the dash is the letter as in output. Also that is a zero, not capital O (ohh) in the short URL. The short URL goes
here: https://github.com/kubernetes/kompose/releases/download/v1.1.0/kompose-linux-amd64
```
root@master:/localdocker# curl -L https://bit.ly/2tN0bEa -o kompose
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                               Dload  Upload   Total   Spent    Left  Speed
100   169  100   169    0     0   3313      0 --:--:-- --:--:-- --:--:--  3313
100   625  100   625    0     0   4340      0 --:--:-- --:--:-- --:--:--  4340
100 45.3M  100 45.3M    0     0  41.8M      0  0:00:01  0:00:01 --:--:-- 60.0M
root@master:/localdocker# chmod +x kompose
```

9. Move the binary to a directory in our `$PATH`. Then return to your non-root user.
```
root@master:/localdocker# mv ./kompose /usr/local/bin/kompose
root@master:/localdocker# exit
```

10. Create two physical volumes in order to deploy a local registry for Kubernetes. 200Mi for each should be enough for
each of the volumes. Use the `hostPath` storageclass for the volumes.
More details on how persistent volumes and persistent volume claims are covered in an upcoming chapter, Deployment
Configuration.
```
student@master:˜$ vim vol1.yaml
```

Here is the content:
```
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    type: local
  name: task-pv-volume
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 200Mi
  hostPath:
    path: /tmp/data
  persistentVolumeReclaimPolicy: Retain
```


```
student@master:˜$ vim vol2.yaml
```

Here is the content:
```
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    type: local
  name: registryvm
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 200Mi
  hostPath:
    path: /tmp/nginx
  persistentVolumeReclaimPolicy: Retain
```

11. Create both volumes.
```
student@master:˜$ kubectl create -f vol1.yaml
1 persistentvolume/task-pv-volume created
student@master:˜$ kubectl create -f vol2.yaml
1 persistentvolume/registryvm created
```

12. Verify both volumes have been created. They should show an Available status.

```
student@master:˜$ kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
registryvm       200Mi      RWO            Retain           Available                                   36s
task-pv-volume   200Mi      RWO            Retain           Available                                   40s
```

13. Go to the configuration file directory for the local Docker registry.
```
student@master:˜$ cd /localdocker/
student@master:˜/localdocker$ ls
data  docker-compose.yaml  nginx
```

14. Convert the Docker file into a single YAML file for use with Kubernetes. Not all objects convert exactly from Docker to
`kompose`, you may get errors about the mount syntax for the new volumes. They can be safely ignored.
```
student@master:˜/localdocker$ sudo kompose convert -f docker-compose.yaml -o localregistry.yaml
WARN Volume mount on the host "/localdocker/nginx/" isn't supported - ignoring path on the host
WARN Volume mount on the host "/localdocker/data" isn't supported - ignoring path on the host
```

15. Review the file. You’ll find that multiple Kubernetes objects will have been created such as `Services`,
`Persistent Volume Claims` and `Deployments` using environmental parameters and volumes to configure the container within.
```
student@master:/localdocker$ less localregistry.yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Service
  metadata:
    annotations:
      kompose.cmd: kompose convert -f docker-compose.yaml -o localregistry.yaml
      kompose.version: 1.1.0 (36652f6)
    creationTimestamp: null
    labels:
      io.kompose.service: nginx
    name: nginx
  spec:
...
```


16. View the cluster resources prior to deploying the registry. Only the cluster service and two available persistent volumes should exist in the default namespace.
```
student@master:˜/localdocker$ kubectl get pods,svc,pvc,pv,deploy
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   20h
NAME                              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
persistentvolume/registryvm       200Mi      RWO            Retain           Available                                   3m51s
persistentvolume/task-pv-volume   200Mi      RWO            Retain           Available                                   3m55s
```

17. To illustrate the fast changing nature of Kubernetes you will show that the API has changed for Deployments. With
each new release of Kubernetes you may want to plan on a YAML review. First determine new object settings and
configuration, then compare and contrast to your existing YAML files. Edit and test for the new configurations.
Another more common way to find YAML issues is to attempt to create an object using previous YAML in a new version
of Kubernetes and track down errors. Not suggested, but what often happens instead of the following process.
To view the current cluster requirements use the `--dry-run` option for the `kubectl create` command to see what the
API now uses. We can compare the current values to our existing (previous version) YAML files. This will help determine
what to edit for the local registry in an upcoming step.


```
student@master:˜/localdocker$ kubectl create deployment drytry --image=nginx --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: drytry
  name: drytry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: drytry
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: drytry
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```


18. From the previous command output and comparing line by line to objects in the existing localregistry.yaml file
output we can see that the apiVersion of the Deployment object has changed, and we need to add selector, add
matchLabels, and a label line. The three lines to add will be part of the replicaSet information, right after the replicas
line, with selector the same indentation as replicas.
Following is a diff output, a common way to compare two files to each other, before and after an edit. Use the man page
to decode the output if you are not already familiar with the command.
```
student@master:˜/localdocker$ sudo cp localregistry.yaml old-localregistry.yaml
student@master:˜/localdocker$ sudo vim localregistry.yaml
<make edits>
student@master:˜/localdocker$ diff localregistry.yaml old-localregistry.yaml
41c41
< - apiVersion: apps/v1
---
> - apiVersion: extensions/v1beta1
53,55d52
<     selector:
<      matchLabels:
<       io.kompose.service: nginx
93c90
< - apiVersion: apps/v1
---
> - apiVersion: extensions/v1beta1
105,107d101
<     selector:
<       matchLabels:
<         io.kompose.service: registry
```

19. Use kubectl to create the local docker registry.
```
student@master:˜/localdocker$ kubectl create -f localregistry.yaml
```

20. View the newly deployed resources. The persistent volumes should now show as `Bound`. Be aware that due to the
manner that volumes are bound it is possible that the registry claim may not to be bound to the registry volume. Find
the service IP for the registry. It should be sharing port 5000. In the example below the IP address is 10.104.27.122.
```
student@master:˜/localdocker$ kubectl get pods,svc,pvc,pv,deploy
NAME                           READY   STATUS    RESTARTS   AGE
pod/nginx-79f868896b-svbqp     1/1     Running   0          17s
pod/registry-6b5bb79c4-ddsqm   1/1     Running   0          17s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    20h
service/nginx        ClusterIP   10.98.38.243    <none>        443/TCP    3m13s
service/registry     ClusterIP   10.104.27.122   <none>        5000/TCP   3m13s

NAME                                    STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/nginx-claim0      Bound    registryvm       200Mi      RWO                           3m13s
persistentvolumeclaim/registry-claim0   Bound    task-pv-volume   200Mi      RWO                           3m13s

NAME                              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS   REASON   AGE
persistentvolume/registryvm       200Mi      RWO            Retain           Bound    default/nginx-claim0                              17m
persistentvolume/task-pv-volume   200Mi      RWO            Retain           Bound    default/registry-claim0                           17m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx      1/1     1            1           18s
deployment.apps/registry   1/1     1            1           18s
```

21. Verify you get the same {} response using the Kubernetes deployed registry as we did when using docker-compose.
Note you must use the trailing slash after v2. Please also note that if the connection hangs it may be due to a firewall
issue. If running your nodes using GCE ensure your instances are using VPC setup and all ports are allowed. If using
AWS also make sure all ports are being allowed.
Edit the IP address to that of your registry service.
```
student@master:˜/localdocker$ curl http://10.104.27.122:5000/v2/
{}student@master-73kx:/localdocker$
```
22. Edit the Docker configuration file to allow insecure access to the registry. In a production environment steps should be
taken to create and use TLS authentication instead. Use the IP and port of the registry you verified in the previous step.
```
student@master:˜$ sudo vim /etc/docker/daemon.json
{ "insecure-registries":["10.104.27.122:5000"] }
```

23. Restart docker on the local system. It can take up to a minute for the restart to take place. Ensure the service is active.
It should report that the service recently became status as well.
```
student@master:˜$ sudo systemctl restart docker.service
student@master:˜$ sudo systemctl status docker.service | grep Active
   Active: active (running) since Tue 2021-04-27 15:19:01 UTC; 12s ago
```

24. Download and tag a typical image from hub.docker.com. Tag the image using the IP and port of the registry. We will
also use the latest tag.
```
student@master:˜$ sudo docker pull ubuntu
student@master:˜$ sudo docker tag ubuntu:latest 10.104.27.122:5000/tagtest
```

25. Push the newly tagged image to your local registry. If you receive an error about an HTTP request to an HTTPS client
check that you edited the /etc/docker/daemon.json file correctly and restarted the service.
```
student@master:˜$ sudo docker push 10.104.27.122:5000/tagtest
Using default tag: latest
The push refers to repository [10.104.27.122:5000/tagtest]
2f140462f3bc: Pushed
63c99163f472: Pushed
ccdbb80308cc: Pushed
latest: digest: sha256:86ac87f73641c920fb42cc9612d4fb57b5626b56ea2a19b894d0673fd5b4f2e9 size: 943
```

26. We will test to make sure we can also pull images from our local repository. Begin by removing the local cached images.
```
student@master:˜$ sudo docker image remove ubuntu:latest
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:cf31af331f38d1d7158470e095b132acd126a7180a54f263d386da88eb681d93
```

```
student@master:˜$ sudo docker image remove 10.104.27.122:5000/tagtest
Untagged: 10.104.27.122:5000/tagtest:latest
Untagged: 10.104.27.122:5000/tagtest@sha256:86ac87f73641c920fb42cc9612d4fb57b5626b56ea2a19b894d0673fd5b4f2e9
Deleted: sha256:7e0aa2d69a153215c790488ed1fcec162015e973e49962d438e18249d16fa9bd
Deleted: sha256:3dd8c8d4fd5b59d543c8f75a67cdfaab30aef5a6d99aea3fe74d8cc69d4e7bf2
Deleted: sha256:8d8dceacec7085abcab1f93ac1128765bc6cf0caac334c821e01546bd96eb741
Deleted: sha256:ccdbb80308cc5ef43b605ac28fac29c6a597f89f5a169bbedbb8dec29c987439
```

27. Pull the image from the local registry. It should report the download of a newer image.
```
student@master:˜$ sudo docker pull 10.104.27.122:5000/tagtest
Using default tag: latest
latest: Pulling from tagtest
345e3491a907: Pull complete
57671312ef6f: Pull complete
5e9250ddb7d0: Pull complete
Digest: sha256:86ac87f73641c920fb42cc9612d4fb57b5626b56ea2a19b894d0673fd5b4f2e9
Status: Downloaded newer image for 10.104.27.122:5000/tagtest:latest
10.104.27.122:5000/tagtest:latest
```

28. Use docker tag to assign the simpleapp image and then push it to the local registry. The image and dependent images
should be pushed to the local repository.
```
student@master:˜$ sudo docker tag simpleapp 10.104.27.122:5000/simpleapp
student@master:˜$ sudo docker push 10.104.27.122:5000/simpleapp
Using default tag: latest
The push refers to repository [10.104.27.122:5000/simpleapp]
a9548d9ea269: Pushed
e571d2d3c73c: Pushed
da7b0a80a4f2: Pushed
ceee8816bb96: Pushed
47458fb45d99: Pushed
46829331b1e4: Pushed
d35c5bda4793: Pushed
a3c1026c6bcc: Pushed
f1d420c2af1a: Pushed
461719022993: Pushed
latest: digest: sha256:4d818d353a4330e9e8d12b6442972059166677b6f1f34cef7af70b5aed5c90ed size: 2428
```

29. Configure the worker (second) node to use the local registry running on the master server. Connect to the worker node.
Edit the Docker daemon.json file with the same values as the master node and restart the service. Ensure it is active.
```
student@worker:˜$ sudo vim /etc/docker/daemon.json
{ "insecure-registries":["10.104.27.122:5000"] }
```

```
student@worker:˜$ sudo systemctl restart docker.service
student@worker:˜$ sudo systemctl status docker.service
```

30. From the worker node, pull the recently pushed image from the registry running on the master node.
```
student@worker:˜$ sudo docker pull 10.104.27.122:5000/simpleapp
```

31. Return to the master node and deploy the simpleapp in Kubernetes with several replicas. We will name the deployment
try1. Scale to have six replicas. Multiple replicas the scheduler should run some containers on each node.
```
student@master:˜$ kubectl create deployment try1 --image=10.104.27.122:5000/simpleapp
deployment.apps/try1 created
```

```
student@master:˜$ kubectl scale deployment try1 --replicas=6
deployment.apps/try1 scaled
```

32. View the running pods. You should see six replicas of simpleapp as well as two running the locally hosted image
repository.
```
student@master:˜$ kubectl get pods
nginx-79f868896b-svbqp     1/1     Running   1          13m
registry-6b5bb79c4-ddsqm   1/1     Running   1          13m
try1-7c9f9899b8-2ppv4      1/1     Running   0          33s
try1-7c9f9899b8-4g2jx      1/1     Running   0          33s
try1-7c9f9899b8-7hq54      1/1     Running   0          33s
try1-7c9f9899b8-dkbvh      1/1     Running   0          33s
try1-7c9f9899b8-fqd8t      1/1     Running   0          33s
try1-7c9f9899b8-kr5sk      1/1     Running   0          58s
```

33. On the second node use sudo docker ps to verify containers of simpleapp are running. The scheduler will usually
balance pod count across nodes. As the master already has several pods running the new pods may be on the worker.
```
student@worker:˜$ sudo docker ps | grep simple
483f73d2a492   10.104.27.122:5000/simpleapp   "python ./simple.py"     About a minute ago   Up About a minute             k8s_simpleapp_try1-7c9f9899b8-4g2jx_default_058eb53c-b3c2-4749-9dfe-745f106b18a6_0
3e4d58f5991c   10.104.27.122:5000/simpleapp   "python ./simple.py"     About a minute ago   Up About a minute             k8s_simpleapp_try1-7c9f9899b8-2ppv4_default_7562eaae-8b1b-4923-97bd-8c7c52d295e7_0
d638b3a5ba82   10.104.27.122:5000/simpleapp   "python ./simple.py"     About a minute ago   Up About a minute             k8s_simpleapp_try1-7c9f9899b8-fqd8t_default_ca0c359a-9c7b-478b-968a-ba92db57cb0e_0
23694aaeda5a   10.104.27.122:5000/simpleapp   "python ./simple.py"     About a minute ago   Up About a minute             k8s_simpleapp_try1-7c9f9899b8-7hq54_default_56b2c7c5-de51-4a47-8eab-5cb26d1174ea_0
f2dee96fde12   10.104.27.122:5000/simpleapp   "python ./simple.py"     About a minute ago   Up About a minute             k8s_simpleapp_try1-7c9f9899b8-dkbvh_default_76612ac3-c3bf-42fb-9bf5-3c38defb1998_0
01b73d4eaf9a   10.104.27.122:5000/simpleapp   "python ./simple.py"     2 minutes ago        Up 2 minutes                  k8s_simpleapp_try1-7c9f9899b8-kr5sk_default_e9481582-0273-4ae6-93a4-dc53a7a180a2_0
```

34. Return to the master node. Save the try1 deployment as YAML.
```
student@master:˜/app1$ cd ˜/app1/
student@master:˜/app1$ kubectl get deployment try1 -o yaml > simpleapp.yaml
```

35. Delete and recreate the try1 deployment using the YAML file. Verify the deployment is running with the expected six
replicas.
```
student@master:˜$ kubectl delete deployment try1
deployment.apps "try1" deleted
```

```
student@master:˜/app1$ kubectl create -f simpleapp.yaml
deployment.apps/try1 created
```

```
student@master:˜/app1$ kubectl get deployment
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
nginx      1/1     1            1           17m
registry   1/1     1            1           17m
try1       6/6     6            6           37s
```


#### Exercise 3.3: Configure Probes
When large datasets need to be loaded or a complex application launched prior to client access, a readinessProbe
can be used. The pod will not become available to the cluster until a test is met and returns a successful exit code.
Both `readinessProbes` and `livenessProbes` use the same syntax and are identical other than the name. Where the
`readinessProbe` is checked prior to being ready, then not again, the `livenessProbe` continues to be checked.

There are three types of liveness probes:
 - a command returns a zero exit value, meaning success
 - an HTTP request returns a response code in the 200 to 399 range
 - the third probe uses a TCP socket.

In this example we’ll use a command, cat, which will return a zero exit code when the file /tmp/healthy has been created and can be accessed.


1. Edit the YAML deployment file and add the stanza for a readinessprobe. Remember that when working with YAML
whitespace matters. Indentation is used to parse where information should be associated within the stanza and the
entire file. Do not use tabs. If you get an error about validating data, check the indentation. It can also be helpful to
paste the file to this website to see how indentation affects the JSON value, which is actually what Kubernetes ingests:
https://www.json2yaml.com/ An edited file is also included in the tarball, but requires the image name to be edited to
match your registry IP address.
```
student@master:˜/app1$ vim simpleapp.yaml
```

Add:
```
...
readinessProbe:
  periodSeconds: 5
  exec:
    command:
    - cat
    - /tmp/healthy
...
```

2. Delete and recreate the try1 deployment.
```
student@master:˜/app1$ kubectl delete deployment try1
deployment.apps "try1" deleted
student@master:˜/app1$ kubectl create -f simpleapp.yaml
deployment.apps/try1 created
```

3. The new try1 deployment should reference six pods, but show zero available. They are all missing the /tmp/healthy
file.
```
student@master:˜/app1$ kubectl get deployment
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
nginx      1/1     1            1           27m
registry   1/1     1            1           27m
try1       0/6     6            0           26s
```

4. Take a closer look at the pods. Choose one of the `try1` pods as a test to create the health check file.
```
student@master:˜/app1$ kubectl get pods
NAME                       READY   STATUS        RESTARTS   AGE
nginx-79f868896b-svbqp     1/1     Running       1          27m
registry-6b5bb79c4-ddsqm   1/1     Running       1          27m
try1-74f9446d7b-6tl7k      0/1     Running       0          71s
try1-74f9446d7b-8nwsx      0/1     Running       0          71s
try1-74f9446d7b-g77bj      0/1     Running       0          72s
try1-74f9446d7b-r4zct      0/1     Running       0          71s
try1-74f9446d7b-vwwjx      0/1     Running       0          71s
try1-74f9446d7b-xlhv9      0/1     Running       0          72s
try1-7c9f9899b8-9dnfd      0/1     Terminating   0          10m
try1-7c9f9899b8-n6gch      0/1     Terminating   0          10m
try1-7c9f9899b8-rvkwr      0/1     Terminating   0          10m
```

5. Run the bash shell interactively and touch the /tmp/healthy file.
```
student@master:˜/app1$ kubectl exec -it try1-7c9f9899b8-rvkwr -- /bin/bash
root@try1-9869bdb88-rtchc:kubectl exec -it try1-74f9446d7b-xlhv9 -- /bin/bash
root@try1-74f9446d7b-xlhv9:/# touch /tmp/healthy
root@try1-9869bdb88-rtchc:/# exit
```

6. Wait at least five seconds, then check the pods again. Once the probe runs again the container should show available
quickly. The pod with the existing /tmp/healthy file should be running and show 1/1 in a READY state. The rest will
continue to show 0/1.
```
student@master:˜/app1$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
nginx-79f868896b-svbqp     1/1     Running   1          31m
registry-6b5bb79c4-ddsqm   1/1     Running   1          31m
try1-74f9446d7b-6tl7k      0/1     Running   0          4m31s
try1-74f9446d7b-8nwsx      0/1     Running   0          4m31s
try1-74f9446d7b-g77bj      0/1     Running   0          4m32s
try1-74f9446d7b-r4zct      0/1     Running   0          4m31s
try1-74f9446d7b-vwwjx      0/1     Running   0          4m31s
try1-74f9446d7b-xlhv9      1/1     Running   0          4m32s
```

7. Touch the file in the remaining pods. Consider using a for loop, as an easy method to update each pod.
```
student@master:˜$ for name in try1-74f9446d7b-6tl7k try1-74f9446d7b-8nwsx try1-74f9446d7b-g77bj try1-74f9446d7b-r4zct try1-74f9446d7b-vwwjx
> do
> kubectl exec $name -- touch /tmp/healthy
> done
```

8. It may take a short while for the probes to check for the file and the health checks to succeed.
```
student@master:˜/app1$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
nginx-79f868896b-svbqp     1/1     Running   1          33m
registry-6b5bb79c4-ddsqm   1/1     Running   1          33m
try1-74f9446d7b-6tl7k      1/1     Running   0          7m7s
try1-74f9446d7b-8nwsx      1/1     Running   0          7m7s
try1-74f9446d7b-g77bj      1/1     Running   0          7m8s
try1-74f9446d7b-r4zct      1/1     Running   0          7m7s
try1-74f9446d7b-vwwjx      1/1     Running   0          7m7s
try1-74f9446d7b-xlhv9      1/1     Running   0          7m8s
```

9. Now that we know when a pod is healthy, we may want to keep track that it stays healthy, using a `livenessProbe`. You
could use one probe to determine when a pod becomes available and a second probe, to a different location, to ensure
ongoing health.
Edit the deployment again. Add in a livenessProbe section as seen below. This time we will add a Sidecar container
to the pod running a simple application which will respond to port 8080. Note that the dash (-) in front of the name. Also
goproxy is indented the same number of spaces as the - in front of the image: line for simpleapp earlier in the file. In
this example that would be seven spaces
```
student@master:˜/app1$ vim simpleapp.yaml
```

Added:
```
...
resources: {}
 terminationMessagePath: /dev/termination-log
 terminationMessagePolicy: File
- name: goproxy               #<-- From here
 image: k8s.gcr.io/goproxy:0.1
 ports:
 - containerPort: 8080
 readinessProbe:
  tcpSocket:
   port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
 livenessProbe:
  tcpSocket:
   port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20          #<-- To here
dnsPolicy: ClusterFirst
restartPolicy: Always
...
```


10. Delete and recreate the deployment.
```
student@master:˜$ kubectl delete deployment try1
deployment.apps "try1" deleted
```

```
student@master:˜$ kubectl create -f simpleapp.yaml
deployment.apps/try1 created
```

11. View the newly created pods. You’ll note that there are two containers per pod, and only one is running. The new
simpleapp containers will not have the /tmp/healthy file, so they will not become available until we touch the /tmp/
healthy file again. We could include a command which creates the file into the container arguments. The output below
shows it can take a bit for the old pods to terminate.
```
student@master:˜$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
nginx-79f868896b-svbqp     1/1     Running   1          43m
registry-6b5bb79c4-ddsqm   1/1     Running   1          43m
try1-6bc6f69476-4j9d6      1/2     Running   0          47s
try1-6bc6f69476-fttlq      1/2     Running   0          47s
try1-6bc6f69476-k6hmx      1/2     Running   0          47s
try1-6bc6f69476-n7mk4      1/2     Running   0          47s
try1-6bc6f69476-q49gt      1/2     Running   0          47s
try1-6bc6f69476-xhpxf      1/2     Running   0          47s
```

12. Create the health check file for the readinessProbe. You can use a for loop again for each action, this setup will
leverage labels so you don’t have to look up the pod names. As there are now two containers in the pod, you should
include the container name for which one will execute the command. If no name is given, it will default to the first
container. Depending on how you edited the YAML file try1 should be the first pod and goproxy the second. To ensure
the correct container is updated, add -c simpleapp to the kubectl command. Your pod names will be different. Use the
names of the newly started containers from the kubectl get pods command output. Note the >character represents
the secondary prompt, you would not type in that character.

```
student@master:˜$ for name in $(kubectl get pod -l app=try1 -o name)
> do
> kubectl exec $name -c simpleapp -- touch /tmp/healthy
> done
```

13. In the next minute or so the Sidecar container in each pod, which was not running, will change status to Running. Each
should show 2/2 containers running.
```
student@master:˜$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
nginx-79f868896b-svbqp     1/1     Running   1          44m
registry-6b5bb79c4-ddsqm   1/1     Running   1          44m
try1-6bc6f69476-4j9d6      2/2     Running   0          2m10s
try1-6bc6f69476-fttlq      2/2     Running   0          2m10s
try1-6bc6f69476-k6hmx      2/2     Running   0          2m10s
try1-6bc6f69476-n7mk4      2/2     Running   0          2m10s
try1-6bc6f69476-q49gt      2/2     Running   0          2m10s
try1-6bc6f69476-xhpxf      2/2     Running   0          2m10s
```


14. View the events for a particular pod. Even though both containers are currently running and the pod is in good shape,
note the events section shows the issue.
```
student@master:˜/app1$ kubectl describe pod try1-6bc6f69476-xhpxf | tail
----     ------     ----                  ----               -------
Normal   Scheduled  2m56s                 default-scheduler  Successfully assigned default/try1-6bc6f69476-xhpxf to worker-8f1k
Normal   Pulling    2m54s                 kubelet            Pulling image "10.104.27.122:5000/simpleapp"
Normal   Pulled     2m53s                 kubelet            Successfully pulled image "10.104.27.122:5000/simpleapp" in 796.708635ms
Normal   Created    2m53s                 kubelet            Created container simpleapp
Normal   Started    2m52s                 kubelet            Started container simpleapp
Normal   Pulled     2m52s                 kubelet            Container image "k8s.gcr.io/goproxy:0.1" already present on machine
Normal   Created    2m52s                 kubelet            Created container goproxy
Normal   Started    2m52s                 kubelet            Started container goproxy
Warning  Unhealthy  83s (x18 over 2m48s)  kubelet            Readiness probe failed: cat: /tmp/healthy: No such file or directory
```


15. If you look for the status of each container in the pod, they should show that both are Running and ready showing True.
```
student@master:˜/app1$ kubectl describe pod try1-6bc6f69476-xhpxf | grep -E 'State|Ready'
    State:          Running
    Ready:          True
    State:          Running
    Ready:          True
  Ready             True
  ContainersReady   True
```
