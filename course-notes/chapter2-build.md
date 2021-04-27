# Chapter 2 Kubernetes - Build

In this chapter, we are going to deploy our new Python application into our Kubernetes cluster. We’ll first containerize it, using
Docker commands, then create and use our own local registry. After testing that the registry is available to all of our nodes, we
will deploy our application in a multi-container pod. In keeping with the ideas of decoupled applications that are transient, we
will also use readinessProbes, to make sure that the application is fully running prior to accepting traffic from the cluster, and
then, livenessProbes, to ensure that the container and the application continues to run in a healthy state, and startupProbes
to allow for a slow-starting application.

### Container Options
- There are many competing organizations working with containers. As an orchestration tool, Kubernetes is being developed
to work with many of them, with the overall community moving toward open standards and easy interoperability.
The early and strong presence of Docker meant that historically, this was not the focus. As Docker evolved, spreading
their vendor-lock characteristics through the container creation and deployment life cycle, new projects and features
have become popular. As other container engines become mature Kubernetes continues to become more open and
independent.
- A `container runtime` is the component which runs the containerized application upon request. Docker Engine remains
the default for Kubernetes, though cri-o and others are gaining community support.
- The containerized image is moving from Docker to one that is not bound to higher-level tools and that is more portable
across operating systems and environments. The `Open Container Initiative (OCI)` was formed to help with this. Docker donated their libcontainer project to form a new code base called runC to support these goals. More information about
runC can be found on GitHub at https://github.com/opencontainers/runc.
- Where Docker was once the only real choice for developers, the trend toward open specifications and flexibility indicates
that building with vendor-neutral features is a wise choice.

#### Docker
- Launched in 2013, Docker has become synonymous with running containerized applications. Docker made containerizing,
deploying, and consuming applications easy. As a result, it became the default option in production. With an open registry of images, Docker Hub (https://hub.docker.com/), you can download and deploy vendor or individual-created images on multiple architectures with a single and easy to use tool set. This ease meant it was the sensible default choice for any developer as well. Kubernetes defaults to using the Docker engine to run containers.
- Over the past few years, Docker has continued to grow and add features, including orchestration, which they call `Swarm`.
These added features addressed some of the pressing needs in production, but also increased the vendor-lock and size of the product.
This has lead to an increase in open tools and specifications such as `cri-o`. A developer looking toward the future would be wise to work with mostly open tools for containers and Kubernetes, but he or she should understand that Docker is still the production tool of choice outside of a Red Hat environment at the moment.

- *Notes*: RedHat is done with Docker and other teams/companies are moving away from it.


#### CRI-O
- This project is currently in incubation as part of Kubernetes. It uses the Kubernetes Container Runtime Interface with
OCI-compatible runtimes, thus the name CRI-O (https://github.com/kubernetes-sigs/cri-o). Currently, there is support
for runC (default) and Clear Containers, but a stated goal of the project is to work with any OCI-compliant runtime.
- While newer than Docker or rkt, this project has gained major vendor support due to its flexibility and compatibility.


### Creating the Dockerfile
- Create a directory to hold application files. The docker build process will pull everything in the directory when it creates
the image. Move the scripts and files for the containerized application into the directory.
- Also, in the directory, create a `Dockerfile`. The name is important, as it must be those ten characters, beginning with a capital `D`; newer versions allow a different filename to be used after -f <filename>. This is the expected file for the
docker build to know how to create the image.
- Each instruction is iterated by the Docker daemon in sequence. The instructions are not case-sensitive, but are often written in uppercase to easily distinguish from arguments. This file must have the `FROM` instruction first. This declares the
base image for the container. Following can be a collection of `ADD`, `RUN` and a `CMD` to add resources and run commands to populate the image. More details about the Dockerfile reference at https://docs.docker.com/engine/reference/builder
can be found in the Docker documentation.
- Test the image by verifying that you can see it listed among other images, and use `docker run <app-name>` to execute it. Once you find the application performs as expected, you can push the image to a local repository in Docker Hub, after
creating an account. Here is process summary:
– Create Dockerfile
– Build the container
```
sudo docker build -t simpleapp
```
– Verify the image
```
sudo docker images
sudo docker run simpleapp
```
– Push to the repository
```
sudo docker push
```

### Hosting a Local Repository
- While Docker has made it easy to upload images to their Hub, these images are then public and accessible to the world.
A common alternative is to create a local repository and push images there. While it can add administrative overhead,
it can save downloading bandwidth, while maintaining privacy and security.
- Once the local repository has been configured, you can use docker tag, then push to populate the repository with local
images. Consider setting up an insecure repository until you know it works, then configuring TLS access for greater
security.

### Creating a Deployment
- Once you can push and pull images using the docker command, try to run a new deployment inside Kubernetes using
the image. The string passed to the --image argument includes the repository to use, the name of the application, then
the version.
- Use kubectl create to test the image:
```
kubectl create deployment <Deploy-Name> --image=<repo>/<app-name>:<version>
```
Example:
```
kubectl create deployment time-date --image=10.110.186.162:5000/simpleapp:v2.2
```
- Be aware that the string `latest` is just that: a string, and has no reference to actually being the latest. Only use this string if you have a process in place to name and rename versions of the application as they become available. If this is
not understood, you could be using the “latest” release, which is several releases older then the most current.
- Verify the Pod shows a running status and that all included containers are running as well.
```
kubectl get pods
```
- Test that the application is performing as expected, running whichever tempest and QA testing the application has.
Terminate a pod and test that the application is as transient as designed.
