# Docker + Kubernetes + Go experiment

In this experiment we setup a local Kubernetes cluster to run a hello-world go http server.

## Initial Setup

- Install [Docker](https://www.docker.com/get-started). Used to build the Go image and container.
- Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows). CLI used to interface with k8s (Kubernetes) cluster.
- Install [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/). Minikube is a locally running K8s cluster that is used for learning / testing.

`main.go` contains a basic http server written in Go that will return "Hello World" to any request, it serves on port 8081. Build the container for the application:

## Build Docker Image

`docker build -t go-test-app .`

test that the container is working correctly by running:

`docker run -it -p 8080:8081 go-test-app`

Then visit "localhost:8080" and ensure the hello world message is visible.

## Configure Minikube

Minikube is not an emulation of Kubernetes, it runs a single node cluster inside a virtual machine. As a result it takes some time to initialise.  Start it with:

`minikube start`

During its initialisation it automatically configures Kubectl to use the Minikube cluster. Once it has started check that you can access it using:

`minikube ssh`

Now connected to the machine you can view the docker containers available to using:

`docker images`

Notice that the Minikube VM does not have access to the `go-test-app` docker image built previously. In order to fix this we need to tell the local environment to reuse the Docker daemon inside the Minikube instance. Minikube has a set of bash commands that do this, to view them run:

`minikube docker-env`

to actually execute them need to evaluate them, using powershell:

`minikube docker-env | Invoke-Expression`

Now that Docker is using the Minikube Daemon build the docker image again, connect to the Minikube instance and confirm the images are present.

## Creating and Starting the Deployment

`kubectl run minikubegohello --image=go-test-app --port=8001`
`kubectl get pods`
`kubctl expose deployment minikubegohello --type=NodePort`
not running due to error image?...

References:
- [Setting up minikube for windows](https://medium.com/@maumribeiro/running-your-own-docker-images-in-minikube-for-windows-ea7383d931f6)

## Teardown

Remember to stop Minikube when you're done.

`minikube stop`
