# Setting The Path

We will now try setting the path value.

The path will be explicitly called from what you set like any
[kustomize url](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/remoteBuild.md).

In previous examples we were deploying a Deployment, Service and ConfigMap

In the branch
["path" on the examples repo](https://github.com/slipway-gitops/slipway-example-app/blob/path/kustomize/base/kustomization.yaml)
the Service has been removed.

```yaml
commonLabels:
  app: hello

resources:
- deployment.yaml
- configMap.yaml
```

We will insure we are not running and gitrepo-samples at this time
```bash
$ kubectl delete gitrep gitrepo-sample
```

Now we will deploy the same basic deployment we used before but explicitly set the path to the "path" branch

```
apiVersion: git.gitops.slipway.org/v1
kind: GitRepo
metadata:
  name: gitrepo-sample
spec:
  uri: "git@github.com:slipway-gitops/slipway-example-app.git"
  operations:
    - operation: test
      path: "git@github.com:slipway-gitops/slipway-example-app.git//kustomize/base?ref=path"
      weight: 3
      optype: branch
      reference: master

```
We will now apply this manifest
```bash
$ kubectl apply -f path-url.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample configured
```
Now if we check all the resources that were deployed.
```bash
$ kubectl get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/the-deployment-7fd7749979-7tcjj   1/1     Running   0          50m
pod/the-deployment-7fd7749979-89p7j   1/1     Running   0          50m
pod/the-deployment-7fd7749979-scqp7   1/1     Running   0          50m

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/the-deployment   3/3     3            3           50m

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/the-deployment-7fd7749979   3         3         3       50m
```

We can see the Service was not deployed.

Explicitly setting the path is one option.
Another option is using the commit hash you are currently working on.

## HashPath
Using the same  basic example we will set hash path to true. and change the branch
to "path". So we are now matching the "path" branch but using the regular path.
Normally this would deploy all three services but because we have set "hashpath"
It will automatically append "?ref=yourhashhere" to the deployment.

```yaml
apiVersion: git.gitops.slipway.org/v1
kind: GitRepo
metadata:
  name: gitrepo-sample
spec:
  uri: "git@github.com:slipway-gitops/slipway-example-app.git"
  operations:
    - operation: test
      path: "git@github.com:slipway-gitops/slipway-example-app.git//kustomize/base"
      weight: 3
      hashpath: true
      optype: branch
      reference: path
```
We will apply this.
```bash
$ kubectl apply -f path-url.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample configured
```
First we may notice the deployed has is different.
```
$ kubectl get hash
NAME                                       AGE
abb5f4faacac21c1ebf0d50b20666842b19aeab0   6s
```

We are no longer using the master branch reference, but the
[hash to the last commit of the "path" branch.](https://github.com/slipway-gitops/slipway-example-app/commit/abb5f4faacac21c1ebf0d50b20666842b19aeab0)

You will also notice we still have not deployed the Service, because under that commit
It is not set.

Given these two options, you can explicitly set kustomize to deploy explicitly in the
same way everytime or you can have it behave contextually to your commit.

Tear it down.
```bash
$ kubectl delete gitrep gitrepo-sample
```
