# K8s Lab 10

This lab is related to Chapter #10 - Exposing Applications (p.219)


#### 1: Exposing Applications: Expose a Service
In this lab we will explore various ways to expose an application to other pods and outside the cluster. We will add to
the NodePort used in previous labs other service options.
1. We will begin by using the default service type ClusterIP. This is a cluster internal IP, only reachable from within the
cluster. Begin by viewing the existing services.
```
student@master:˜$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    3d
nginx        ClusterIP   10.98.38.243    <none>        443/TCP    2d4h
registry     ClusterIP   10.104.27.122   <none>        5000/TCP   2d4h
```

2. Create a YAML file for a replacement service, which would be persistent. Use the label to select the secondapp. Expose
the same port and protocol of the previous service.
```
student@master:˜/app2$ vim service.yaml
apiVersion: v1
kind: Service
metadata:
  name: secondapp
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  type: NodePort
  selector:
    example: second
```

3. Create the service, find the new IP and port. Note there is no high number port as this is internal access only.
```
student@master:˜/app2$ kubectl create -f service.yaml
service/secondapp created
student@master:˜/app2$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        3d
nginx        ClusterIP   10.98.38.243    <none>        443/TCP        2d4h
registry     ClusterIP   10.104.27.122   <none>        5000/TCP       2d4h
secondapp    NodePort    10.104.56.56    <none>        80:31373/TCP   6s
```

4. Test access. You should see the default welcome page again.
```
student@master:˜/app2$ curl http://10.104.56.56
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<output_omitted>
```

5. To expose a port to outside the cluster we will create a NodePort. We had done this in a previous step from the
command line. When we create a NodePort it will create a new ClusterIP automatically. Edit the YAML file again. Add
type: NodePort. Also add the high-port to match an open port in the firewall as mentioned in the previous chapter. You’ll
have to delete and re-create as the existing IP is immutable, but not able to be reused. The NodePort will try to create a
new ClusterIP instead.
```
student@master:˜/app2$ vim service.yaml
...
   protocol: TCP
   nodePort: 32000 #<-- Add this and following line
  type: NodePort
  selector:
   example: second
...
```

```
student@master:˜/app2$ kubectl delete svc secondapp ; kubectl create -f service.yaml
service "secondapp" deleted
service/secondapp created
```

6. Find the new ClusterIP and ports for the service.
```
student@master:˜/app2$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        3d
nginx        ClusterIP   10.98.38.243    <none>        443/TCP        2d4h
registry     ClusterIP   10.104.27.122   <none>        5000/TCP       2d4h
secondapp    NodePort    10.96.67.80     <none>        80:32000/TCP   8s
```

8. Test the low port number using the new ClusterIP for the secondapp service.
```
student@master:˜/app2$ curl 10.96.67.80
<!DOCTYPE html>
<html>
<head>
...
```

9. Test access from an external node to the host IP and the high container port. Your IP and port will be different. It should
work, even with the network policy in place, as the traffic is arriving via a 192.168.0.0 port.
```
user@laptop:˜/Desktop$ curl http://35.184.219.5:32000
<!DOCTYPE html>
<html>
<head>
...
```

10. The use of a LoadBalancer makes an asynchronous request to an external provider for a load balancer if one is available.
It then creates a NodePort and waits for a response including the external IP. The local NodePort will work even before
the load balancer replies. Edit the YAML file and change the type to be LoadBalancer.
```
student@master:˜/app2$ vim service.yaml
....
- port: 80
  protocol: TCP
  nodePort: 32000
type: LoadBalancer #<-- Edit this line
selector:
  example: second
...
```

```
student@master:˜/app2$ kubectl delete svc secondapp ; kubectl create -f service.yaml
service "secondapp" deleted
service/secondapp created
```

11. As mentioned the cloud provider is not configured to provide a load balancer; the External-IP will remain in pending
state. Some issues have been found using this with VirtualBox.
```
student@master:˜/app2$ kubectl get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        3d
nginx        ClusterIP      10.98.38.243    <none>        443/TCP        2d4h
registry     ClusterIP      10.104.27.122   <none>        5000/TCP       2d4h
secondapp    LoadBalancer   10.99.35.38     <pending>     80:32000/TCP   7s
```
12. Test again local and from a remote node. The IP addresses and ports will be different on your node.
```
serewic@laptop:˜/Desktop$ curl http://35.184.219.5:32000
<!DOCTYPE html>
<html>
<head>
...
```

Remainder of exercise(s) is available in PDF

#### Exercise 11.2: Ingress Controller
If you have a large number of services to expose outside of the cluster, or to expose a low-number port on the host
node you can deploy an ingress controller. While nginx and GCE have controllers officially supported by Kubernetes.io,
the Traefik Ingress Controller is easier to install. At the moment
...
