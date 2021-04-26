# Chapter 1 - Kubernetes architecture

### 1. What is Kubernetes?

- Running a container on a laptop is relatively simple. Deploying and connecting containers across multiple hosts, scaling them, deploying applications without downtime, and service discovery among several aspects can be complex.

 - *Notes*: We are working with Legos, not a model airplane. We are not getting a "COST" all-included product/platform. Instead we need to build.

- Kubernetes addresses those challenges from the start with a set of primitives and a powerful open and extensible API.
The ability to add new objects and operators allows easy customization for various production needs. The ability to add
multiple schedulers and multiple API servers adds to this customization.
- According to the https://kubernetes.io website, Kubernetes is:
`“an open-source system for automating deployment, scaling, and management of containerized applications”`
 - *Notes*: Kubernetes is orchestration software for automating deployment, scaling and mananaging containerized applications - offering Flexibility, Durability.

- A key aspect of Kubernetes is that it builds on 15 years of experience at Google in a project called Borg.
- Google’s infrastructure started reaching high scale before virtual machines became pervasive in the datacenter, and containers provided a fine-grained solution for packing clusters efficiently. Efficiency in using clusters and managing distributed applications
has been at the core of Google challenges.

- In Greek, `knbernhthV` means the Helmsman, or pilot of the ship. Keeping with the maritime theme of Docker containers, *Kubernetes is the pilot of a ship of containers*. Due to the difficulty in pronouncing the name, many will use a nickname, `K8s`, as Kubernetes has eight letters between K and S. The nickname is said like Kate’s.

- Kubernetes can be an integral part of Continuous Integration/Continuous Delivery (CI/CD), as it offers many of the necessary components.
 - Continuous Integration - A consistent way to build and test software. Deploying new packages with code written each day, or every hour, instead of quarterly. Tools like Helm and Jenkins are often part of this with Kubernetes.
 - Continuous Delivery - An automated way to test and deploy software into various environments. Kubernetes handles the life cycle of containers and connection of infrastructure resources to make rolling updates and rollbacks
easy, among other deployment schemes.
 - There are several options and possible configurations when building a CI/CD pipeline. Tools such as Jenkins, Spinnaker, GitHub, GitLab, and Helm among others may be part of your particular pipeline.

  - *Notes*: Here is a pipeline typical flow: `Developer -> Jenkins -> Spinaker -> Helm -> Kubernetes`
  - *Notes*: solarwinds hack: someone took control of Jenkins. the SW took out the code pushed via Jenkins.
  - *Notes*: all the tools/code we use in lab, we can use it after the course to build our own cluster.


### 2. Components of Kubernetes
  - *Notes*: K8s is NOT another VM deployment tool. **Hardest part is writing proper code**, not Kubernetes

  - *Notes*: if we work for a large company and have a lot of legacy code, this is hard. Getting a K8s cluster up and running is easy. **Optimizing the code for K8s in more cases than not is NOT easy**. To note in our case (MLaaS), this may not be as hard as we already have containerzied our app as micro-services.

  - *Notes*: Key is that our micro-services need to be `transient` and `decoupled`. They need to be as small as reasonable - per task or function to be completed.
  - `microservice` - as small as reasonable - per task or function to be completed
  - `transient` - write all images with expectation that they be terminated any time and all the time. Can you use Chaos Monkey in production?
  - `decoupled` - least number of images possible. Changes should take place in secrets and configMaps.

  ==> We need to manage a small number of images. We want small/lean/transient micro-services so that transient changes (IP addresses, users, key rotating,...) are easy to apply and we don't need a new image for it.


- Developers new to Kubernetes sometimes assume it is another `virtual-machine` manager, similar to what they have been using for decades and continue to develop applications in the same way as prior to using Kubernetes. **This is a mistake**. The decoupled, transient, microservice architecture is not the same. Most legacy applications will need to be rewritten to optimally run in a cloud. In this diagram we see the legacy deployment strategy on the left with a monolithic applications deployed to nodes. On the right we see an example of the same functionality, on the same hardware, using
multiple microservices.
![legacy](../images/Legacy-vs-Cloud-Architecture.png)

- Communication to, as well as internally, between components is API call-driven, which allows for flexibility. Configuration
information is stored in a JSON format, but is most often written in YAML. Kubernetes agents convert the YAML to JSON
prior to persistence to the database.


### challenges
- Containers have seen a huge rejuvenation. They provide a great way to package, ship, and run applications - that is the `Docker` motto, now provided by many tools.
- The developer experience has been boosted tremendously thanks to containers. Containers, and Docker specifically, have empowered developers with ease of building container images, simplicity of sharing images via registries, and
providing a powerful user experience to manage containers. Now several tools such as `Buildah`, `Podman`, `cri-o`, `containerd`, `frakti`, and others allow for easy container creation and management.
- However, managing containers at scale and architecting a distributed application based on microservices’ principles is still challenging.
- You first need a continuous integration pipeline to build your container images, test them, and verify them. Then, you need a cluster of machines acting as your base infrastructure on which to run your containers. You also need a system to launch your containers, and watch over them when things fail and self-heal. You must be able to perform rolling updates and rollbacks, and eventually tear down the resource when no longer needed.
- All of these actions require flexible, scalable, and easy-to-manage network and storage. As containers are launched on any worker node, the network must join the resource to other containers, while still keeping the traffic secure from others. We also need a storage structure which provides and keeps or recycles storage in a seamless manner.

  - *Notes*: Would users notice if you ran **Chaos Monkey**, which terminates any container randomly? If so you may have more work making containers and applications more decoupled and transient.
