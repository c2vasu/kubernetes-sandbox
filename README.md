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
```
