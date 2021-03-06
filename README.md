# Kubernetes Training

This training is from Linux Foundation, covering Kubernetes for Developers with a bit of Administation too.
This is a custom Curriculum for CapitalOne. It includes lab work.

This course is for developers looking to learn how to deploy, configure, and test their containerized applications on a
multi-node Kubernetes cluster.

This course is delivered by the Linux Foundation - The Linux Foundation is the non-profit consortium dedicated to fostering the growth of Linux and many other Open Source Software (OSS) projects and communities.


### Objectives
By the end of this course, you should be able to:
– Containerize and deploy a new Python script.
– Configure the deployment with ConfigMaps, Secrets, and SecurityContexts.
– Understand multi-container Pod design.
– Configure probes for pod health.
– Update and roll back an application.
– Implement services and NetworkPolicies.
– Use PersistentVolumeClaims for state persistence.


### Preparing your system
Preparing Your System
- You will need a computer installed with a current Linux distribution, with the important developer tools (for compiling,
etc.) properly deployed.

- Once you have installed your favorite OS, go to https://training.linuxfoundation.org/cm/prep/ and download a copy of the
`ready-for.sh` script, either by downloading through your browser, or using wget:
```bash
$ wget https://training.linuxfoundation.org/cm/prep/ready-for.sh
$ chmod 755 ready-for.sh
$ ./ready-for.sh LFD5461
• If there are missing dependencies, you can do:
$ ./ready-for.sh --install LFD5461
```


### Chapters

#### 1. Kubernetes Architecture
In this chapter we are going to cover a little bit of the history of Kubernetes. This helps to understand the why things are
where they are. Then we’ll talk about `master` nodes and the main agents that work to make Kubernetes what it is. We’ll also talk about `minion` or `worker` nodes. These are nodes that use API calls back to the master node to understand what the configuration should be, and report back status.

We’ll also talk about `CNI network plugin configuration`, which continues to be the future of Kubernetes networking. And we’ll cover some common terms so as you learn more about Kubernetes, you’ll understand major components and how they work with the other components in our decoupled and transient environment.

- See [Course Notes - chapter1-k8s-architecture](course-notes/chapter1-k8s-architecture.md)
- See [Lab - chapter 1 - k8s-architecture](labs/lab-1/lab_1.md)


#### 2. Build
In this chapter, we are going to deploy our new Python application into our Kubernetes cluster. We’ll first containerize it, using Docker commands, then create and use our own local registry. After testing that the registry is available to all of our nodes, we
will deploy our application in a multi-container pod. In keeping with the ideas of decoupled applications that are transient, we will also use `readinessProbes`, to make sure that the application is fully running prior to accepting traffic from the cluster, and then, `livenessProbes`, to ensure that the container and the application continues to run in a healthy state, and `startupProbes` to allow for a slow-starting application.
- See [Course Notes - chapter 2 - build](course-notes/chapter2-build.md)
- See [Lab - chapter 2 - build](labs/lab-2/lab_2.md)


#### 3. APIs and Access
- See [Course Notes - chapter 3 - apis-access](course-notes/chapter3-apis-access.md)
- See [Lab - chapter 3 - api-access](labs/lab-3/lab_3.md)


#### 4. API Objects
- See [Course Notes - chapter 4 - api-objects](course-notes/chapter4-api-objects.md)
- See [Lab - chapter 4 - api-objects](labs/lab-4/lab_4.md)


#### 5. Design
In this chapter, we are going to talk about resource requirements for our applications, and the ability to set limits for CPU,
memory and storage in the root filesystem of the node. We’re also going to talk about containers that can assist our primary
application, such as a sidecar, which is typically something that enhances the primary application. For example, if our application
does not include logging, we can add a second container to the pod, to handle logging on our behalf. An adapter
container might conform the traffic to match the other applications or needs of our cluster. This allows for full flexibility for the
primary application, to remain generating the data that it does, and not have to be modified to fit that particular cluster. There
is also an ambassador container, which is our representation to the outside world. It can act as a proxy then, so that perhaps
I might want to split inbound traffic: if it’s a read, it goes to one container, and for write, it goes to a different container, allowing
me to optimize my environment and my application. We’ll also consider other application design concepts, that can be helpful
in the planning stage for the deployment of your application. We’ll then finish by talking about jobs, whether it’s a batch-like
job that will run a container once and be done, or cron jobs, which will run a container on a regular basis on our behalf.

- See [Course Notes - chapter 5 - design](course-notes/chapter5-design.md)
- See [Lab - chapter 5 - design](labs/lab-5/lab_5.md)


#### 6. Deployment Configuration
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

- See [Course Notes - chapter 6 - deployment configuration](course-notes/chapter6-deployment-configuration.md)
- See [Lab - chapter 6 - deployment configuration](labs/lab-6/lab_6.md)


#### 7. Custom Resource Definition

We have been working with built-in resources, or API endpoints. The flexibility of Kubernetes allows for **dynamic addition of
new resources** as well. Once these Custom Resources have been added the objects can be created and accessed using
standard calls and commands like `kubectl`.

- See [Course Notes - chapter 7 - deployment configuration](course-notes/chapter7-custom-resource-definition.md)
- See [Lab - chapter 7 - deployment configuration](labs/lab-7/lab_7.md)


#### 8. Scheduling

The larger and more diverse a Kubernetes deployment becomes the more administration of scheduling can be important. The
`kube-scheduler` determines which nodes will run a Pod.
Users can set the priority of a pod, which will allow preemption of lower priority pods. The eviction of lower priority pods would
then allow the higher priority pod to be scheduled.

