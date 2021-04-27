# Chapter 5 Kubernetes - Design


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
job that will run a container once and be done, or cron jobs, which will run a container on a regular basis on our behalf

#### 1. Decoupled Resources
- The use of decoupled resources is integral to Kubernetes. Instead of an application using a dedicated port and socket,
for the life of the instance, the goal is for each component to be decoupled from other resources. The expectation and
software development toward separation allows for each component to be removed, replaced, or rebuilt.
- Instead of hard-coding a resource in an application, an intermediary, such as a Service, enables connection and reconnection
to other resources, providing flexibility. A single machine is no longer required to meet the application and user
needs; any number of systems could be brought together to meet the needs when, and, as long as, necessary.
- As Kubernetes grows, even more resources are being divided out, which allows for an easier deployment of resources.
Also, Kubernetes developers can optimize a particular function with fewer considerations of others objects.

#### 2. Transience
- Equally important is the expectation of transience. Each object should be developed with the expectation that other
components will die and be rebuilt. With any and all resources planned for transient relationships to others, we can
update versions or scale usage in an easy manner.
- An upgrade is perhaps not quite the correct term, as the existing application does not survive. Instead, a controller
terminates the container and deploys a new one to replace it, using a different version of the application or setting.
Typically, traditional applications were not written this way, opting toward long-term relationships for efficiency and ease
of use.


#### 3. Flexible Framework
- Like a school of fish, or pod of whales, multiple independent resources working together, but decoupled from each
other and without expectation of individual permanent relationship, gain flexibility, higher availability and easy scalability.
Instead of a monolithic Apache server, we can deploy a flexible number of nginx servers, each handling a small part of
the workload. The goal is the same, but the framework of the solution is distinct.
- A decoupled, flexible and transient application and framework is not the most efficient. In order for the Kubernetes
orchestration to work, we need a series of agents, otherwise known as controllers or watch-loops, to constantly monitor
the current cluster state and make changes until that state matches the declared configuration.
- The commoditization of hardware has enabled the use of many smaller servers to handle a larger workload, instead of
single, huge systems.


#### 4. Managing Resource Usage
- As with any application, an understanding of resource usage can be helpful for a successful deployment. Kubernetes
allows us to easily scale clusters, larger or smaller, to meet demand. An understanding of how the Kubernetes clusters
view the resources is an important consideration. The `kube-scheduler`, or a custom scheduler, uses the PodSpec to determine the best node for deployment.
- In addition to administrative tasks to grow or shrink the cluster or the number of Pods, there are autoscalers which
can add or remove nodes or pods, with plans for one which uses cgroup settings to limit CPU and memory usage by
individual containers.
- **By default, Pods use as much CPU and memory as the workload requires**, behaving and coexisting with other Linux
processes. Through the use of resource requests, the scheduler will only schedule a Pod to a node if resources exist to
meet all requests on that node. The scheduler takes these and several other factors into account when selecting a node
for use.
- Monitoring the resource usage cluster-wide is not an included feature of Kubernetes. Other projects, like `Prometheus`,
are used instead. In order to view resource consumption issues locally, use the `kubectl describe` pod command.
You may only know of issues after the pod has been terminated.


##### CPU
- CPU requests are made in CPU units, each unit being a millicore, using `mille` – the Latin word for thousand. Some
documentation uses millicore, others use millicpu, but both have the same meaning. Thus, a request for .7 of a CPU
would be 700 millicore. Should a container use more resources than allowed, it won’t be killed. The exact amount of
overuse is not definite. Note the notation often found in documentation. Each dot would represent a new line and indent
if converted to YAML.
 - `spec.containers[].resources.limits.cpu`
 - `spec.containers[].resources.requests.cpu`

- The value of CPUs is not relative. It does not matter how many exist, or if other Pods have requirements. One CPU, in
Kubernetes, is equivalent to:
- 1 AWS vCPU
- 1 GCP Core
- 1 Azure vCore
- 1 Hyperthread on a bare-metal Intel processor with Hyperthreading.

*Notes*: if `current` shows 900 millicore, it means we are currently using 90% of a processor. we can set a `limit` to 1500.
`request` is what is reserved, and held back from others

##### Memory (RAM)
- With Docker engine, the `limits.memory` value is converted to an integer value and becomes the value to the
`docker run --memory <value> <image>` command. The handling of a container which exceeds its memory limit
is not definite. It may be restarted, or, if it asks for more than the memory request setting, the entire Pod may be evicted
from the node.
 - `spec.containers[].resources.limits.memory`
 - `spec.containers[].resources.requests.memory`


##### Ephemeral Storage (Beta Feature in 1.14)
- Container files, logs, and EmptyDir storage, as well as Kubernetes cluster data, reside on the root filesystem of the host
node. As storage is a limited resource, you may need to manage it as other resources. The scheduler will only choose
a node with enough space to meet the sum of all the container requests. Should a particular container, or the sum of
the containers in a Pod, use more then the limit, the Pod will be evicted.
 - `spec.containers[].resources.limits.ephemeral-storage`
 - `spec.containers[].resources.requests.ephemeral-storage`


