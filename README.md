# Demo haxx presentation
This operator defined in this repo creates a powerpoint presentation based on markdown.
The idea is to have a easy way to generate slides by using markdown.
For this we use remarkjs, which is a library that supports this functionality (https://remarkjs.com/).

## Prerequisites
In order to test the end result, you only need:
- kubernetes cluster (either local or remote)
- kubectl installed locally

In order to develop with this repository, you should have installed the following dependencies:
- golang (https://golang.org/doc/install)
- operator-sdk (https://sdk.operatorframework.io/docs/installation/install-operator-sdk/)
- kubernetes cluster (f.e. https://microk8s.io/)

## Getting started
Deploy the haxx-presentation operator by using:
`kubectl apply -k config/default/`

Deploy a presentation resource with markdown:
`kubectl apply -f example-presentation.yaml`

Inspect your presentation as follows:
`kubectl port-forward example-presentation-pod 8082:80`

Now you can browse to localhost:8082 to view your presentation

## Operator code
The rest of the code in this repository implements the operator functionality.
The operator is created using the operator-sdk (https://sdk.operatorframework.io/)

The scaffolding was created through following commands:
`operator-sdk init --project-name haxx-presentation-operator --domain axxes.com --owner niels.claeys@axxes.com
operator-sdk create api --kind Presentation --group haxx --version v1 --resource=true --controller=true`

Note: these commands were executed in the $GOPATH/src directory for simplicity.

I only made changes to:
- `api/v1/presentation_types.go` to specify the custom resource Presentation with a markdown property
- `controllers/presentation_controller.go` to implement the controller logic

### markdown to presentation
The docker container that receives markdown and generates a ppt, can be found in the haxx-presentation-container directory.
It is a simple nginx container, with 2 templates (before, after) that configure remarkjs. 
I made some tweaks to the css, such that it is in line with the specification of haxx.
The content of the slides should be passed in by mounting a file to the config/slides.md directory.

To test this, you can use the following command 
`docker run -p 8082:80 -v <PATH>/slides.md:/config/slides.md -it nilli9990/haxx-presentation:1.0`

### Controller imlementation
The controller receives as input that the Presenation resource is created, updated or deleted.
For all these flows it is responsible for reconciling the user intention to the state of the cluster.

When a Presentation resource is created, the controller will write the markdown content to a configmap.
Additionally it will also launch a pod based on the haxx-presentation docker container as described above and mount the configmap to /config directory in the pod.

The result is that a web page will serve the presentation build based on the markdown logic.
