# K8s Lab 11

This lab is related to Chapter #11 - Logging and Troubleshooting - II (p258)

In addition to various logs files and command output, you can use journalctl to view logs from the node perspective.
We will view common locations of log files, then a command to view container logs. There are other logging options,
such as the use of a sidecar container dedicated to loading the logs of another container in a pod.
Whole cluster logging is not yet available with Kubernetes. Outside software is typically used, such as Fluentd, part of
http://fluentd.org/, which is another member project of CNCF.io, like Kubernetes.
