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
Deployments are the central meaphor for "apps" / "services" and are described as a collection of resources and References.

Use Kubectl to deploy the image.

`kubectl run minikubegohello --image=go-test-app --port=8001 --image-pull-policy=Never`

expose the service to external IP addresses. NodePort allows us to have an external IP address connected to the port on the container we just deployed.

`kubectl expose deployment minikubegohello --type=NodePort`

Check that the image is running correctly by querying the pods:

`kubectl get pods`

Get the url of the running service:

`minikube service minikubegohello --url`

and curl it to ensure the service is externally accessible.

## Deploying from file definition
The `deployment.yaml` file describes the resources required for the deployment that we took manually in the previous stage and can be deployed using:

`kubectl apply -f ./deployment.yaml`

## Teardown

Remember to delete the service and stop Minikube when you're done.

`kubectl delete deployment minikubegohello`

`minikube stop`

## GCloud deployment

To allow a remote Kubernetes to deploy docker files it must be connected to an image repository. One such commonly used repository is the Google Cloud Container Registry. In order to complete this you must have an active Cloud project with the container registery API enabled.

Configure docker to use the gcloud cli to auth requests to Container Registery.

`gcloud auth configure-docker`

Tag the docker image with a registry name, this configures the `docker push` command to push the image to the correct location

`docker tag go-test-app gcr.io/<project-id>/go-test-app:latest`

Push the image to the Container Registry

`docker push gcr.io/<project-id>/go-test-app:latest`

verify the image arrived at the host location by visiting

`http://gcr.io/<project-id>/go-test-app`

# Useful Commands

`kubectl exec -it pod-name bash` - executes the bash command on named services.

`kubectl port-forward <pod-name> <local-port:remote-port>` - Forwards local port to specified pod port.

`kubctl attach <pod-name> -c <container-name>` - Attaches to a process that is running in an existing container.


# Glossary
- A **Pod** is a Kubernetes abstraction that represents a group of one or more application containers. Containers on a pod are tightly coupled and share IP/Port space are always co-located and co-scheduled
- A **Node** is a physical or virtual machine on which the pods are deployed. Each Node has a `kublet` service which is responsible for communication between the master and the node and a `kube-proxy` service which handles port forwarding on the node.
- The **Master Node** Orchestrates the state of the cluster.

# References:
- [Setting up minikube for windows](https://medium.com/@maumribeiro/running-your-own-docker-images-in-minikube-for-windows-ea7383d931f6)
