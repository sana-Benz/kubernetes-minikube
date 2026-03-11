# kubernetes-minikube

Minikube is a tool that lets you run Kubernetes locally. 
minikube runs a single-node Kubernetes cluster on your personal computer (including Windows, macOS and Linux PCs) so that you can try out Kubernetes, or for daily development work.
Minikube provides a kubernetes cluster composed of a single node that acts as both the control plane and the worker node at the same time.

## Docker installation

### installation for Mac, Windows 10 Pro, Enterprise, or Education

https://www.docker.com/get-started

Choose Docker Desktop

### installation for Windows home

https://docs.docker.com/docker-for-windows/install-windows-home/

## Kuberntes Minikube installation

https://minikube.sigs.k8s.io/docs/start/

Minikube provides a dashboard (web portal). Access the dashboard using the following command:

```
minikube dashboard
```
## Kubectl installation
kubectl is a command-line tool used to control Kubernetes. It allows you to create deployments, start pods, expose services, scale applications, and perform updates.

You can find the installation steps for kubectl for your operating system at the following link:
https://kubernetes.io/docs/tasks/tools/


## Download this project

This project contains a web service coded in Java, but the language doesn't matter. This project has already been built and the binary version is there:

First of all, download and uncompress the project: https://github.com/charroux/kubernetes-minikube

You can also use git: `git clone https://github.com/charroux/kubernetes-minikube`

Then move to the sud directory with `cd kubernetes-minikube/myservice` where a DockerFile is.

## Test this project using Docker

Build the docker image:
```
docker build -t myservice .
```

Check the image:
```
docker images
```

Start the container:
```
docker run -p 4000:8080 -t myservice
```

8080 is the port of the web service, while 4000 is the port for accessing the container. Test the web service using a web browser: http://localhost:4000 It displays hello.

Ctrl-C to stop the Web Service.

Check the containerID:
```
docker ps
```

Stop the container:
```
docker stop containerID
```

## Publish the image to the Docker Hub

Retreive the image ID:
```
docker images
```

Tag the docker image: 
```
docker tag imageID yourDockerHubName/imageName:version
```

Example: `docker tag 1dsd512s0d myDockerID/myservice:1`

Login to docker hub: 
```
docker login
```
or
```
docker login http://hub.docker.com
```
or 
```
docker login -u username -p password
```

Push the image to the docker hub:
```
docker push yourDockerHubName/imageName:version
```

Example: `docker push myDockerID/myservice:1`

## Create a kubernetes deployment from a Docker image

In this step, we deploy an application in Kubernetes using an existing Docker image hosted on Docker Hub.
Kubernetes will automatically download the image, create a pod, and run the container inside the cluster.

First, we verify that the Kubernetes cluster is running and that a node is available:
```
kubectl get nodes
```
Then, we create a deployment using a Docker image:
```
kubectl create deployment myservice --image=sanabns/myservice:1
```
When this command is executed, Kubernetes will: 
* pull the Docker image from Docker Hub
* create a pod
* run the container inside the pod
* ensure the application keeps running

The image used comes from the Docker hub: https://hub.docker.com/r/sanabns/myservice . But you can use your own image instead.

Check the pod that has been created :
```
kubectl get pods
```

Check if the state is running.

Get complete logs for a pods: 
```
kubectl describe pods
```

Retreive the IP address but notice that this IP address is ephemeral since a pods can be deleted and replaced by a new one.

Then retrieve the deployment in the minikube dashboard. 
Actually the Docker container is runnung inside a Kubernetes pods (look at the pod in the dashboard).
  
You can also enter inside the container in a interactive mode. To open an interactive shell inside the container, run:
```
kubectl exec -it podname -- /bin/bash
```

Replace podname with the name of the pod obtained using:
```
kubectl get pods
```

Once inside the container, you can explore the filesystem. For example:
```
ls
```

Don't forget to exit the container with:
```
exit
```

## Expose the Deployment through a service

In kubernetes, applications run inside pods. However, pods can be created, destroyed or replaced anytime. Because of this, the IP address of a pod is unstable and can change frequently.
 
To provide a stable way to access a group of pods, kubernetes introduces the concept of a service.

