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

See [Course Notes - chapter1-k8s-architecture.md](course-notes/chapter1-k8s-architecture.md)

#### 2. Build
In this chapter, we are going to deploy our new Python application into our Kubernetes cluster. We’ll first containerize it, using Docker commands, then create and use our own local registry. After testing that the registry is available to all of our nodes, we
will deploy our application in a multi-container pod. In keeping with the ideas of decoupled applications that are transient, we will also use `readinessProbes`, to make sure that the application is fully running prior to accepting traffic from the cluster, and then, `livenessProbes`, to ensure that the container and the application continues to run in a healthy state, and `startupProbes` to allow for a slow-starting application.
See [Course Notes - chapter 2 - build.md](course-notes/chapter2-build.md)





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
