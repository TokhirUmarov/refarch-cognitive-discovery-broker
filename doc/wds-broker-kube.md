# WDS Broker deployment as container on Kubernetes pods

The Watson Discovery Broker code is packaged as a container and deployed to Bluemix Kubernetes Service.

## Docker File
The docker file is using the node:alpine image from docker public repository. Alpine is the lightweight operating system so using the minimum set of resources, and it includes a nodejs and npm. The exposed port is 6010.
```
FROM node:alpine
MAINTAINER https://github.com/ibm-cloud-architecture/ - IBM
RUN mkdir -p /wds-broker
COPY . /wds-broker

WORKDIR /wds-broker
RUN cd /wds-broker
EXPOSE 6010
CMD node server/server
```

## Build
To build the image: (adapt the version number and the namespace)
` docker build -t registry.ng.bluemix.net/ibm_mls/case/wdsbroker:v0.0.1 .`

Then it can be run locally by using the command:
`docker run -d -p 6010:6010 -t case/wdsbroker`

## Deploy
Once built, the image is uploaded to the Bluemix private container registry `registry.ng.bluemix.net/<namespace>/<imagename>`  with commands like:

```
# connect to bluemix
$ bx login -a https://api.ng.bluemix.net -u <userid> -p <password> -a <account> -o <org> -s <space>

# Get the list of namespaces: in case you forgot them... by using the container registry CLI
$ bx cr namespaces

# Tag local image with a name including the url and namespace and add a version number
$ docker tag case/wdsbroker  registry.ng.bluemix.net/ibm_mls/casewdsbroker:v0.0.1

# Log the local Docker client in to IBM Bluemix Container Registry
$ bx cr login
# push local image to bluemix registry
$ docker push registry.ng.bluemix.net/ibm_mls/casewdsbroker:v0.0.1
```
Once the image is uploaded it is possible to build a Kubernetes Deployment and deploy the container to the pods.

## Work with the Kubernetes cluster

The created cluster is cyancomputecluster: to connect to it and get the KUBECFG variable set use the following:
```shell
$ bx cs cluster-config cyancomputecluster

Downloading cluster config for cyancomputecluster
OK
The configuration for cyancomputecluster was downloaded successfully. Export environment variables to start using Kubernetes.

# Then export the config to the env variable:
$ export KUBECONFIG=/Users/jeromeboyer/.bluemix/plugins/container-service/clusters/cyancomputecluster/kube-config-hou02-cyancomputecluster.yml

```

So now the `kubectl` CLI will work and be connected to the remote cluster.

```
# Get cluster information
$ kubectl cluster-info

# Get nodes information
$ kubectl get nodes

# Create a Deployment and run it on the pods
$ kubectl run casewdsbroker --image=registry.ng.bluemix.net/ibm_mls/casewdsbrokerv01 --port=6010

# List your current deployments
$ kubectl get deployments

# Get pod list
$ kubectl get pods

# Get workers IP address
$ bx cs workers cyancomputecluster

Listing cluster workers...
OK
ID                                                 Public IP         Private IP     Machine Type   State    Status   
kube-hou02-pa523a20b106d8494d9e379d587f3b807a-w1   184.172.242.164   10.47.87.155   free           normal   Ready  


# Get details for the pods, to view the current image version of the app, run
$ kubectl describe pods
```
This last command is very important as it will help you to do some version control.


To verify the logs generated by the application:
`kubectl logs $POD_NAME`

To make the container visible to public cloud, we need to create a Kubernetes Service:
```
$ kubectl get services

$ kubectl expose casewdsbroker --type="NodePort" --port 6010

$ export NODE_PORT=$(kubectl get services/casewdsbroker -o go-template='{{(index .spec.ports 0).nodePort}}')

```
So from the Worker public IP address and the node port the URL to access the Broker User interface looks like: `http://184.172.242.164:32571/wdsWeather`


## Continuous Deployment
For developers to continuously update their code, they use the *rolling upgrade** capability. Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones.

To update the image of the application to next version, use the `kubectl set image` command, followed by the deployment name and the new image version
```
$
```

## Kubernetes Compendum
See the note [here](https://github.com/ibm-cloud-architecture/refarch-cognitive/blob/master/doc/cyancluster.md#kubernetes-compendum)