A Kubernetes Service is an abstraction which defines a logical set of Pods running somewhere in the cluster, 
that all provide the same functionality. 
When created, each Service is assigned a stable and unique IP address (also called clusterIP). 
This IP address :
* remains stable for the entire lifetime of a service
* allows other components in the cluster to reliably communicate with the application
* automatically load-balances requests across the Pods behind the Service.

Even if Pods are recreated or replaced, the SErvice continues to route traffic to the new pods.

## Expose HTTP and HTTPS routes from outside the cluster to services within the cluster

By default, Services are only accessible inside the kubernetes cluster. However, some parts of your application (for example, frontends or web API) need to be accessible from outside the cluster.

Kubernetes provides different Service types to control how the Service is exposed. The default one is ClusterIP.

Type values and their behaviors are:

* ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.
* NodePort: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting NodeIP:NodePort.
* LoadBalancer: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
* ExternalName: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record

## Expose HTTP and HTTPS route using NodePort

Run the following command to expose your Deployment using a NodePort Service :

```
kubectl expose deployment myservice --type=NodePort --port=8080
```
This command creates a Service that :
* targets the Pods of the myservice Deployment
* exposes port 8080
* assigns a NodePort automatically

You can verify the service with :
```
kubectl get services
```
You can verify the ports in the output. For example, 8080 is the Service port and 30205 is the NodePort chosen randomly by kubernetes.

Retrieve the service address with :
```
minikube service myservice --url
```
This format of this address is `http://NodeIP:NodePort`. 

Test this address inside your browser. It should display hello again.

Look from the NodeIP and the NodePort in the minikube dashboard.

## Scaling and load balancing

One of the main advantages of kubernetes is the ability to scale applications easily and distribute traffic automatically between instances.

In fact, we can run multiple replicas of the same application and kubernetes will ensure that they always stay running and automatically distribute incoming requests between them.

Check if the myservice deployment is running:

```
kubectl get deployments
```
At this stage, the Deployment should have 1 replica.

To see the Pods created by the Deployment:

```
kubectl get pods
```
Here you should only see one Pod, which means that only one instance of the application is running.

### Scale the application

Now we will increase the number of application instances.
Start a second instance:

```
kubectl scale --replicas=2 deployment/myservice
```
This command updates the Deployment configuration so that two replicas of the pod must be running.
Kubernetes will automatically create a second pod to match the desired state.

You can now check the deployment 
```
kubectl get deployments
```

and check the pods again

```
kubectl get pods
```

You should now be able to see two pods running.
In fact, if one Pod fails, kubernetes will automatically start a new one to maintain the desired number of replicas.

## Creating a Service of type LoadBalancer

In the previous section, the application was exposed using a NodePort Service. However, in real production environments, applications are usually exposed using a LoadBalancer Service.

A LoadBalancer Service automatically creates an external load balancer that distributes incoming traffic to the application pods.

Check if the myservice deployment is still running:

```
kubectl get deployments
```
Before creating a new Service of type LoadBalancer, we must ensure that there is no existing Service already exposing the Deployment.

List the services 
```
kubectl get services
```
Since a service already exists for myservice, we must delete it.

```
kubectl delete service serviceName
```
This removes the previous NodePort Service. Note that deleting a Service does not affect the Pods or the Deployment. It only removes the network access configuration.

Now create a new Service of type Load Balancer 
```
kubectl expose deployment myservice --type=LoadBalancer --port=8080
```
You can verify the newly created service with 
```
kubectl get services
```
You can retreive the new Service url with 
```
minikube service myservice --url
```
Test in your web browser.

## Rolling updates

When deploying applications in production, it is important to update applications without interrupting the service. Kubernetes provides a mechanism called Rolling Updates to acheive this.

A Rolling Update allows a Deployment to update its Pods gradually, replacing old verions of the application with new ones without downtime. 
So, instead of stopping all running Pods and starting new ones, kubernetes :
* Creates a new Pod with the updated version
* Waits until the new Pod is ready 
* Terminates one of the old Pods
* Repeats the process until all Pods are updated

This ensures that the application remains available during the update. At no point are all the Pods stopped simultaneously.


