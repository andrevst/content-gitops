# Course Labs

## Lab 1 Installing and Configuring Flux with GitHub

### Introduction

This lab introduces the steps necessary for installing Flux and configuring it to work with a repository in GitHub. We'll need our own GitHub account to fork a sample repository, and this lab will spin up a Kubernetes cluster to enable us to install and configure Flux.

Create a GitHub Repository
The linuxacademy/content-gitops repository is already online for us. Let's get into it and press the Fork button. Once the new repository is created, we'll use the credentials on the hands-on lab overview page to log into our Kubernetes master node as cloud_user.

Deploy Flux Into Your Cluster
Let's make sure Kubernetes is running, and that we have some nodes:

$ kubectl get nodes
Now let's make sure Flux is installed and running:

$ fluxctl version
We should get response of unversioned, which is fine. Now let's take another look at our Kubernetes deployment:

$ kubectl get pods --all-namespaces
Create a namespace for Flux:

$ kubectl create namespace flux
This will let us make sure it got created:

$ kubectl get namespaces
Set the GHUSER environment variable, then check to make sure it was set:

$ export GHUSER=[Our GitHub Handle]
$ env | grep GH
Now we can deploy Flux, using the fluxctl command:

$ fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/content-gitops \
--git-path=namespaces,workloads \
--namespace=flux | kubectl apply -f -
Verify The Deployment and Obtain the RSA Key
Once that fluxctl command is finished running, we can verify:

$ kubectl get pods --all-namespaces
$ kubectl -n flux rollout status deployment/flux
Now we can get the Flux RSA key created by fluxctl:

$ fluxctl identity --k8s-fwd-ns flux
Copy that RSA key, and let's head back over to GitHub.

Implement the RSA Key in GitHub
In the GitHub user interface, make sure we're in our new repository and click on the Settings tab. In there, click Deploy keys, then click the Add deploy key button. We can give it a Title of something like GitOps Deploy Key, then paste the key we copied earlier down in the Key field. Check the Allow write access box, and then click Add key.

Use the fluxctl sync Command to Synchronize the Cluster with the Repository
Use fluxctl to sync the cluster with the new repository:

$ fluxctl sync --k8s-fwd-ns flux
Then check the existence of the lasample namespace:

$ kubectl get namespaces
Then check that the Nginx deployment is running:

$ kubectl get pods --namespace=lasample
We should see the deployment running, with two replicas.

Conclusion
We're finished. We've installed Flux and configured it to work with a Kubernetes cluster and a GitHub repository. Congratulations!
## Lab 3

### Analyze The YAML Used To Install Flux

To output the yaml that is applied by kubectl, simply use the same command and either display it or route it to a file instead of piping it to kubectl.

```shell
$ fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/content-gitops \
--git-path=namespaces,production \
--namespace=flux
```
This output has also been placed in a file within the docs folder on the course repo, and is at this link:

https://github.com/andrevst/content-gitops/blob/master/docs/Flux_Install_Yaml.md

### Display the log produced by the fluxd Daemon

To display the log produced by the fluxd daemon running as a deployment in your cluster, enter the command:

```shell
$ kubectl -n flux logs deployment/flux
```

This assumes you have deployed flux to the flux namespace.


### Display details about the flux pod running

To display details about the pod deployed in your cluster, first obtain the unique pod name with the following command:

```Shell
$ kubectl -n flux get pods
```

Then copy the unique pod name to your clipboard and input:

```Shell
$ kubectl -n flux describe pod [your flux pod name here]
```

Example command with sample pod name:

```Shell
kubectl -n flux describe pod flux-c97899756-wsfhb
```

## Lab 5

Using Manifests and Flux with Kubernetes Clusters
Introduction
This hands-on learning experience challenges you to install and configure Flux with your own code repository (e.g., GitHub or GitLab). Once configured, you will create a namespace to deploy an application into, create a Deployment of a sample "Hello" application, and update the Deployment to a new release of the same application. Some troubleshooting is required to achieve the goals of this lab.

Solution
Open the lab's provided GitHub repository.

If you don't already have a personal GitHub account, create one before you start the lab by signing up at github.com.

Set up a GitHub Repository with the Required YAML
At the top right of the main GitHub page, click + > New repository.
In Respository name, enter "manifests_with_flux".
In Description, enter "Sample Repo for Lab".
Click Create repository.
In the new repository under Quick setup, get started by clicking creating a new file.
In the empty field after the the repository name, enter "namespaces/" to create the namespaces folder.
In the next empty field, enter the file name "myyns.yaml".
Under <> Edit new file, paste in the following sample YAML:

