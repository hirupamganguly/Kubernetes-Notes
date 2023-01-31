# Kubernetes-Notes

## PODS :- 

![1pod](https://user-images.githubusercontent.com/77056200/215839019-ead24552-dbac-4d37-ab8e-c024be1643b1.png)

Containers of Docker are wrapped withing Pods. Or in other words One Microservice and one helper service of that Microservice are wraaped within one POD.

Kubernetes is responsible for Managing of these PODs.

Yaml file look like : first-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: webapp
    image: docker-iamges-folder/k8s-angular:release0
```
``` bash
kubectl apply -f first-pod.yaml

kubectl get all

kubectl describe pod webapp

kubectl exec webapp ls

kubectl -it exec webapp sh
```
PODS are not visible outside of the cluster. So the server can't be accessible from Browser.

PODs are regularly die/re-created, so pods are short lived components.

## SERVICE :- 

Service are Long running components of kubernetes by which Outside world/Browser can communicate. It has Stable IP Address.

Service use Selector which help to choose POD. 
Via Labels of Pod, Service Spawn PODs.

Yaml file look like : first-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp #mapper of service
    release: "0" #mapper of service
spec:
  containers:
  - name: webapp
    image: docker-iamges-folder/k8s-angular:release0

---
apiVersion: v1
kind: Pod
metadata:
  name: webapp-release-0-5
  labels:
    app: webapp #mapper of service
    release: "0-5" #mapper of service
spec:
  containers:
  - name: webapp
    image: docker-iamges-folder/k8s-angular:release0-5
```
Yaml file look like : webapp-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: angular-webapp # VVI - service name

spec:
  # This defines which pods are going to be represented by this Service

  selector:
    app: webapp #mapper of pod
    release: "0-5" #mapper of pod

  ports:
    - name: http
      port: 80
      targetPort: 9376
      nodePort: 30080 # greater than 30,000 for MiniCube only

  type: NodePort
```

A Service can map any incoming port to a targetPort. By default and for convenience, the targetPort is set to the same value as the port field.


Type values and their behaviors are:

ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default that is used if you don't explicitly specify a type for a Service. You can expose the service to the public with an Ingress or the Gateway API.
    
NodePort: Exposes the Service on each Node's IP at a static port (the NodePort). To make the node port available, Kubernetes sets up a cluster IP address, the same as if you had requested a Service of type: ClusterIP.
    
LoadBalancer: Exposes the Service externally using a cloud provider's load balancer.

ExternalName: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up.

```bash
kubectl describe service webapp

kubectl get pod --show-labels
```

### Zero DownTime
For minimum down time we create new pod without delete old one, then we just change release label which will be pointing to the new pod, of service and then apply.

## REPLICASET :- 

As PODs are short lived, so when a POD die our WebService/WebSite goes down. So we want to run multiple PODs at same time so If one POD die others will be alive and serve the service. And also we want from Kubernetes that when a POD die Kubernetes will re spawn that POD via Specify How many Replicas we want to active for all the time.

Yaml file look like : pod.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp
spec:
  selector:
    matchLabels:
      app: webapp
  replicas: 2 # 2 pods are always running
  template: # template for the pods
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: docker-iamges-folder/k8s-angular:release0-5

---
apiVersion: v1
kind: Pod
metadata:
  name: queue
  labels:
    app: queue
spec:
  containers:
  - name: queue
    image: docker-iamges-folder/k8s-queue:release1
```
Yaml file look like : service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: angular-webapp

spec:
  selector:
    app: webapp

  ports:
    - name: http
      port: 80
      nodePort: 30080

  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: apache-queue

spec:
  selector:
    app: queue

  ports:
    - name: http
      port: 8161
      nodePort: 30010

  type: NodePort
```