# Welcome to workshop on K8s, TAS4k8s, TBS, ISTIO (Service Mesh)

## Prerequisites  

You need the following tools on your system to be able to run the workshop:
1. Docker 19.03.x or higher  
2. Git 2.24.x  
3. Make (Optional)
4. Visual Studio Code (Optional)

### Workshop Environment Dependency

1. Kubernetes cluster(kubeconfig file to connect to the cluster you intend to use for this workshop)
2. Harbor repository access.

## Building and Running the Workshop Container

### Build Container
`make build`

### Rebuild Container
`make rebuild`

### Start and exec to the container
`make run`

### Join Running Container
`make join`

### Start an already built Local Management Container
`make start`

### Stop a running Local Management Container
`make stop`

## LABS
1. [LABS-01 - Kubernetes Concepts](labs/labs-01/LABS-01.md)

2. [LABS-02 - TAS4k8s Setup](labs/labs-02/LABS-02.md)

3. [LABS-03 - Tanzu Build Service/Kpack](labs/labs-03/LABS-03.md)

4. [LABS-04 - Istio Service Mesh - Setup BookInfo Application](labs/labs-04/LABS-04.md)
