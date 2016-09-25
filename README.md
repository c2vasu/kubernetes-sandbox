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
