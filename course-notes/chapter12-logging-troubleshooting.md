# Chapter 12 Kubernetes - Logging and Troubleshooting II

Kubernetes relies on API calls and is sensitive to network issues. Standard Linux tools and processes are the best method for
troubleshooting your cluster. If a shell, such as `bash` is not available in an affected Pod, consider deploying another similar pod
with a shell, like `busybox`. DNS configuration files and tools like dig are a good place to start. For more difficult challenges
you may need to install other tools like tcpdump.
Large and diverse workloads can be difficult to track, so monitoring of usage is essential. Monitoring is about collecting key
metrics such as CPU, memory, and disk usage, and network bandwidth on your nodes, as well as monitoring key metrics in
your applications. These features are being been ingested into Kubernetes with the Metric Server, which a cut-down version
of the now deprecated Heapster. Once installed the Metrics Server exposes a standard API which can be consumed by other
agents such as autoscalers. Once installed this endpoint can be found here on the master server: /apis/metrics/k8s.io/
Logging activity across all the nodes is another feature not part of Kubernetes. Using `Fluentd` can be a useful data collector
for a unified logging layer. Having aggregated logs can help visualize the issue and provides the ability to search all logs.
A good place to start when local network troubleshooting does not expose the root cause. It can be downloaded from
http://www.fluentd.org
Another project from cncf.io combines logging, monitoring, and alerting called `Prometheus` can be found here https:
//prometheus.io. It provides a time-series database as well as integration with `Grafana` for visualization and dashboards.


We are going to review some of the basic `kubectl` commands that you can use to debug what is happening, and we will
walk you through the basic steps to be able to debug your containers, your pending containers, and also the systems
in Kubernetes.

#### 2 Troubleshooting Flow
Troubleshooting Steps
Errors from command line
- Pod logs and state of the Pod
- Use shell to troubleshoot Pod DNS and network
- Check node logs for errors. Enough resources
- RBAC, SELinux or AppArmor security settings
- API calls to and from controllers to `kube-apiserver`
- Enable `auditing`
- Inter-node network issues, DNS and firewall
- Master server controllers.
 - Control Pods in pending or error state
 - Errors in log files
 - Enough resources


The flow of troubleshooting should start with the obvious. If there are errors from the command line investigate them first. The
symptoms of the issue will probably determine the next step to check. Working from the application running inside a container
to the cluster as a whole may be a good idea. The application may have a shell you can use, for example:
```
$ kubectl create deploy busybox --image=busybox --command sleep 3600
$ kubectl exec -ti <busybox_pod> -- /bin/sh
```

If the Pod is running use `kubectl logs pod-name` to view standard out of the container. Without logs you may consider
deploying a sidecar container in the Pod to generate and handle logging. The next place to check is networking including
DNS, firewalls and general connectivity using standard Linux commands and tools.
Security settings can also be a challenge. `RBAC`, covered in the security chapter, provides mandatory or discretionary access
control in a granular manner. SELinux and AppArmor are also common issues, especially with network-centric applications.

A newer feature of Kubernetes is the ability to enable auditing for the `kube-apiserver`, which can allow a view into actions
after the API call has been accepted.
The issues found with a decoupled system like Kubernetes are similar to those of a traditional datacenter, plus the added
layers of Kubernetes controllers.


##### Ephemeral Containers
- Add a tool-filled container to a running pod
- Alpha with v1.16 release
```
kubectl debug buggypod --image debian --attach
```
A feature new to the 1.16 version is the ability to add a container to a running pod. This would allow a feature-filled container
to be added to an existing pod without having to terminate and re-create. Intermittent and difficult to determine problems may
take a while to reproduce, or not exist with the addition of another container.
As an Alpha stability feature, it may change or be removed at any time. As well they will not be restarted automatically, and
several resources such as ports or resources are not allowed.
These containers are added via the ephemeralcontainers handler via an API call, not via the podSpec. As a result the use
of kubectl edit is not possible.
You may be able to use the kubectl attach command to join an existing process within the container. This can be helpful
instead of kubectl exec which executes a new process. The functionality of the attached process depends entirely on what
you are attaching to.


#### 3 Basic Start Sequence
- `systemd` starts `kubelet.service`
 - Uses `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`
 - Uses `/var/lib/kubelet/config.yaml config file`
 - `staticPodPath` set to `/etc/kubernetes/manifests/`
- `kubelet` creates all pods from *.yaml in directory
 - kube-apiserver
 - etcd
 - kube-controller-manager
 - kube-scheduler
- `kube-controller-manager` control loops use etcd data to start rest