- See [Course Notes - chapter 8 - scheduling](course-notes/chapter8-scheduling.md)
- See [Lab - chapter 8 - scheduling](labs/lab-8/lab_8.md)


#### 9. Security
In this chapter, we are going to talk about how an API call is ingested into the cluster. We will go through the details of the
three different phases each call goes through. We’ll look at the different types of authentication that are available to us, and
work with RBAC, which handles the authorization on our behalf. We’ll also configure pod policies. The use of a pod policy
allows us to configure the details of what a container can do, and how processes run within that container.
We’ll finish by understanding a network policy. If the network plugin honors a network policy, this allows you to isolate a pod
from other pods in the environment. The default Kubernetes architecture says that all pods should be able to see all pods. So,
this is a change from that historic approach. But, as we use it in a production environment, you may want to limit ingress

- See [Course Notes - chapter 9 - security](course-notes/chapter9-security.md)
- See [Lab - chapter 9 - security](labs/lab-9/lab_9.md)

#### 10. Exposing Applications
In this chapter, we are going to talk about exposing our applications using services. We’ll start by talking about a ClusterIP
service. This allows us to expose our application inside of the cluster only. A NodePort will create a ClusterIP and then also,
get a high number port from the node. This allows us then to expose our applications internally on a low number port, and
externally, but only on a high number port. The use of a LoadBalancer first creates a NodePort on our behalf. The difference is
it also sends an asynchronous request to a cloud provider to make use of a LoadBalancer. If we are not using a cloud provider,
or if a load balancer is not available, it continues to run as a NodePort. The fourth service that we are going to discuss is
called ExternalName. This is useful for handling requests to an outside DNS resource, instead of one inside of the cluster.
This is helpful if you were in the process of moving your application from outside of the cluster inside. We’ll also discuss the
configuration of an Ingress Controller. The two reasons that you might want to use an Ingress Controller is, for example, if
you have thousands of services, having them independent can become difficult to manage, and wasteful of resources. You
can then consolidate it to a single Ingress Controller. Or multiple Ingress Controllers if you want more flexibility. The second
reason is an Ingress Controller allows you to expose a low numbered port to your application. Without that, you could get
there, but you’d have to use iptables, which could become difficult and not terribly flexible to manage.


- See [Course Notes - chapter 9 - exposing applications](course-notes/chapter10-exposing-applications.md)
- See [Lab - chapter 10 - exposing applications](labs/lab-10/lab_10.md)

#### 11 and 12: Monitoring, Logging and Troubleshooting

Troubleshooting, which we’ll cover in this chapter, can be difficult in a distributed and decoupled environment like Kubernetes.
We will cover the basic flow that can be used to diagnose and identify issues you may have. We do not have cluster-wide
logging built-in into Kubernetes. So, we will talk about other projects you may want to implement, like Prometheus and Fluentd,
that offer monitoring and logging in a cluster-wide sense. We will talk about agent logs locally, and node logs locally, as well.
With conformance testing, we can ensure that the deployment of Kubernetes that we have chosen conforms to standards used
in the community. We’ll also talk about cluster certification, which is a way of knowing that the Kubernetes distribution you are
using is up to the standards of the community, in general.

Large and diverse workloads can be difficult to track, so monitoring of usage is essential. Monitoring is about collecting key
metrics such as CPU, memory, and disk usage, and network bandwidth on your nodes, as well as monitoring key metrics in
your applications. These features are being been ingested into Kubernetes with the Metric Server, which a cut-down version
of the now deprecated Heapster. Once installed the Metrics Server exposes a standard API which can be consumed by other
agents such as autoscalers. Once installed this endpoint can be found here on the master server: /apis/metrics/k8s.io/
Logging activity across all the nodes is another feature not part of Kubernetes. Using Fluentd can be a useful data collector
for a unified logging layer. Having aggregated logs can help visualize the issue and provides the ability to search all logs.
A good place to start when local network troubleshooting does not expose the root cause. It can be downloaded from
http://www.fluentd.org
Another project from cncf.io combines logging, monitoring, and alerting called Prometheus can be found here https:
//prometheus.io. It provides a time-series database as well as integration with Grafana for visualization and dashboards.


- See [Course Notes - chapter 11 - troubleshooting](course-notes/chapter11-troubleshooting.md)
- See [Lab - chapter 11 - troubleshooting](labs/lab-11/lab_11.md)
- See [Course Notes - chapter 12 - logging and troubleshooting](course-notes/chapter12-logging-troubleshooting.md)
- See [Lab - chapter 12 - logging and troubleshooting](labs/lab-12/lab_12.md)


### Getting ready for exam
- registration -needed to get certs for exam, and access to online class
 https://trainingportal.linuxfoundation.org/redeem
 code:  `lfd5461capitolone26april21`


### Resources
 - [LFD5461 Course (pdf)](course/LFD5461-WM_V1.20.pdf)
 - Keys for accessing the Linux Foundation linux hosts and run labs/k8s: see keys folder.
 - Instructor: Tim Serewicz / tim@serewicz.com
 - Company: [Linux Foundation](https://www.linuxfoundation.org/)
 - Training Manager: Kevlin Husser / kevlin@linuxfoundation.org / (925) 216-2955
 - [Large-scale cluster management at Google with Borg (paper)](https://research.google/pubs/pub43438/)
