# GitOps on Kubernetes with ArgoCD and Helm

## Instructions

### Pre-requisites

Kubernetes cluster to be up and running

### Step 1 - Deploy ArgoCD

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait for pods to be created, keep checking the status:

```
kubectl get pods -n argocd
```

Once they are all running move on to step 2.

### Step 2 - Grab the ArgoCD password

You'll need the ArgoCD password for navigating into the dashboard.

By default, ArgoCD generates a password for you. To extract this run the following command:

```
kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2
```

### Step 3 - Port forwarding into the ArgoCD dashboard

We can use kubectl to port forward requests from our machine into the cluster. 

This is a fairly common technique for viewing a web application on the cluster that you do not wish to publicly expose.

To port forward local requests for port 8080 into Argo running on port 443 you can run:

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then you should be able to visit [http://localhost:8080](http://localhost:8080). It will likely tell you that the certificate cannot be verified. This is because you haven't provisioned a full SSL certificate. You can ignore this warning and follow the instructions to "Visit the site anyway"

### Step 4 - Creating an application via UI


Plan 

Re-provision their cluster from a branch, this time with a container registry

Build their image and push to their container registry

Manually deploy their application to the cluster by editing the deployment.yaml

But what is problem? It was manual, it wasn;t observed, we can't easily see history.

Whats good at storing changes of what happened and when?

Enter GitOps - version control for when operations do something

Create a repository for your operations config

Repository will be a Helm chart based repo

Change the image address to your container registry address

Get it deploying your Flask API

Build a new image and push to your container registry

Update your ops repo and change tag

Sync with ArgoCD (out of date)

Apply changes


Teardown ArgoCD

```
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```



