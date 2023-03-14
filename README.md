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

# Demo Overview

In this demo, we'll showcase how **roxctl generate netpol** simplifies the experience of creating and maintaining Kubernetes network policies, This means you can deploy an application with initial auto-generated micro-segmentation from day zero. 

Here's an overview of the different elements we'll be showcasing:

## Pipeline
Our CI/CD pipeline is integrated with ACS/Stackrox and based on Openshift-Pipelines/Tekton. It generates new network policies whenever changes are made to the application and pushed to the Git repository.

![pipeline](images/pipeline.png)

## Local Git Repository - Gogs
Our Git repository has two branches: "main" and "shiftleft-netpol." The "shiftleft-netpol" branch is the one our pipeline uses to push changes.

![branches](images/branches.png)

## Openshift-Gitops - argocd App
We're using an Openshift-Gitops, and the shiftleft-netpol-demo(argocd app) is configured to pull changes from the "shiftleft-netpol" branch of the Git repository. ArgoCD is responsible for delivering the new network policies to the live Kubernetes cluster.

![argocd](images/argocd-app.png)

## RHACS/Stackrox Network Dashboard - Without the Network Policies
The RHACS dashboard displays the current security posture of the Kubernetes cluster. In this demo, we'll start by showing the dashboard without any network policies applied.

![rhacs](images/rhacs-no-netpols.png)

## Running the Pipeline
The pipeline is generating new network policies based on the static YAML resource definitions in the branch "main" and once the new network policies are generated the pipeline will make changes to the "shiftleft-netpol" branch of the Git repository,

![pipelinerun](images/pipelinerun.png)

The changes to the "shiftleft-netpol" branch includes the file netpols.yaml as a resource to the kustomize.yaml file.

![git-changes](images/repo-changes.png)

## ArgoCD Status - out-of-sync
Once the changes are pushed to the Git repository, the ArgoCD status will change to "out-of-sync" because it has not yet delivered the new policies.

![out-of-sync](images/argocd-out-of-sync.png)

## Syncing the ArgoCD App Manually
We'll manually sync the ArgoCD app to deliver the new network policies to the live Kubernetes cluster and see that the status now shows as "Synced."

![synced](images/argocd-synced.png)

## RHACS Network Dashboard - With Network Policies Applied
Finally, we'll revisit the RHACS dashboard to see that the workloads are now secure, and strict access controls are being enforced between pods and services.

![rhacs](images/rhacs-with-netpols.png)

That concludes our demo of RHACS!