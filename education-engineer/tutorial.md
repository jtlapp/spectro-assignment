# Deploy Applications with Kind

Kubernetes is a popular orchestration platform for deploying applications, but it is complicated and takes time to master. This tutorial introduces the [`kind`](https://kind.sigs.k8s.io/) utility as a way to get started with Kubernetes.

`kind` deploys Kubernetes locally on your computer, doing so by executing applications in Docker containers. In fact, the name "kind" is a contraction for "Kubernetes in Docker." Work through this tutorial to learn how to use `kind` to create a local cluster and to deploy a simple "Hello World" web application on this cluster.

## Prerequisites

This tutorial makes the following assumptions:

- You have Docker installed on your computer. For installation assistance, follow the [docker installation instructions](https://docs.docker.com/engine/install/).
- You have `kind` installed. For installation assistance, follow the [`kind` installation instructions](https://kind.sigs.k8s.io/docs/user/quick-start).

## Create a Cluster

The first step is to create a local Kubernetes cluster. This cluster will run locally as a Docker container. Execute the following command and wait for setup to complete:

```shell
kind create cluster
```

You should see the following output:

```
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

Now execute the following command to check for connectivity with Kubernetes:

```
kubectl cluster-info --context kind-kind
```

The output should look like this:

```
Kubernetes control plane is running at https://127.0.0.1:61727
CoreDNS is running at https://127.0.0.1:61727/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use `kubectl cluster-info dump`.
```

The Kubernetes cluster is now installed and running, and it is ready to receive your application.

## Deploy an Application

The next step is to load an application into the cluster. You'll be loading a sample "Hello World" app from an online location (`http://gcr.io/google-samples/hello-app:1.0`).

Create a file named **app.yaml** and insert the following configuration:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
        - image: gcr.io/google-samples/hello-app:1.0
          name: hello-app
          resources: {}
status: {}
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  ports:
    - port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    app: web
  type: NodePort
status:
  loadBalancer: {}
```

The configuration file contains both a deployment configuration and a service configuration. The deployment configuration tells Kubernetes how to deploy instances of the application across the cluster. The service configuration defines a service on the cluster as a whole and ties that service to the application.

Notice the following details about these configurations:

- The container image is the aforementioned "Hello World" application.
- The name of the application is `web`.
- The name of the service is also `web`.
- The node receives requests on port 8080.
- The application receives requests on port 8080 (the value of `targetPort`).

To deploy the application, execute the following command:

```shell
kubectl apply -f app.yaml
```

The application is now available to the cluster, but it is not yet accessible outside the cluster, such as via a browser. The final step is to forward a local port to the cluster.

## Forward a Port

To complete the setup, you need to establish a port on `localhost` that forwards to the node in the cluster. Accomplishing this requires both the name of the container and the port of the `web` service. Store these values in environment variables via the following commands:

```shell
CONTAINER_NAME=$(kubectl get pods --template '{{range .items}}{{.metadata.name}}{{end}}' --selector=app=web)

SERVICE_PORT=$(kubectl get service web -o jsonpath='{.spec.ports[0].nodePort}')
```

You can now forward a local port to the `web` service on the cluster. Execute the following command to start the forwarding service:

```shell
kubectl port-forward $CONTAINER_NAME $SERVICE_PORT:8080
```

Take note of the output of this command. The output contain the port you need to use to access the service. For example, given the following output, you will find the service at `http://127.0.0.1:31095` (equivalent to `http://localhost:31095`):

```
Forwarding from 127.0.0.1:31095 -> 8080
Forwarding from [::1]:31095 -> 8080
```

The `kubectl port-forward` command blocks the shell for the duration of the forwarding service. Press Control-C from within the shell to terminate the command.

## Use the Service

The application is now set up and running on the indicated port. If you open a browser and visit `localhost` at this port (e.g. at `http://localhost:31095`), you should see a page similar to the following:

```
Hello, world!
Version: 1.0.0
Hostname: web-548f6458b5-jwvsq
```

## Cleanup

This tutorial created a Kubernetes cluster running in Docker and executed a blocking port-forwarding command. When you're done with the tutorial, clean up as follows:

- To terminate the port-forwarding command, press Control-C from within the shell that is running the command.
  -To terminate the Kubernetes cluster and remove it from Docker, execute the command `kind delete cluster`.

# Next Steps

Congratulations! You are now familiar with the Kubernetes `kind` command. You've created a local cluster with the command, loaded an application into the cluster, and exposed a port on the cluster via port forwarding. While this gave you some practice with Kubernetes, you will also find `kind` command useful for locally testing Kubernetes clusters. To use `kind` for local testing, you would instead configure the cluster for your own application.

Here are some resources for learning more about Kubernetes, more about `kind`, and more about locally configuring a cluster for your own application:

- [Package an application in a Docker image](https://docs.docker.com/build/building/packaging/)
- [Create a cluster with `kind`](https://kind.sigs.k8s.io/docs/user/quick-start/#creating-a-cluster)
- [Learn more about Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
