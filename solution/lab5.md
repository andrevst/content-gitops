# Lab 5 Using Manifests and Flux with Kubernetes Clusters

Using Manifests and Flux with Kubernetes Clusters
Introduction
This hands-on learning experience challenges you to install and configure Flux with your own code repository (e.g., GitHub or GitLab). Once configured, you will create a namespace to deploy an application into, create a Deployment of a sample "Hello" application, and update the Deployment to a new release of the same application. Some troubleshooting is required to achieve the goals of this lab.

## Solution

### Set up a GitHub Repository with the Required YAML

- Respository name, "[manifests_with_flux](https://github.com/andrevst/manifest_with_flux)".

file name "myyns.yaml".

```YAML
apiVersion: v1
kind: Namespace
metadata:
  labels:
    name: lamanifest
  name: lamanifest
```

- Commit file.
- Log into the Kubernetes Master Node
In a terminal, log into the master node using the public IP and login credentials provided in the hands-on lab overview page.

Install Flux on the lab as in [LAB 1 Installing and Configuring Flux with GitHub](##lab-1-installing-and-configuring-flux-with-github)

```shell
fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/manifest_with_flux \
--git-path=namespaces,workloads \
--git-branch=main \
--namespace=flux | kubectl apply -f -
```

### Config RSA Key

- Retrieve the Flux RSA key created with fluxctl:

```shell
fluxctl identity --k8s-fwd-ns flux
```

- Copy the RSA to the clipboard for later use
- add to github repo settings
- Perform a sync:

```shell
fluxctl sync --k8s-fwd-ns flux
```

- Check our namespaces, lamanifest namespace hasn't been created.

```shell
kubectl get namespaces
```

### Create the Application Deployment YAML

Return to GitHub.
In the Code tab of our repository, click Add file > Create new file.
In the empty field, enter "workloads/" to create the workloads folder.
In the next empty field, enter "hello.yaml".

```YAML
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
```

- Commit

Set the environment variable to the Flux namepace:

```Shell
export FLUX_FORWARD_NAMESPACE=flux
```

- Sync again and retrieve a list of namespaces:

```shell
fluxctl sync
```

```shell
kubectl get namespaces
```

- See the lamanifest namespace.

- Check if the Deployment exists

```shell
kubectl get pods --all-namespaces
```

***There are a couple of flux Pods, but the hello Pod is missing. There Is a Problem.***

- Return to the hello.yaml file in GitHub
- Edit it, change namespace from lasample to lamanifest: 

```YAML
namespace: lamanifest
```

- Commit chnages
- Sync again

```Shell
fluxctl sync
kubectl get namespaces
```

- Check again for the hello Pod

```Shell
kubectl get pods --all-namespaces
```

Check Workloads and Images
Check the Flux workloads in the lamanifest namespace:

```Shell
fluxctl -n lamanifest list-workloads
```

**hellov1.0 image is updating.**

- Check the deployed container image

```Shell
fluxctl -n lamanifest list-images
```

*other images that we have out on Docker Hub and an untagged message in the output where our hellov1.0 image should be listed.*

- Verify the Image Tag Names and Deploy Containers

### Use the automate command to add automation to the deployment, and see if that changes anything

```Shell
fluxctl automate --workload=lamanifest:deployment/hello
fluxctl -n lamanifest list-images
```

**hellov1.0 image is still untagged.**

- Return to the hello.yaml file in GitHub and refresh the browser to the folowing code added in the metadata section

```YAML
  annotations:
    fluxcd.io/automated: 'true'
```

- Return to the terminal and list the images again

```Shell
fluxctl -n lamanifest list-images
```

Nothing has changed in our output, so the problem is something else.

In another browser tab, go to http://hub.docker.com/repository/docker/linuxacademycontent/gitops(http://hub.docker.com/repository/docker/linuxacademycontent/gitops and view the images. Note that hellov1.0 isn't listed.

Note: If you do not already have a Docker account, you will need to create one first to view the images.

### Fixing the Version

- To fix the issue, use the release command to specify a different container

```Shell
fluxctl release --workload=lamanifest:deployment/hello --update-image=linuxacademycontent/gitops:hellov1.2.2
```

- Return to GitHub and refresh the page. We should now see hellov1.0 (down around line 22) has been changed to hellov1.2.

- Check the Pods again

```Shell
kubectl get pods -n lamanifest
```

**we should see the running Pod now terminating and a new Pod spinning up.**

- Check the workloads again

```Shell
fluxctl -n lamanifest list-workloads
```

**We should see that hellov1.2 is there, and ready.**

- Test this again, specifying hellowv1.2.1 image container this time

```Shell
fluxctl release --workload=lamanifest:deployment/hello --update-image=linuxacademycontent/gitops:hellov1.2.1
```

- Check workloads again

fluxctl -n lamanifest list-workloads

**We should see the new version 1.2.1 listed now.**