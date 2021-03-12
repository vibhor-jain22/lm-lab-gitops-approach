# GitOps on Kubernetes with ArgoCD and Helm

## Instructions

### Pre-requisites

* Kubernetes cluster to be up and running and ArgoCD deployed.

* You should fork this repository so that you can edit, commit and push back to your own repository

### Step 1 - Fork this repository

Fork this repository to your own GitHub account. 

We fork the repository so that you can edit some of the code and push back your changes.

### Step 2 - Explore the devops-bookstore-api Helm chart

The [devops-bookstore-api](./devops-bookstore-api) directory is set up as a [Helm chart](https://helm.sh/docs/) for deploying the bookstore API.

You'll see the following files:

**Chart.yaml** This file defines our Helm chart including the name, chart version and application version.

**values.yaml** This file defines all the variables that are passed to our Helm chart. They essentially configure what is deployed and how it is deployed. We'll be editing this shortly.

**templates** This directory contains the Kubernetes YAML files that make up this chart. It bundles together the service, deployment and anything else needed.

### Step 3 - Update the values.yaml and push

Within the **values.yaml** file you'll see a value for the repository. You should update this to be that of the Docker image you pushed.

Examples of what it might look like:

On GCP it would look something like:  **eu.gcr.io/some-project-id/devops-bookstore-api**

On AWS it would look something like:  **AWS_ACCOUNT_ID.dkr.ecr.eu-west-2.amazonaws.com/devops-bookstore-api**

On Azure it would look something like: **devopsupskillregistry.azurecr.io/devopsupskill/devops-bookstore-api**

Once you have made the changes, commit them and push back to your repository.

### Step 4 - Grab the ArgoCD password

Now the exciting part - lets get your container deployed!

Firstly we need to log in to the ArgoCD tool. This is also covered in the provisioning guide so please ignore if you've already done this

By default, ArgoCD generates a password for you. To extract this run the following command:

```
kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2
```

### Step 5 - Port forwarding into the ArgoCD dashboard

We can use `kubectl` to port forward requests from our machine into the cluster. 

This is a fairly common technique for viewing a web application on the cluster that you do not wish to publicly expose.

To port forward local requests for port 8080 into Argo running on port 443 you can run:

```
kubectl port-forward svc/argocd-server -n argocd 9000:443
```

Then you should be able to visit [http://localhost:9000](http://localhost:8080). It will likely tell you that the certificate cannot be verified. This is because you haven't provisioned a full SSL certificate. You can ignore this warning and follow the instructions to "Visit the site anyway"

### Step 6 - Log in to the dashboard

It will ask you for the username and password.

**Username:** admin

**Password:** (As per what you generated in step 4)

### Step 7 - Create your application 

Follow this short video for creating your application:

[Creating your application in ArgoCD](https://storage.googleapis.com/devops-up-skill/session-004-argocd-app-creation.mp4)

### Step 8 - Marvel at your application running!

If you've edited things correctly then you should be able to see running pods when you do:

```
kubectl get pods
```

And you should see something similar to this:

```
NAME                                  READY   STATUS    RESTARTS   AGE
devops-bookstore-api-b4d85f9c-4x2mw   1/1     Running   0          100s
devops-bookstore-api-b4d85f9c-b8ks6   1/1     Running   0          100s
devops-bookstore-api-b4d85f9c-jcndf   1/1     Running   0          100s
```

In fact, because we made the Kubernetes service of type **LoadBalancer** you should even be able to see it publicly

Have a look at your **services**

```
kubectl get services
```

You should see something like this:

```
NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
devops-bookstore-api-service   LoadBalancer   10.147.241.171   35.246.32.61   5000:31102/TCP   6m41s
kubernetes                     ClusterIP      10.147.240.1     <none>         443/TCP          7h31m
```

Notice that **EXTERNAL-IP** - grab its address and go to it in the browser, using the example above I would go to:

[http://35.246.32.61:5000/books](http://35.246.32.61:5000/books)

**NOTE** On AWS it might show a Load Balancer DNS address for the EXTERNAL-IP. You can visit it just the same such as:

http://aeecab1189e7a4bde9834bc71baf6593-1118953692.eu-west-2.elb.amazonaws.com:5000/books

### Step 9 - Update your code, build the docker image and push to the registry

Let us pretend we've got our Dev hat on.

Go back to the codebase for the python app and update the code to include a new book.

Build the docker image remembering to tag with with a new version such as:

```
docker build -t devops-bookstore-api:1.1 .
```

And then go back over the steps you did before to push that new version up to your cloud (AWS, GCP or Azure) container registry. Don't forget to make sure you use the right tags.

You should now have a new version up in your container registry.

_Spoiler alert_: This develop, test, build, push cycle is what we'll be covering in session five where we set up a CI/CD pipeline.

### Step 10 - Update the GitOps repo and push to Git

So you've done a bit of that dev stuff. Now put your ops person hat on.

Dev have said there is a new release. 

Working in a GitOps manner, everything is stored in Git so we just need to update this repository to trigger Argo to do our next deploy.

Update the **Chart.yaml** and change the **appVersion** to be the new version such as **1.1**. 

Commit and push that change.

### Step 11 - Observe the change in Argo

After a few moments, ArgoCD will observe that it is now **Out of Sync**. If you get impatient just click **Refresh**

Essentially that is ArgoCD saying there seems to be a new version of that deployment when comparing to GitHub.

If you click **Sync** it will go ahead and update your application in turn deploying the latest version of the app.

### Step 12 - Relax and praise yourself

Well done!!! You're now fully GitOps

Marvel at what you've created! Let the world know, take those screenshots of Argo and share on social media! 

Yeah damn right you're proud - great work!

### Step 13 - Saving costs!

Once the dust has settled, you've taken your screenshots to share with the world then its probably time to destroy your cluster and save on costs.

**Important:** Log in to the ArgoCD dashboard and click **Delete** on the application to remove all the deployments. This is because they might have in turn provisioned load balancers (that weren't terraform managed).

Once you have deleted your the applications from Argo you can then remove Argo itself from your cluster:

```
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

And then finally go back to your provisioning directory (where all the terraform stuff was) and run:

```
terraform destroy
```



