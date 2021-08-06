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
