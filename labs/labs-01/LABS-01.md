# LABS-01 - Kubernetes Concepts

[Main](../../README.md)


# Introduction

Now that we have a cluster up and running we can start exploring the Kubernetes CLI via the `kubectl` command.

`kubectl` interacts with the Kubernetes API Server, which runs on the master nodes in the cluster.

Kubernetes as a platform has a number of abstractions that map to API objects. These Kubernetes API Objects can be used to describe your cluster’s desired state - including information such as applications and workloads running, container images, networking resources, and more.

This section explains the most-used Kubernetes API concepts and how to interact with them via `kubectl`.

- [Display Nodes](#display-nodes)
- [Pods](#pods)
  * [Create a Pod](#create-a-Pod)
  * [Get the list of Pods](#get-the-list-of-pods)
  * [Get Pod details](#get-pod-details)
  * [Get Pod logs](#get-pod-logs)
  * [Execute shell on running pod](#execute-shell-on-running-pod)
  * [Delete-a-pod](#delete-a-pod)
- [Deployment](#deployment)
    + [Create a Deployment](#create-a-deployment)
    + [Scaling a Deployment](#scaling-a-deployment)
    + [Update a Deployment](#update-a-deployment)
    + [Deployment details](#deployment-details)
    + [Rollback a Deployment](#rollback-a-deployment)
    + [Delete a Deployment](#delete-a-deployment)
- [Service](#validate-the-deployment)
    * [Create a Deployment for Service](#create-a-deployment-for-service)
    * [Create a Service](#create-a-service)
    * [Publish a Service](#publish-a-service)
    * [Get the list of Services](#get-the-list-of-services)
    * [Get Service details](#get-service-details)
    * [Delete a Service](#delete-a-service)
- [Namespaces](#namespaces)
    * [Default namespace](#default-namespace)
    * [Custom namespace](#custom-namespace)
        + [Create a Namespace](#create-a-namespace)    
        + [Create a Deployment in namespace](#create-a-deployment)
        + [Get Deployments in the newly created Namespace](#get-deployments-in-the-newly-created-namespace)
        + [Get Deployments in all Namespaces](#get-deployments-in-all-namespaces)
- [Quota and Limits](#quota-and-limits) (Optional)
    * [Create ResourceQuota](#create-resourcequota)
    * [Scale resources with ResourceQuota](#scale-resources-with-resourcequota)
    * [Create resources with ResourceQuota](#create-resources-with-resourcequota)

# Display Nodes

This command will show all the nodes available in your kubernetes cluster:

    $ kubectl get nodes

        It will show an output similar to:

        NAME                                            STATUS    ROLES     AGE       VERSION
        ip-192-168-160-85.us-west-2.compute.internal    Ready     <none>    10m       v1.10.3
        ip-192-168-229-150.us-west-2.compute.internal   Ready     <none>    10m       v1.10.3
        ip-192-168-79-105.us-west-2.compute.internal    Ready     <none>    10m       v1.10.3

> If you do not see this output, or receive an error, please ensure that you’ve followed the steps [here](../102-your-first-cluster) and have a validated cluster.

# Pods

A Pod is the smallest deployable unit that can be created, scheduled, and managed. It’s a logical collection of containers that belong to an application. Pods are created in a namespace. All containers in a pod share the namespace, volumes and networking stack. This allows containers in the pod to "`find`" each other and communicate using `localhost`.

## Create a Pod

Each resource in Kubernetes can be defined using a configuration file. For example, an busybox pod can be defined with configuration file shown in below:

    $ cat pod.yaml

        apiVersion: v1
        kind: Pod
        metadata:
        creationTimestamp: null
        labels:
            run: busybox
        name: busybox
        spec:
        containers:
        - args:
            - bin/sh
            - -c
            - ls; sleep 3600
            image: busybox
            name: busybox1
            resources: {}
        - args:
            - bin/sh
            - -c
            - echo Hello world; sleep 3600
            image: busybox
            name: busybox2
            resources: {}
        dnsPolicy: ClusterFirst
        restartPolicy: Never
        status: {}

Create the pod as shown below:

    $ kubectl apply -f pod.yaml 
    
    pod/busybox created

## Get the list of Pods:

    $ kubectl get pods

    NAME        READY     STATUS    RESTARTS   AGE
    busybox     2/2       Running   0          22s

## Get Pod details:

Get additional details for the pod by using the `<pod-name>` from the above output:

    $ kubectl describe pod busybox 
    
    Name:         busybox
    Namespace:    default
    Priority:     0
    Node:         ip-10-0-1-137.us-west-2.compute.internal/10.0.1.137
    Start Time:   Sat, 12 Dec 2020 05:47:01 +0000
    Labels:       run=busybox
    Annotations:  cni.projectcalico.org/podIP: 192.168.178.184/32
                kubernetes.io/psp: vmware-system-tmc-privileged
    Status:       Running
    IP:           192.168.178.184
    IPs:
    IP:  192.168.178.184
    Containers:
    busybox1:
        Container ID:  containerd://7adfe13601df1e382bbff0a47251d319329aa7478cf6d7a896de4c7175a64510
        Image:         busybox
        Image ID:      docker.io/library/busybox@sha256:bde48e1751173b709090c2539fdf12d6ba64e88ec7a4301591227ce925f3c678
        Port:          <none>
        Host Port:     <none>
        Args:
        bin/sh
        -c
        ls; sleep 3600
        State:          Running
        Started:      Sat, 12 Dec 2020 05:47:04 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from default-token-8n5lf (ro)
    busybox2:
        Container ID:  containerd://0fed492c5326c1189eca1f466ea61cc8147040ebbcb9e23c4ea61893b12efef3
        Image:         busybox
        Image ID:      docker.io/library/busybox@sha256:bde48e1751173b709090c2539fdf12d6ba64e88ec7a4301591227ce925f3c678
        Port:          <none>
        Host Port:     <none>
        Args:
        bin/sh
        -c
        echo Hello world; sleep 3600
        State:          Running
        Started:      Sat, 12 Dec 2020 05:47:05 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from default-token-8n5lf (ro)
    Conditions:
    Type              Status
    Initialized       True 
    Ready             True 
    ContainersReady   True 
    PodScheduled      True 
    Volumes:
    default-token-8n5lf:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-8n5lf
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                    node.kubernetes.io/unreachable:NoExecute for 300s
    Events:
    Type    Reason     Age   From                                               Message
    ----    ------     ----  ----                                               -------
    Normal  Scheduled  71s   default-scheduler                                  Successfully assigned default/busybox to ip-10-0-1-137.us-west-2.compute.internal
    Normal  Pulling    70s   kubelet, ip-10-0-1-137.us-west-2.compute.internal  Pulling image "busybox"
    Normal  Pulled     68s   kubelet, ip-10-0-1-137.us-west-2.compute.internal  Successfully pulled image "busybox" in 2.164965826s
    Normal  Created    68s   kubelet, ip-10-0-1-137.us-west-2.compute.internal  Created container busybox1
    Normal  Started    68s   kubelet, ip-10-0-1-137.us-west-2.compute.internal  Started container busybox1
    Normal  Pulling    68s   kubelet, ip-10-0-1-137.us-west-2.compute.internal  Pulling image "busybox"
    Normal  Pulled     67s   kubelet, ip-10-0-1-137.us-west-2.compute.internal  Successfully pulled image "busybox" in 737.953549ms
    Normal  Created    67s   kubelet, ip-10-0-1-137.us-west-2.compute.internal  Created container busybox2
    Normal  Started    67s   kubelet, ip-10-0-1-137.us-west-2.compute.internal  Started container busybox2


> By default, pods are created in a `default` namespace. In addition, a `kube-system` namespace is also reserved for Kubernetes system pods. A list of all the pods in `kube-system` namespace can be displayed as shown:

    $ kubectl get pods --namespace=kube-system 

    NAME                                                              READY   STATUS    RESTARTS   AGE
    calico-kube-controllers-678cbd7fcf-khc5c                          1/1     Running   0          31h
    calico-node-ckglc                                                 1/1     Running   0          31h
    calico-node-khhd5                                                 1/1     Running   0          31h
    calico-node-kzckc                                                 1/1     Running   0          31h
    calico-node-vlx6b                                                 1/1     Running   0          31h
    coredns-5bcf65484d-nzmft                                          1/1     Running   0          31h
    coredns-5bcf65484d-p6dhc                                          1/1     Running   0          31h
    etcd-ip-10-0-1-55.us-west-2.compute.internal                      1/1     Running   0          31h
    kube-apiserver-ip-10-0-1-55.us-west-2.compute.internal            1/1     Running   0          31h
    kube-controller-manager-ip-10-0-1-55.us-west-2.compute.internal   1/1     Running   0          31h
    kube-proxy-26k9t                                                  1/1     Running   0          31h
    kube-proxy-8r8xg                                                  1/1     Running   0          31h
    kube-proxy-9ndn2                                                  1/1     Running   0          31h
    kube-proxy-lg2p8                                                  1/1     Running   0          31h
    kube-scheduler-ip-10-0-1-55.us-west-2.compute.internal            1/1     Running   0          31h

    Again, the exact output may vary but your results should look similar to these.

## Get Pod logs:

If the containers in the pod generate logs, then they can be seen using the command shown (a fresh nginx does not have logs - check again later once you have accessed the service):

    $ kubectl logs busybox -c busybox1

	bin
    dev
    etc
    home
    proc
    root
    sys
    tmp
    usr
    var

## Execute a shell on the running pod 

This command will open a TTY to a shell in your pod:

    $ kubectl exec -it busybox -c busybox1 /bin/sh

This opens a bash shell and allows you to look around the filesystem of the container.

## Delete a Pod

    $ kubectl delete pod/busybox

In the next sections, we will go into more detail about Pods, Deployments, and other commonly used Kubernetes objects.

# Deployment

A "`desired state`", such as 4 replicas of a pod, can be described in a Deployment object. The Deployment controller in Kubernetes cluster then ensures the desired and the actual state are matching. Deployment ensures the recreation of a pod when the worker node fails or reboots.

If a pod dies, then a new pod is started to ensure the desired vs actual matches. It also allows both up- and down-scaling the number of replicas. This is achieved using ReplicaSet. The Deployment manages the ReplicaSets and provides updates to those pods.

## Create a Deployment

The folowing example will create a Deployment with 3 replicas of NGINX base image. Let’s begin with the template:

    $ cat deployment.yaml

    apiVersion: apps/v1
    kind: Deployment # kubernetes object type
    metadata:
    name: nginx-deployment # deployment name
    spec:
    replicas: 3 # number of replicas
    selector:
        matchLabels:
        app: nginx
    template:
        metadata:
        labels:
            app: nginx # pod labels
        spec:
        containers:
        - name: nginx # container name
            image: nginx:1.12.1 # nginx image
            imagePullPolicy: IfNotPresent # if exists, will not pull new image
            ports: # container and host port assignments
            - containerPort: 80
            - containerPort: 443

This deployment will create 3 instances of NGINX image.

Run the following command to create Deployment:

    $ kubectl create -f deployment.yaml 

    deployment.apps/nginx-deployment created

To monitor deployment rollout status:

    $ kubectl rollout status deployment/nginx-deployment 
    
    deployment "nginx-deployment" successfully rolled out

> A Deployment creates a ReplicaSet to manage the number of replicas. Let’s take a look at existing deployments and replica set.

## Get the deployments:

    $ kubectl get deployments
    
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           76s

Get the list of running pods:

    $ kubectl get pods

    NAME                                READY   STATUS    RESTARTS   AGE
    busybox                             2/2     Running   0          15m
    nginx-deployment-66b6c48dd5-8bgmw   1/1     Running   0          104s
    nginx-deployment-66b6c48dd5-jkssc   1/1     Running   0          104s
    nginx-deployment-66b6c48dd5-jm29m   1/1     Running   0          104s

## Scaling a Deployment

Number of replicas for a Deployment can be scaled using the following command:

    $ kubectl scale --replicas=5 deployment/nginx-deployment 

    deployment "nginx-deployment" scaled

Verify the deployment:

    $ kubectl get deployments

    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   5/5     5            5           2m30s

Verify the pods in the deployment:

    $ kubectl get pods

    NAME                                READY   STATUS    RESTARTS   AGE
    busybox                             2/2     Running   0          16m
    nginx-deployment-66b6c48dd5-62lhd   1/1     Running   0          40s
    nginx-deployment-66b6c48dd5-8bgmw   1/1     Running   0          2m59s
    nginx-deployment-66b6c48dd5-jkssc   1/1     Running   0          2m59s
    nginx-deployment-66b6c48dd5-jm29m   1/1     Running   0          2m59s
    nginx-deployment-66b6c48dd5-wrlcs   1/1     Running   0          40s

## Update a Deployment

A more general update to Deployment can be made by making edits to the pod spec. In this example, let’s change to the latest nginx image.

First, type the following to open up a text editor:

    $ kubectl edit deployment/nginx-deployment

Next, change the image from `nginx:1.14.2` to `nginx:latest`.

This should perform a rolling update of the deployment. To track the deployment details such as revision, image version, and ports - type in the following:

## Deployment details

    $ kubectl describe deployments 

    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Sat, 12 Dec 2020 06:00:56 +0000
    Labels:                 app=nginx
    Annotations:            deployment.kubernetes.io/revision: 2
    Selector:               app=nginx
    Replicas:               5 desired | 3 updated | 7 total | 4 available | 3 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
    Labels:  app=nginx
    Containers:
    nginx:
        Image:        nginx:latest
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
    Volumes:        <none>
    Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    ReplicaSetUpdated
    OldReplicaSets:  nginx-deployment-66b6c48dd5 (4/4 replicas created)
    NewReplicaSet:   nginx-deployment-75b69bd684 (3/3 replicas created)
    Events:
    Type    Reason             Age    From                   Message
    ----    ------             ----   ----                   -------
    Normal  ScalingReplicaSet  7m15s  deployment-controller  Scaled up replica set nginx-deployment-66b6c48dd5 to 3
    Normal  ScalingReplicaSet  4m56s  deployment-controller  Scaled up replica set nginx-deployment-66b6c48dd5 to 5
    Normal  ScalingReplicaSet  6s     deployment-controller  Scaled up replica set nginx-deployment-75b69bd684 to 2
    Normal  ScalingReplicaSet  6s     deployment-controller  Scaled down replica set nginx-deployment-66b6c48dd5 to 4
    Normal  ScalingReplicaSet  5s     deployment-controller  Scaled up replica set nginx-deployment-75b69bd684 to 3
    
## Rollback a Deployment

To rollback to a previous version, first check the revision history:

    $ kubectl rollout history deployment/nginx-deployment 
    
    deployment.apps/nginx-deployment 
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>

If you only want to rollback to the previous revision, enter the following command:

    $ kubectl rollout undo deployment/nginx-deployment
    deployment.apps/nginx-deployment rolled back

In our case, the deployment will rollback to use the `nginx:1.14.2` image. Check the image name:

    $ kubectl describe deployments | grep Image
    
    Image:        nginx:1.14.2

>If rolling back to a specific revision then enter:

    $ kubectl rollout undo deployment/nginx-deployment --to-revision=<version>

## Delete a Deployment

Run the following command to delete the Deployment:

    $ kubectl delete -f deployment.yaml 
    
    deployment "nginx-deployment" deleted

# Service

A pod is ephemeral. Each pod is assigned a unique IP address. If a pod that belongs to a replication controller dies, then it is recreated and may be given a different IP address. Further, additional pods may be created using Deployment or Replica Set. This makes it difficult for an application server, such as WildFly, to access a database, such as MySQL, using its IP address.

A Service is an abstraction that defines a logical set of pods and a policy by which to access them. The IP address assigned to a service does not change over time, and thus can be relied upon by other pods. Typically, the pods belonging to a service are defined by a label selector. This is similar mechanism to how pods belong to a replica set.

This abstraction of selecting pods using labels enables a loose coupling. The number of pods in the deployment may scale up or down but the application server can continue to access the database using the service.

A Kubernetes service defines a logical set of pods and enables them to be accessed through microservices.

## Create a Deployment for Service

Pods belong to a service by using a loosely-coupled model where labels are attached to a pod and a service picks the pods by using those labels.

Let’s create a Deployment first that will create 3 replicas of a pod:

    $ cat echo-deployment.yaml
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: echo-deployment
          spec:
            replicas: 3
            selector:
              matchLabels:
                app: echo-pod
            template:
              metadata:
                labels:
                  app: echo-pod
              spec:
                containers:
                - name: echoheaders
                  image: k8s.gcr.io/echoserver:1.10
                  imagePullPolicy: IfNotPresent
                  ports:
                  - containerPort: 8080

This example creates an echo app that responds with HTTP headers from an Elastic Load Balancer.

Type the following to create the deployment:

    $ kubectl create -f echo-deployment.yaml

    deployment.apps/echo-deployment created

Use the `kubectl describe deployment` command to confirm `echo-app` has been deployed:

    $ kubectl describe deployment echo-deployment

    Name:                   echo-deployment
    Namespace:              default
    CreationTimestamp:      Sat, 12 Dec 2020 06:15:47 +0000
    Labels:                 <none>
    Annotations:            deployment.kubernetes.io/revision: 1
    Selector:               app=echo-pod
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
    Labels:  app=echo-pod
    Containers:
    echoheaders:
        Image:        k8s.gcr.io/echoserver:1.10
        Port:         8080/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
    Volumes:        <none>
    Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   echo-deployment-668cdc9776 (3/3 replicas created)
    Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  88s   deployment-controller  Scaled up replica set echo-deployment-668cdc9776 to 3


Get the list of pods:

    $ kubectl get pods 

    NAME                                READY   STATUS    RESTARTS   AGE
    busybox                             2/2     Running   0          31m
    echo-deployment-668cdc9776-j7r2l    1/1     Running   0          2m52s
    echo-deployment-668cdc9776-krfkp    1/1     Running   0          2m52s
    echo-deployment-668cdc9776-q4q2h    1/1     Running   0          2m52s

Check the label for a pod:

    $ kubectl describe pods/echo-deployment-3396249933-8slzp | grep Label 
    
    Labels: app=echo-pod  

> Each pod in this deployment has `app=echo-pod` label attached to it.

## Create a Service

In the following example, we create a service `echo-service`:

    $ cat service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: echo-service
    spec:
      selector:
        app: echo-pod
      ports:
      - name: http
        protocol: TCP
        port: 80
        targetPort: 8080

The set of pods targeted by the service are determined by the label `app: echo-pod` attached to them. It also defines an inbound port 80 to the target port of 8080 on the container.

Kubernetes supports both TCP and UDP protocols.

## Publish a Service

A service can be published to an external IP using the `type` attribute. This attribute can take one of the following values:

1.  `ClusterIP`: Service exposed on an IP address inside the cluster. This is the default behavior.

2.  `NodePort`: Service exposed on each Node’s IP address at a defined port.

3.  `LoadBalancer`: If deployed in the cloud, exposed externally using a cloud-specific load balancer.

4.  `ExternalName`: Service is attached to the `externalName` field. It is mapped to a CNAME with the value.

Let's create a clusterIP service and expose your services via the Istio Ingress gateway.

Run the following command to create the Service:

    $ kubectl create -f service.yaml

    service/echo-service created

## Get the list of Services

    $ kubectl get service

    NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
    echo-service   ClusterIP   10.102.198.216   <none>        80/TCP    46h
    kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP   3d7h

The Service is exposed to an IP address inside the cluster. To access it externally, define a Virtual Service to route it via the Istio ingress gateway. 

    $ kubectl create -f virtual-service-echo.yaml

    virtualservice.networking.istio.io/echo created

Now access the deployed service http://echo.workshop-01.frankcarta.com/

Now, the number of pods in the deployment can be scaled up and down. Or the pods may terminate and restart on a different host. But the service will still be able to target those pods because of the labels attached to the pod and used by the service.

## Delete a Service

Run the following command to delete the Service:

    $ kubectl delete -f service.yaml

The backend Deployment needs to be explicitly deleted as well:

    $ kubectl delete -f echo-deployment.yaml

# Namespaces

Namespaces allows a physical cluster to be shared by multiple teams. A namespace allows to partition created resources into a logically named group. Each namespace provides:

1.  A **unique scope** for resources to avoid name collisions

2.  **policies** to ensure appropriate authority to trusted users

3.  ability to specify **constraints for resource consumption**

This allows a Kubernetes cluster to share resources by multiple groups and provide different levels of QoS each group. Resources created in one namespace are hidden from other namespaces. Multiple namespaces can be created, each potentially with different constraints.

# Default namespace

By default, all resources in Kubernetes cluster are created in a `default` namespace.

`kube-public` is the namespace that is readable by all users, even those not authenticated. Any clusters booted with `kubeadm` will have a `cluster-info` ConfigMap. The clusters in this workshop are created using kops and so this ConfigMap will not exist.

`kube-system` is the namespace for objects created by the Kubernetes system.

Let’s create a Deployment:

    $ kubectl apply -f deployment.yaml

    deployment "nginx-deployment" created

Check its namespace:

    $ kubectl get deployment -o jsonpath={.items[].metadata.namespace}

    default

# Custom namespace

A new namespace can be created using a configuration file or `kubectl`.

# Create Namespace

    The following configuration file can be used to create Namespace:

        $ cat namespace.yaml

        kind: Namespace
        apiVersion: v1
        metadata:
          name: dev
          labels:
            name: dev

        $ kubectl apply -f namespace.yaml 
        
        namespace/dev created

    Alternatively, a namespace can be created using `kubectl` as well.

        $ kubectl create ns dev2
        
        namespace/dev2 created

# Get the list of Namespaces:

        $ kubectl get ns

        NAME          STATUS    AGE
        default       Active    3h
        dev           Active    12s
        kube-public   Active    3h
        kube-system   Active    3h

# Get more details about the Namespace:

    $ kubectl describe ns/dev 

    Name:         dev
    Labels:       name=dev
    Annotations:  Status:  Active

    No resource quota.

    No LimitRange resource.


# Create a Deployment in newly created Namespace

    $ cat deployment-namespace.yaml

	apiVersion: extensions/v1beta1
	kind: Deployment
	metadata:
	  name: nginx-deployment-ns
	  namespace: dev
	spec:
	  replicas: 3
	  selector:
	    matchLabels:
	      app: nginx
	  template:
	    metadata:
	      labels:
	        app: nginx
	    spec:
	      containers:
	      - name: nginx
	        image: nginx:1.12.1
	        ports:
	        - containerPort: 80
	        - containerPort: 443
    

The main change is the addition of `namespace: dev`.

Create the Deployment:

    $ kubectl apply -f deployment-namespace.yaml

    deployment.apps/nginx-deployment-ns created

# Query Deployment in a Namespace 

You can be queried resources in a namespace by providing an additional switch `-n` as shown:

    Get Deployments in the newly created Namespace

        $ kubectl get deployments -n dev

        NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
        nginx-deployment-ns   3/3     3            3           27s

    Get Deployments in all Namespaces:

        $ kubectl get deployments --all-namespaces
        
        NAMESPACE     NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
        default       nginx-deployment      3         3         3            3           1h
        dev           nginx-deployment-ns   3         3         3            3           1h
        dev2          nginx-deployment      3         3         3            3           1m
        kube-system   dns-controller        1         1         1            1           5h
        kube-system   kube-dns              2         2         2            2           5h
        kube-system   kube-dns-autoscaler   1         1         1            1           5h

## Quota and Limits

Each namespace can be assigned resource quota. Specifying quota allows to restrict how much of cluster resources can be consumed across all resources in a namespace. Resource quota can be defined by a ResourceQuota object. A presence of ResourceQuota object in a namespace ensures that resource quotas are enforced. T

A quota can be specified for compute resources such as CPU and memory, storage resources such as PersistentVolume and PersistentVolumeClaim and number of objects of a given type. A complete list of resources that can be restricted using ResourceQuota are listed at https://kubernetes.io/docs/concepts/policy/resource-quotas/.

### Create ResourceQuota

A ResourceQuota can be created using a configuration file or `kubectl`.

The following configuration file can be used to create ResourceQuota:

	$ cat resource-quota.yaml

	apiVersion: v1
	kind: ResourceQuota
	metadata:
	  name: quota
	spec:
	  hard:
	    cpu: "4"
	    memory: 6G
	    pods: "10"
	    replicationcontrollers: "3"
	    services: "5"
	    configmaps: "5"

This configuration file places the following requirements on the namespace:

1. Every new Container created must have a memory and CPU limit
2. Total number of Pods in this namespace cannot exceed 10
3. Total number of ReplicationController in this namespace cannot exceed 3
4. Total number of Service in this namespace cannot exceed 5
5. Total number of ConfigMap in this namespace cannot exceed 5

Create a new ResourceQuota:

	$ kubectl apply -f resource-quota.yaml
	
    resourcequota "quota" created

Alternatively, a ResourceQuota may be created using the `kubectl` CLI:

	kubectl create resourcequota quota2 --hard=cpu=10,memory=6G,pods=10,services=5,replicationcontrollers=3


In either this case, these restrictions would be placed on the `default` namespace in this case. An alternate namespace can be specified either in the configuration file or using the `--namespace` option on the `kubectl` CLI.

Get the list of ResourceQuota:

	$ kubectl get quota
	NAME      AGE
	quota     25s

Get more details about the ResourceQuota:

	$ kubectl describe quota/quota

	Name:                   quota
	Namespace:              default
	Resource                Used  Hard
	--------                ----  ----
	configmaps              0     5
	cpu                     300m  4
	memory                  0     6G
	pods                    3     10
	replicationcontrollers  0     3
	services                1     5

The output shows that three Pods and one Service already exists in the `default` namespace.

### Scale resources with ResourceQuota

Now that the ResourceQuota has been created, let's see how this impacts the new resources that are created or existing resources that are scaled.

We already have a Deployment `nginx-deployment`. Let's scale the number of replicas to exceed the assigned quota and see what happens.

Scale the number of replicas for the Deployment:

	$ kubectl scale --replicas=12 deployment/nginx-deployment
	deployment "nginx-deployment" scaled

The command output says that the Deployment is scaled.

Let's check if all the replicas are available:

	$ kubectl get deployment/nginx-deployment -o jsonpath={.status.availableReplicas}

	3

It shows only three replicas are available.

More details can be found:

	$ kubectl describe deployment nginx-deployment
        ...
        Conditions:
          Type             Status  Reason
          ----             ------  ------
          Progressing      True    NewReplicaSetAvailable
          Available        False   MinimumReplicasUnavailable
          ReplicaFailure   True    FailedCreate

The current reason is displayed in the output.

### Create resources with ResourceQuota

Let's create a Pod with the following configuration file:

	$ cat pod.yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  name: nginx-pod
	  labels:
	    name: nginx-pod
	spec:
	  containers:
	  - name: nginx
	    image: nginx:latest
	    ports:
	    - containerPort: 80

You may have to remove a previously running Pod or Deployment before attempting to create this Pod.

	$ kubectl apply -f pod.yaml
	Error from server (Forbidden): error when creating "pod.yaml": pods "nginx-pod" is forbidden: failed quota: quota: must specify memory

The error message indicates that a ResourceQuota is in effect, and that the Pod must explicitly specify memory resources.

Update the configuration file to:

	$ cat pod-cpu-memory.yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  name: nginx-pod
	  labels:
	    name: nginx-pod
	spec:
	  containers:
	  - name: nginx
	    image: nginx:latest
	    resources:
	      requests:
	        memory: "100m"
	    ports:
	    - containerPort: 80

There is an explicity memory resource defined here. Now, try to create the pod:

	$ kubectl apply -f pod-cpu-memory.yaml

	pod "nginx-pod" created

The Pod is successfully created.

Get more details about the Pod:

	$ kubectl get pod/nginx-pod -o jsonpath={.spec.containers[].resources}

	map[requests:map[cpu:1 memory:100m]

Get more details about the ResourceQuota:

	$ kubectl describe quota/quota
	Name:                   quota
	Namespace:              default
	Resource                Used  Hard
	--------                ----  ----
	configmaps              0     5
	cpu                     400m  4
	memory                  100m  6G
	pods                    4     12
	replicationcontrollers  0     3
	services                1     5

Note, how CPU and memory resources have incremented values.

> https://github.com/kubernetes/kubernetes/issues/55433[kubernetes#55433] provide more details on how an explicit CPU resource is not needed to create a Pod with ResourceQuota.

	$ kubectl delete quota/quota

	$ kubectl delete quota/quota2