#### 4 Monitoring
- Enable the add-ons
- Metrics Server and API
- Prometheus

Monitoring is about collecting metrics from the infrastructure, as well as applications.
The long used and now deprecated Heapster has been replaced with an integrated `Metrics Server`. Once installed and
configured the server exposes a standard API which other agents can use to determine usage. It can also be configured to
expose custom metrics, which then could also be used by autoscalers to determine if an action should take place

`Prometheus` is part of the Cloud Native Computing Foundation (CNCF). As a Kubernetes plugin, it allows one to
scrape resource usage metrics from Kubernetes objects across the entire cluster. It also has several client libraries
which allow you to instrument your application code in order to collect application level metrics.

#### 5 Plugins

##### Using Krew
- Install the software using steps at https://krew.dev
```
$ kubectl krew help
krew is the kubectl plugin manager.
You can invoke krew through kubectl: "kubectl krew [command]..."
Usage:
 krew [command]
Available Commands:
 help Help about any command
 info Show information about a kubectl plugin
 install Install kubectl plugins
 list List installed kubectl plugins
 search Discover kubectl plugins
 uninstall Uninstall plugins
 update Update the local copy of the plugin index
 upgrade Upgrade installed plugins to newer versions
 version Show krew version and diagnostics
```


We have been using the `kubectl` command throughout the course. The basic commands can be used together in a more complex
manner extending what can be done. There are over seventy and growing plugins available to interact with Kubernetes
objects and components.
At the moment plugins cannot overwrite existing kubectl commands. Nor can it add sub-commands to existing commands.
Writing new plugins should take into account the command line runtime package and a Go library for plugin authors.
As a plugin the declaration of options such as namespace or container to use must come after the command.
```
$ kubectl sniff bigpod-abcd-123 -c mainapp -n accounting
```

Plugins can be distributed in many ways. The use of `krew` allows for cross-platform packaging and a helpful plugin index,
which makes finding new plugins easy.
More information can be found here: https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/

##### Managing Plugins
- Add paths to plugins to $PATH variable
- View current plugins with kubectl plugin list
- Find new plugins with kubectl krew search
- Installing using kubectl krew install new-plugin
- Once installed use as kubectl sub-command
- upgrade and uninstall also available


The help option explains basic operation. After installation ensure the $PATH includes the plugins. krew should allow easy
installation and use after that.
```
$ export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
$ kubectl krew search
```

```
$ kubectl krew install tail
Updated the local copy of plugin index.
Installing plugin: tail
Installed plugin: tail
\
 | Use this plugin:
 ....
 | | Usage:
 | |
 | | # match all pods
 | | $ kubectl tail
 | |
 | | # match pods in the 'frontend' namespace
 | | $ kubectl tail --ns staging
....
```


##### Sniffing Traffic With Wireshark
- Read installation output for basic usage
- Note some require extra packages and configuration.
- `sniff` requires `Wireshark` and ability to export graphical display
- Pass the command the pod and container to use
```
$ kubectl krew install sniff nginx-123456-abcd -c webcont
```

#### 6 logging
Logging Tools
- No cluster-wide logging in Kubernetes
- Often aggregated and digested by outside tools like ElasticSearch
- Fluentd
- Kibana

Logging, like monitoring, is a vast subject in IT. It has many tools that you can use as part of your arsenal.
Typically, logs are collected locally and aggregated before being ingested by a search engine and displayed via a dashboard
which can use the search syntax. While there are many software stacks that you can use for logging, the `Elasticsearch`,
`Logstash`, and `Kibana` Stack (ELK) has become quite common.
In Kubernetes, the `kubelet` writes container logs to local files (via the `Docker` logging driver). The command kubectl logs
allows you to retrieve these logs.
Cluster-wide, you can use `Fluentd` to aggregate logs: http://www.fluentd.org/. Check the cluster administration logging concepts
for a detailed description: https://kubernetes.io/docs/concepts/cluster-administration/logging/.
`Fluentd` is part of the `Cloud Native Computing Foundation (CNCF)` and, together with `Prometheus`, makes a nice
combination for monitoring and logging. You can find a detailed walk-through of running `Fluentd` on Kubernetes here:
https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana.

#### 7 Troubleshooting Resources
- General guidelines and instructions:
https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/
- Troubleshooting applications:
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application
- Troubleshooting clusters:
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster
- Debugging pods:
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller
- Debugging services:
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/
- Github Site for issue and bug tracking
https://github.com/kubernetes/kubernetes/issues
- Kubernetes Slack channel
kubernetes.slack.com
