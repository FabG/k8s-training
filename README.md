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

### Resources
 - [LFD5461 Course (pdf)](course/LFD5461-WM_V1.20.pdf)
 - Keys for accessing the Linux Foundation linux hosts and run labs/k8s: see keys folder.
 - Instructor: Tim Serewicz / tim@serewicz.com
 - Company: [Linux Foundation](https://www.linuxfoundation.org/)
 - Training Manager: Kevlin Husser / kevlin@linuxfoundation.org / (925) 216-2955