apiVersion: v1
kind: Namespace
metadata:
  labels:
    name: lamanifest
  name: lamanifest
Scroll to the bottom of the page and enter a description of the commit in the empty field.

Click Commit new file.
Log into the Kubernetes Master Node
In a terminal, log into the master node using the public IP and login credentials provided in the hands-on lab overview page.
Check the nodes:

kubectl get nodes
Check that Flux is installed:

fluxctl version
We should get response of unversioned.

Create a new namespace:

kubectl create namespace flux
Set the GHUSER environment variable, replacing <GITHUB_HANDLE> with your own GitHub handle:

export GHUSER=<GITHUB_HANDLE>
Install Flux:

fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/manifests_with_flux \
--git-path=namespaces,workloads \
--git-branch=main \
--namespace=flux | kubectl apply -f -
The RSA Key
Retrieve the Flux RSA key created with fluxctl:

fluxctl identity --k8s-fwd-ns flux
Copy the RSA to the clipboard for later use.

Return to the main GitHub page.
Click Settings.
From the left Options menu, select Deploy keys.
Click Add deploy key.
In Title, enter "manifests".
In Key, paste in the RSA key.
Select Allow write access and click Add key.
Sync the Cluster with the Repository and Check the Results
Return to the terminal.
Perform a sync:

fluxctl sync --k8s-fwd-ns flux
Check our namespaces:

kubectl get namespaces
We should see that the lamanifest namespace hasn't been created.

Create the Application Deployment YAML
Return to GitHub.
In the Code tab of our repository, click Add file > Create new file.
In the empty field, enter "workloads/" to create the workloads folder.
In the next empty field, enter "hello.yaml".
Paste in the following sample YAML:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  namespace: lasample
  labels:
    app: hello
spec:
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: linuxacademycontent/gitops:hellov1.0
Click Commit new file.

Sync Again
Set the environment variable to the Flux namepace:

export FLUX_FORWARD_NAMESPACE=flux
Sync again and retrieve a list of namespaces:

fluxctl sync
kubectl get namespaces
We should now see the lamanifest namespace.

Check that the Deployment exists:

kubectl get pods --all-namespaces
We should see a couple of flux Pods, but the hello Pod is missing.

There Is a Problem
Return to the hello.yaml file in GitHub.
Click the pencil icon to edit the file.
In namespace, change the given namespace from lasample to lamanifest:

namespace: lamanifest
Click Commit new file.

Sync again and check the Pods:

fluxctl sync
kubectl get namespaces
Check again for the hello Pod:

kubectl get pods --all-namespaces
Check Workloads and Images
Check the Flux workloads in the lamanifest namespace:

fluxctl -n lamanifest list-workloads
We should see that the hellov1.0 image is updating.

Check the deployed container image:

fluxctl -n lamanifest list-images
We should see the other images that we have out on Docker Hub and an untagged message in the output where our hellov1.0 image should be listed.

Verify the Image Tag Names and Deploy Containers
Use the automate command to add automation to the deployment, and see if that changes anything:

fluxctl automate --workload=lamanifest:deployment/hello
fluxctl -n lamanifest list-images
We should see that the hellov1.0 image is still untagged.

Return to the hello.yaml file in GitHub and refresh the browser to the folowing code added in the metadata section:

  annotations:
    fluxcd.io/automated: 'true'
Return to the terminal and list the images again:

fluxctl -n lamanifest list-images
Nothing has changed in our output, so the problem is something else.

In another browser tab, go to http://hub.docker.com/repository/docker/linuxacademycontent/gitops(http://hub.docker.com/repository/docker/linuxacademycontent/gitops and view the images. Note that hellov1.0 isn't listed.

Note: If you do not already have a Docker account, you will need to create one first to view the images.

Fixing the Version
To fix the issue, use the release command to specify a different container:

fluxctl release --workload=lamanifest:deployment/hello --update-image=linuxacademycontent/gitops:hellov1.2.2
In the output, you should see that it pushes to the repository and commits.

Return to GitHub and refresh the page. We should now see hellov1.0 (down around line 22) has been changed to hellov1.2.

Check the Pods again:

kubectl get pods -n lamanifest
We should see the running Pod now terminating and a new Pod spinning up.

Check the workloads again:

fluxctl -n lamanifest list-workloads
We should see that hellov1.2 is there, and ready.

Test this again, specifying hellowv1.2.1 image container this time:

fluxctl release --workload=lamanifest:deployment/hello --update-image=linuxacademycontent/gitops:hellov1.2.1
Check workloads again:

fluxctl -n lamanifest list-workloads
We should see the new version 1.2.1 listed now.

Conclusion
Congratulations — you've completed this hands-on lab!
