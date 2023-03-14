# Overview

This README provides a step-by-step guide for installing the OpenShift Pipelines and OpenShift GitOps Operators and creating a demo pipeline for the shiftleft-netpol using RHACS.

# Pre-Requirements

Before proceeding with the installation, please ensure that you have the following:
- Access to an OpenShift cluster
- oc CLI tool installed and configured to work with your OpenShift cluster

# Installation Steps

## Clone the demo repository
```
git clone https://github.com/ralvares/shiftleft-netpol
cd shiftleft-netpol
```

## Install OpenShift Pipelines and OpenShift GitOps Operators

To install the OpenShift Pipelines and OpenShift GitOps Operators, execute the following commands:
```
oc apply -k operators/
oc wait --for condition=ready pod --selector=name=openshift-pipelines-operator --timeout=300s -n openshift-operators
oc wait --for condition=ready pod --selector=app=tekton-operator --timeout=300s -n openshift-operators
```

## Install Git Repository - Gogs
To install the Git Repository - Gogs, execute the following commands:
```
oc apply -k gogs/
oc wait --for condition=ready pod --selector=app=gogs --timeout=300s -n gogs
```

### Accessing the Gogs GUI
To access the Gogs GUI, execute the following command to get the route:
```
oc get route -n gogs
```
The command will output the URL for accessing the Gogs interface. Copy the URL and open it in a web browser to access the GUI.

*Please note that the Gogs GUI is not encrypted and is only intended for demo purposes.*

## Creating the shiftleft-netpol Demo Pipeline
To create the shiftleft-netpol demo pipeline, execute the following commands:

```
oc get clustertasks/git-clone && oc get clustertasks/git-cli && oc apply -k pipelines/
oc patch serviceaccount pipeline -p '{"secrets": [{"name": "git-repo-auth"}]}' -n shiftleft-netpol
``` 

## Creating the shiftleft-netpol argocd APP
To create the shiftleft-netpol demo argocd app, execute the following commands:

```
oc apply -k gitops/
```

### To get argocd admin password
```
oc get secret/openshift-gitops-cluster -n openshift-gitops -o jsonpath='{.data.admin\.password}' | base64 -d
```