### Update the application image
For this part, we have created a new version (v2) of the Docker image on Docker Hub available here https://hub.docker.com/r/sanabns/myservice. We can now update the Deployment to use this new image.

To update the image of the application to version 2, we use the set image subcommand, followed by the deployment name and the new image version  following this structure :

```
kubectl set image deployment/DEPLOYMENT_NAME CONTAINER_NAME=IMAGE
```

```
kubectl set image deployment/myservice myservice=dockerHubId/my-image:v2
```

You can also confirm the update by running the rollout status subcommand:
```
kubectl rollout status deployments/myservice
```

To roll back the deployment to your last working version, use the rollout undo subcommand:
```
kubectl rollout undo deployments/myservice
```
This will restore the Deployment to its last working version automatically, again with no downtime.

## Create a deployment and a service using a yaml file

Yaml files can be used instead of using the command `kubectl create deployment` and `kubectl expose deployment`. This approach is versionable and easier to manage in production.

The required YAML files are located in the root of the project folder:

* myservice-deployment.yml → defines the Deployment for the application

* myservice-service.yml → defines a NodePort Service to expose the Deployment

* myservice-loadbalancing-service.yml → defines a LoadBalancer Service to expose the Deployment

Apply the deployment:
```
kubectl apply -f myservice-deployment.yml
```

Apply the node port service: 
```
kubectl apply -f myservice-service.yml
```

or 

Apply the service of type loadbalancer:
```
kubectl apply -f myservice-loadbalancing-service.yml
```
Then test if it works as expected.

# Routing rule to a service using Ingress

An Ingress in Kubernetes is a resource that allows you to expose your services to the outside world through a single entry point.
It is not a Service type, but acts like a reverse proxy that routes incoming HTTP/HTTPS requests to the appropriate Services in your cluster.
It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address.
An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting.

## Set up Ingress on Minikube with the NGINX Ingress Controller

Enable the NGINX Ingress controller: 

```
minikube addons enable ingress
```
Verify that the NGINX Ingress controller is running:
```
kubectl get pods -n ingress-nginx
```

Make sure you have a Deployment running and exposed as a NodePort (not LoadBalancer).
Test that your service works before configuring Ingress.

A yaml file for ingress is located in the root of the project folder.

```
kubectl apply -f ingress.yml
```

Check the Ingress resource:

```
kubectl get ingress
```
Example output 

```
NAME                 CLASS    HOSTS                  ADDRESS        PORTS   AGE

example-ingress      nginx   myservice.info         192.168.64.2   80      18m
```
* HOSTS: the domain name configured in Ingress (myservice.info)

* PORTS: the port used (HTTP 80)

* ADDRESS: may be empty in Minikube; you’ll use minikube ip instead

### Configure Your Hosts File
To access the service from your browser using the hostname:

On Linux: edit the `/etc/hosts` file and add at the bottom values for: 

IngressAddress myservice.info

Where address is given by:
```
minikube ip
```

On Mac: edit the `/etc/hosts` file and add at the bottom values for: 

127.0.0.1 myservice.info


On Windows : edit the `c:\windows\system32\drivers\etc\hosts` file, add 

`127.0.0.1 myservice.info`	

Then check in your Web browser: 

http://myservice.info/

### Enable a tunnel for Minikube:
For better host resolution and LoadBalancer support in Minikube:
```
minikube addons enable ingress-dns
```
```
minikube tunnel
```

The tunnel maps LoadBalancer and Ingress IPs to your local machine.

After this, http://myservice.info/ should work consistently.

### Add a second deployment and service:

If you want to route multiple applications through the same Ingress, you can create a second deployment and its service.

In this project, we created a second deployment with a NodePort service using these files:

* myservice2-deployment.yml

* myservice2-service.yml

Then, add a new path to the ingress.yml file to route traffic to the second service:

- path: /service2
  pathType: Prefix
  backend:
    service:
      name: myservice2
      port:
        number: 8080

This allows requests to http://myservice.info/service2 to reach your second application.

## Delete resources

To clean up your cluster, you can delete the services and deployments:
```
kubectl delete services myservice
```
```
kubectl delete deployment myservice
```
You can also delete the second deployment/service if needed:
```
kubectl delete services myservice2
```
```
kubectl delete deployment myservice2
```
