---
title: Getting Started with K8S in Detail, Building a Cluster on a Local VM, and Practicing the Grey Release Feature
date: 2023-11-05 00:05:00
categories: 
  - Technology
tags: 
  - lightweight
  - Kubernetes
  - implementation
  - improve
  - development
  - VMs
  - clusters
  - machine
description: Minikube is a lightweight Kubernetes implementation that creates VMs and deploys simple clusters of just one node on your local machine.
cover: https://s2.loli.net/2023/11/05/bHOC5mxLtTPsp64.png
---
![](https://s2.loli.net/2023/11/05/gJD3OnLoejqkHVs.webp)

# introduction

## Using minikube

> Minikube is a lightweight Kubernetes implementation that creates VMs and deploys simple clusters of just one node on your local machine.

It is recommended to use minikube standalone to experience the basic functions first.， [kubernetes.io](https%3A%2F%2Fkubernetes.io%2Fzh-cn%2Fdocs%2Ftutorials%2Fhello-minikube%2F "https://kubernetes.io/zh-cn/docs/tutorials/hello-minikube/") ，You will still need to use minikube to compare and understand it when debugging later.

install  minikube empress

```ruby
minicube start
minikube dashboard
//Opening http://127.0.0.1:51200/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...

// 端口随机
```

For the dashboard initialized by minikube, there is no need to deal with permissions and login, so it can be used directly. If you build a bare metal dashboard, it will be extremely complicated to configure the permissions and logins for the dashboard.

![](https://s2.loli.net/2023/11/05/Vk1jwLHSJYBzpAa.webp)

The page is mainly about adding, deleting, and checking resources of various classes, which is helpful when you are not familiar with the commands.

![](https://s2.loli.net/2023/11/05/NhAH2R3rmS9YkqQ.webp)

## Prepare the docker image

Here we create an image of Aliyun for debugging.

```bash
// 两个版本，用于测试更新
registry.cn-hangzhou.aliyuncs.com/marquezyang/common:v1
registry.cn-hangzhou.aliyuncs.com/marquezyang/common:v2
```

Simple node service, port 8080, http output current node ip and hostname, v2 will show v2

```perl

index page / index page v2

IP lo10.244.0.158, hostname: test-k8s-5cc7cf6cf9-8d84m

```

## Deploying services

Create namespace for easy management and cleanup

```arduino
kubectl create namespace test
```

If the system is easy to install [kubectx](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fahmetb%2Fkubectx "https://github.com/ahmetb/kubectx") ， Can be installed after kubens test to switch namespace, not convenient to install the subsequent command plus -n test specify namespace for test.

Creating a yaml configuration file locally for kubectl apply -f file.yaml startup is equivalent to creating it on the command line.

appV1.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test-k8s

  name: test-k8s
spec:
  replicas: 3

  selector:
    matchLabels:
      app: test-k8s

  template:
    metadata:
      labels:
        app: test-k8s
    spec:

      containers:
        - name: test-k8s
          image: registry.cn-hangzhou.aliyuncs.com/marquezyang/common:v1

```

Bottom to top:

* A single `pod` is the smallest unit of a k8s deployment and contains one or more containers representing an application. For example, a wordpress docker deployment would have two containers, wordpress+mysql, for a single deployment.
* pod has metadata, which is used to give a selector to the parent abstraction collection, which is then clustered for operation
* replicas: 3 creates a `ReplicaSet` collection, which contains the same pods. In this case, it creates 3 identical pods, contained in a `ReplicaSet`.
* At the top, create a `Deployment` that points to the created `ReplicaSet`.

kubectl create

```bash
kubectl apply -f ./yaml/deploy/appv1.yaml -n test
```

Find the single `Deployment` in the dashboard, click on it and scroll down to find the `ReplicaSet` that it points to, and click on it and scroll down to find the 3 pods that were created.

![](https://s2.loli.net/2023/11/05/8FaqyVuKNpBAkU2.webp)

## Accessing the minikube network

minikube runs in docker and is network isolated. There are two ways to access the minikube network:

* minikube ssh, into the container bash
* minikube tunnel

Here we use minikube ssh to try to access a single pod and dashboard into the details of a particular pod. ![](https://s2.loli.net/2023/11/05/yk9r1QqINMAYRE2.webp)

After minikube ssh into the bash curl the ip address of the pod to access the individual pods. ![](https://s2.loli.net/2023/11/05/1ALgxW63rmPcF4C.webp)

## Creating a Service

> The Service API is an integral part of Kubernetes and is an abstraction that helps you expose collections of Pods on the network. Each Service object defines a logical collection of endpoints (typically these endpoints are Pods) and a policy for how to access those Pods.

Create service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: deploy-service
spec:
  selector:

    app: test-k8s

  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 31123

```

```bash
kubectl apply -f ./yaml/deploy/service.yaml -n test
kubectl get svc -n test
```

![](https://s2.loli.net/2023/11/05/GQVxNDHXrwObq7S.webp)

In minikube ssh, you can curl to the services exposed by servcie, and with **load balancing**, you can see that it is evenly distributed among the three pods 166, 167, and 168.![](https://s2.loli.net/2023/11/05/LKszRhuINUWyabx.webp)

You can also use minikube service to automatically open the page and browser access experience.

```js
minikube service deploy-service -n test
```

## Creating an Ingress Experience Grey Release

First, create a new deployment and service that uses the v2 img, and create a single file, appServiceV2.yaml.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test-k8s-v2

  name: test-k8s-v2
spec:
  replicas: 3

  selector:
    matchLabels:
      app: test-k8s-v2

  template:
    metadata:
      labels:
        app: test-k8s-v2
    spec:

      containers:
        - name: test-k8s-v2
          image: registry.cn-hangzhou.aliyuncs.com/marquezyang/common:v2

apiVersion: v1
kind: Service
metadata:
  name: test-k8s-v2
spec:
  selector:
    app: test-k8s-v2

  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32000
```

```bash
kubectl apply -f ./yaml/deploy/appServiceV2.yaml -n test
kubectl get svc -n test
```

At this point, there are two services, v1 and v2. ![](https://s2.loli.net/2023/11/05/TNsGmoOZ7zxBMaI.webp)

Test the v2 service

```js
minikube service test-k8s-v2 -n test

```

> In local experience, if you refresh the tab a few more times in your browser, you can see that it is hitting different IPs (pods) evenly, and the page shows v2.
>
> At this point, there are already two stable url load balancing to their respective pods. If you want to have a canary effect, where half of the page traffic is v1 and half is v2, you can do it with a local nginx. But k8s already provides a wrapper for this, called Ingress.
>
> > Ingress is an API object that manages external access to services in the cluster, typically through HTTP. Ingress can provide load balancing, SSL termination, and name-based virtual hosting.

[kubernetes.io/docs/tasks/...](https://link.juejin.cn?target=https%3A%2F%2Fkubernetes.io%2Fdocs%2Ftasks%2Faccess-application-cluster%2Fingress-minikube%2F "https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/")

first install ingress

```bash
minikube addons enable ingress
```

creater ingress1.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: k8s-test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: deploy-service
                port:
                  number: 8080

```

ingress2.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: k8s-test-v2-canary
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/canary: 'true'
    nginx.ingress.kubernetes.io/canary-weight: '50'
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: test-k8s-v2
                port:
                  number: 8080
```

```bash
kubectl apply -f ./yaml/deploy/ingress1.yaml -n test
kubectl apply -f ./yaml/deploy/ingress2.yaml -n test
kubectl get ingress -n test
```

![](https://s2.loli.net/2023/11/05/YSCMtH6Jrcw39zp.webp)

At this point, ADDRESS is minikube ip value 192.168.58.2 (docker's intranet address, not reachable locally), which means it has been successful. And ingress is 80, port 443 by default. After minikube ssh into bash, curl localhost (80) several times.

![](https://s2.loli.net/2023/11/05/XRIBzJlxhckweqy.webp)

**It can be seen that it hits v1,v2 evenly, and it also hits each IP (pod) evenly**. We have achieved the expected gray-scale release effect, and the production environment gray-scale release effect is also basically reproduced. (Here you can also minikube tunnel after the browser to visit localhost experience, pay attention to lift the local 80 port occupation.)

Finally clean up the site kubectl delete namespace test can be. If you haven't created a namespace before, it's not as convenient to clean it up. The performance of k8s is actually quite high, and my private cloud VM can't handle it at all, so I can shut it down in time.

## Bare metal setup

## Creating a Virtual Machine

I used the ESXi VM system from my private cloud and created three CentOS 7 VMs with at least 2c4g each. You can install them locally or consider renting a cluster from a cloud provider.

![](https://s2.loli.net/2023/11/05/hRTPSYGUfbO5xgV.webp)

## Creating a Cluster

Generally for the sake of understanding, multi-node bare-metal clusters are recommended to be built manually for the first time. But actually still use kubeadm init, is still mechanized copy command, in this network and system configuration is stuck in no sense, recommended to use a key script to build:

[github.com/lework/kain...](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Flework%2Fkainstall "https://github.com/lework/kainstall")

Go to the 192.168.31.153 terminal and execute the

```ini
export MASTER_NODES="192.168.31.153"
export WORKER_NODES="192.168.31.151,192.168.31.152"
export SSH_USER="root"
export SSH_PASSWORD="xxx"
export SSH_PORT="22"
export KUBE_VERSION="1.20.6"
bash kainstall-centos.sh init --version 1.24.8
```

> Kubernetes executes your [workloads] by placing containers in Pods running on [nodes](https://kubernetes.io/zh-cn/docs/concepts/workloads/) (Nodes) . A node can be a virtual machine or a physical machine, depending on the cluster configuration in which it resides. Each node contains a virtual machine running [Pod](https://link.juejin.cn?target=https%3A%2F%2Fkubernetes.io%2Fzh-cn%2Fdocs%2Fconcepts%2Fworkloads%2Fpods%2F "https:// kubernetes.io/zh-cn/docs/concepts/workloads/pods/"); these nodes are controlled by [control plane](https://link.juejin.cn?target=https%3A%2F%2Fkubernetes.io%2Fzh -cn%2Fdocs%2Freference%2Fglossary%2F%3Fall%3Dtrue%23term-control-plane "https://kubernetes.io/zh-cn/docs/reference/glossary/?all= true#term-control-plane") is responsible for management.

Above, we used minikube as a single node in the local docker. In fact, the definition of a node is consistent with common network terminology and can refer to a single machine. If there are two work nodes in the cluster and a deployment wants to create four pods, those four pods will be deployed evenly across the two nodes (machines).

When you're done building, check out the

```arduino
kubectl get nodes
```

![](https://s2.loli.net/2023/11/05/8vNsxVFuTaqnACY.webp)

The intranet IP is

```
192.168.31.153 k8s-master-node1
192.168.31.151 k8s-worker-node1
192.168.31.152 k8s-worker-node2
```

## Using dashboard

The script already has dashboard installed, but rbac is a bit tricky to configure.

```scss

yum install tmux
tmux

kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$' --port=8001
ctrl+b d

```

interviews [http://192.168.31.153:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login](https://link.juejin.cn?target=http%3A%2F%2F192.168.31.153%3A8001%2Fapi%2Fv1%2Fnamespaces%2Fkubernetes-dashboard%2Fservices%2Fhttps%3Akubernetes-dashboard%3A%2Fproxy%2F%23%2Flogin "http://192.168.31.153:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login")

I found that it requires a login and is restricted to https or localhost only. Here's how to get around it.

[kubernetes.io/zh-cn/docs/...](https://link.juejin.cn?target=https%3A%2F%2Fkubernetes.io%2Fzh-cn%2Fdocs%2Ftasks%2Faccess-application-cluster%2Fweb-ui-dashboard%2F "https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/")

Installation of the dashboard is usually done using a remote yaml such as

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Download this locally, e.g. dashboard.yaml, search for 'args', there is only one place, add two lines

```yaml
      containers:
        - name: kubernetes-dashboard
          image: kubernetesui/dashboard:v2.7.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              protocol: TCP
          args:
            - --auto-generate-certificates
            - --namespace=kubernetes-dashboard
            - --enable-skip-login
            - --disable-settings-authorizer
```

At this point, the login page can be skipped, but there are no data permissions once inside. You need to refer to this issue [[github.com/kubernetes/...](https:// github.com/kubernetes/dashboard/issues/4179#issuecomment-610078007)]. 

Create admin.yaml , copy the configuration in the comment above, kubectl apply -f , and then the unlogged dashboard is available.

![](https://s2.loli.net/2023/11/05/XRnds8LoP7heYZ4.webp)

## Deploying services

Reuse the yaml from the minikube example above to create the two deployments and services Note that at this point, the performance of the appliance is likely to be far less than that of the previous standalone appliance, so you can set the number of pods to be smaller.

![](https://s2.loli.net/2023/11/05/IJzRn1oStmEKdvM.webp)

As you can see, the pods pointed to by a single Service are on node1 , node2 , i.e. two different machines. The 153 terminal can directly connect to both service ip

![](https://s2.loli.net/2023/11/05/puULwIj1DQ6O5Ci.webp)

## Deploying Ingress

Reusing the same yaml as above, create a grayed-out Ingress, knowing that nginx is on node 192.168.31.151.

![](https://s2.loli.net/2023/11/05/RpFw1BIdPJS3s2g.webp)

But at this point, curl 192.168.31.151 can't connect, type in

```arduino
kubectl get service -n ingress-nginx
```

![](https://s2.loli.net/2023/11/05/sun6zShky8WdB1c.webp)

ingress-nginx doesn't have an external-ip, so I tested with Cluster-ip. Multiple curl 10.96.103.254

![](https://s2.loli.net/2023/11/05/Vgv3fzZWilrpbe5.webp)

**It can be seen that the hits are evenly distributed to v1 ,v2 and also to each IP (pod) **. And at this point, the pods are actually distributed across two virtual machines, as expected.

At this point Nginx on node1, you can test the node2 shutdown, at the same time in the master continue to curl , you can find that you can still access the deployment of node1 pod, that is, disaster recovery high availability. Turn node2 on again and the cluster is restored.

# Summary

We have yet to build more complex scenarios such as data persistence and deployment of stateful applications. However, after the above, we already know the concepts of `pod` , `deployment`, `service`, `ingress`, and `node` in k8s very well, and we also successfully built a cluster and experienced the gray-scale release function, so it can be said that we have completely unlocked the skill tree of k8s. In the future, the articles recommended by the system will become its nutrients, and continue to grow, and eventually grow into a big tree.

Writing the article itself is also a learning process, and I would like to ask the readers to point out any mistakes or omissions in the article. If this article is helpful to you, welcome to like the collection.

![](https://s2.loli.net/2023/11/05/bHOC5mxLtTPsp64.png)