#### 5. Using Label Selectors
- `Labels` allow for objects to be selected which may not share other characteristics. For example if a developer were to
label their pods using their name they could affect all of their pods, regardless of the application or deployment the pods
were using.
- Labels are how operators, also known as watch-loops, track and manage objects. As a result if you were to hardcode
the same label for two objects they may be managed by different operators and cause issues. For example one
deployment may call for ten pods, while another with the same label calls for five. All the pods would be in a constant
state of restarting as each operator tries to start or stop pods until the status matches the spec.
- Consider the possible ways you may want to group your pods and other objects in production. Perhaps you may use
development and production labels to differentiate the state of the application. Perhaps add labels for the department,
team, and primary application developer.
- Selectors are namespace scoped. Use the `–all-namespaces` argument to select matching objects in all namespaces.
- The labels, annotations, name, and metadata of an object can be found near the top of `kubectl get ¡obj-name¿ -oyaml`. For example:
```
ckad1$ kubectl get pod examplepod-1234-vtlzd -o yaml
```


#### 6. Multi-Container Pods
- The idea of multiple containers in a Pod goes against the architectural idea of decoupling as much as possible. One
could run an entire operating system inside a container, but would lose much of the granular scalability Kubernetes
is capable of. But there are certain needs in which a second or third co-located container makes sense. By adding
a second container, each container can still be optimized and developed independently, and both can scale and be
repurposed to best meet the needs of the workload.


#### 7. Sidecar Container
- The idea for a sidecar container is to add some functionality not present in the main container. Rather than bloating code,
which may not be necessary in other deployments, adding a container to handle a function such as logging solves the
issue, while remaining decoupled and scalable. Prometheus monitoring and Fluentd logging leverage sidecar containers
to collect data.

#### 8. Adapter Container
- The basic purpose of an adapter container is to modify data, either on ingress or egress, to match some other need.
Perhaps, an existing enterprise-wide monitoring tools has particular data format needs. An adapter would be an efficient
way to standardize the output of the main container to be ingested by the monitoring tool, without having to modify the
monitor or the containerized application. An adapter container transforms multiple applications to singular view.

#### 9. Ambassador
- An ambassador allows for access to the outside world without having to implement a service or another entry in an
ingress controller:
 - Proxy local connection
 - Reverse proxy
 - Limits HTTP requests
 - Re-route from the main container to the outside world.
-  Ambassador (https://www.getambassador.io/ is an ”open source, Kubernetes-native API gateway for microservices built
on Enjoy”.

#### 10 Points to Ponder
- Is my application as decoupled as it could possibly be?
Is there anything that I could take out, or make its own container?
These questions, while essential, often require an examination of the application within the container. Optimal use of
Kubernetes is not typically found by containerization of a legacy application and deployment. Rather most applications
are rebuilt entirely, with a focus on decoupled micro-services.
When every container can survive any other containers regular termination, without the end-user noticing, you have
probably decoupled the application properly.
- Is each container **transient**, does it expect and have code to properly react when other containers are transient?
Could I run **Chaos Monkey** to kill ANY and multiple pods, and my end user would not notice?
Most Kubernetes deployments are not as transient as they should be, especially among legacy applications. The
developer hold on to a previous approach and does not create the containerized application such that every connection
and communication is transient and will be terminated. With code waiting for the service to connect to a replacement
Pod and the containers within.
Some will note that this is not efficient as it could be. This is correct. We are not optimizing and working against the
orchestration tool. Instead we are taking advantage of the decoupled and transient environment to scale and use only
the particular micro-service the end user needs.

- Can I scale any particular component to meet workload demand?
The minimization of an application, breaking it down into the smallest part possible often goes against what developers
have spent a career doing. Each division requires more code to handle errors and communication, and is less efficient
that a tightly coupled application.
Machine code is very efficient, for example, but not portable. Instead code is written in a higher level language, which
may not be interpreted to run as efficiently, but will run in varied environments. Perhaps approach code meant for
Kubernetes in the sense that you are making the highest, and most non-specific way possible. If you have broken down
each component then you can scale only the most necessary component. In this way the actual workload is more
efficient, as the only software consuming CPU cycles is that which the end-user requires. There would be minimal
application overhead and waste.
Ensure you understand how to leverage a service to handle networking. Every Pod is given a single IP address, shared
by all containers in a pod. The watch-loop for services will manipulate firewall rules to get traffic to pods with matching
labels, and interact with the network plugin, such as Calico.
- Have I used the most open standard stable enough to meet my needs?
This item can take a while to investigate, but usually a good investment. With tight production schedules some may
use what they know best, or what is easiest at the moment. Due to the fast changing nature, or CI/CD, of a production
environment this may lead to more work in the long run.


#### 11. Jobs
- Just as we may need to redesign our applications to be decoupled we may also consider that microservices may not
need to run all the time. The use of Jobs and CronJobs can further assist with implementing decoupled and transient
microservices.
- Jobs are part of the batch API group. They are used to run a set number of pods to completion. If a pod fails, it will be
restarted until the number of completion is reached.
- While they can be seen as a way to do batch processing in Kubernetes, they can also be used to run one-off pods. A
Job specification will have a parallelism and a completion key. If omitted, they will be set to one. If they are present, the
parallelism number will set the number of pods that can run concurrently, and the completion number will set how many
pods need to run successfully for the Job itself to be considered done. Several Job patterns can be implemented, like a
traditional work queue.
- Cronjobs work in a similar manner to Linux jobs, with the same time syntax. There are some cases where a job would
not be run during a time period or could run twice; as a result, the requested Pod should be idempotent.
- An option spec field is .spec.concurrencyPolicy which determines how to handle existing jobs, should the time
segment expire. If set to Allow, the default, another concurrent job will be run. If set to Forbid, the current job
continues and the new job is skipped. A value of Replace cancels the current job and starts a new job in its place.
