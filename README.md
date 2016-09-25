# Kubernetes Sandbox
Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.<br/>

Hello World node.js app into a replicated application running on Kubernetes.
<a href="http://kubernetes.io/docs/hellonode/">Know more..</a>

![alt tag](http://kubernetes.io/images/hellonode/image_1.png)

### Initial set up to install Docker, and Google Cloud SDK, use Google Cloud Shell
```
$ export PROJECT_ID="your-project-id"
# export PROJECT_ID="amiable-aquifer-144512"
$ curl -sSL https://sdk.cloud.google.com | bash
$ gcloud auth login
$ gcloud config set project <google-cloud-project-id>
# gcloud config set project amiable-aquifer-144512
$ gcloud components install kubectl
```
### Create Node.js application
hellonode/server.js
```
var http = require('http');
var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!');
};
var www = http.createServer(handleRequest);
www.listen(8081);
```
Run Node server
```
$ node hellonode/server.js
http://localhost:8081/
```
### Create a Docker container image
hellonode/Dockerfile
```
FROM node:6
EXPOSE 8081
COPY server.js .
CMD node server.js
```
Now build and run docker.
Tagging the image with the Google Container Registry repo for your $PROJECT_ID
```
$ docker build -t gcr.io/$PROJECT_ID/hello-node:v1 .
$ docker run -d -p 8081:8081 --name hello_tutorial gcr.io/$PROJECT_ID/hello-node:v1
$ curl "http://$(docker-machine ip YOUR-VM-MACHINE-NAME):8081"
# curl http://localhost:8081
# You can list the docker containers with:
$ docker ps
# Now stop the running container with
$ docker stop hello_tutorial
```
Now that the image works as intended and is all tagged with your $PROJECT_ID, we can push it to the Google Container Registry, a private repository for your Docker images accessible from every Google Cloud project (but also from outside Google Cloud Platform) :
```
$ gcloud docker push gcr.io/$PROJECT_ID/hello-node:v1
```
### Create Kubernetes Cluster
A cluster consists of a Master API server and a set of worker VMs called Nodes.
```
First, choose a Google Cloud Project zone to run service.
$ gcloud config set compute/zone us-central1-a

Create a cluster via the gcloud
$ gcloud container clusters create hello-world

Now deploy containerized application to the Kubernetes cluster.
$ gcloud container clusters get-credentials hello-world
```
### Create and manage pod
```
Create a Pod with the kubectl run command:
$ kubectl run hello-node --image=gcr.io/$PROJECT_ID/hello-node:v1 --port=8081

To view the Deployment we just created run:
$ kubectl get deployments
# NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
# hello-node   1         1         1            1           3m

To view the Pod created by the deployment run:
$ kubectl get pods
# NAME                         READY     STATUS             RESTARTS   AGE
# hello-node-478395812-fgi0v   0/1       ImagePullBackOff   0          3m

To view the stdout / stderr from a Pod run (probably empty currently):
$ kubectl logs <POD-NAME>

To view metadata about the cluster run:
$ kubectl cluster-info

To view cluster events run:
$ kubectl get events

To view the kubectl configuration run:
$ kubectl config view
```
### Allow external traffic
The pod to make it accessible outside Kubernetes cluster as Kubernetes Service.
The Kubernetes master creates the load balancer and related Compute Engine forwarding rules, target pools, and firewall rules to make the service fully accessible from outside of Google Cloud Platform.
```
$ kubectl expose deployment hello-node --type="LoadBalancer"
```
To find the ip addresses associated with the service run:
```
$ kubectl get services hello-node
# Note there are 2 IP addresses listed, both serving port 8080. CLUSTER_IP is only visible inside your cloud virtual network. EXTERNAL_IP is externally accessible.
```
### Scale up website
Deployment to manage a new number of replicas for pod
```
$ kubectl scale deployment hello-node --replicas=4
```
Here’s a diagram summarizing the state of our Kubernetes cluster:
![alt tag](http://kubernetes.io/images/hellonode/image_13.png)

### Roll out an upgrade to website
Kubernetes is here to help deploy a new version to production without impacting your users.

First, let’s modify the application.
```
response.end('Hello Kubernetes World!');
```
Build and publish a new container image to the registry with an incremented tag.
```
$ docker build -t gcr.io/$PROJECT_ID/hello-node:v2 .
$ gcloud docker push gcr.io/$PROJECT_ID/hello-node:v2
```
Kubernetes to smoothly update our deployment to the new version of the application.
Need to edit the existing <b>hello-node deployment</b> and change the image from gcr.io/$PROJECT_ID/hello-node:v1 to gcr.io/$PROJECT_ID/hello-node:v2
```
$ kubectl set image deployment/hello-node hello-node=gcr.io/$PROJECT_ID/hello-node:v2
```

### Cleaning it Up
```
Delete the Deployment (which also deletes the running pods) and Service (which also deletes external load balancer):
$ kubectl delete service,deployment hello-node

Delete cluster:
$ gcloud container clusters delete hello-world

To list the images created:
$ gsutil ls

And then to remove the all the images under this path, run:
$ gsutil rm -r gs://artifacts.$PROJECT_ID.appspot.com/
```
