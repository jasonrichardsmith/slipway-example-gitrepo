# Pull Requests


In this example we will see closed and open pull request behavior.

When setting an operation to act on pull requests it will react to ***ALL*** pull requests.
With that in mind it is not advisable to run this against a public repository, since anyone can
submit a pull request to a public repository.  If you wish to go into more detailed tests and to play
with opening and merging pull requests it is suggested you create a
[private duplicate](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/duplicating-a-repository)
(forking does not work) of the [slipway-example-app](https://github.com/slipway-gitops/slipway-example-app) and 
perform pull requests against that repo.  You will only need to change the git repo references in
[pulls.yaml](pulls.yaml).

To Follow on the above point, if you are not running this off a private repository.
If you alter the [pulls.yaml](pulls.yaml) specifically adding hashpath, you run the 
risk of running someone else's kustomize, possibly launching malicious software into your cluster.
If you use the [pulls.yaml](pulls.yaml) as is, you will only deploy safe objects from the [basic example](BASIC.md)


The pulls.yaml looks like this
```yaml
piVersion: git.gitops.slipway.org/v1
kind: GitRepo
metadata:
  name: gitrepo-sample
spec:
  uri: "git@github.com:slipway-gitops/slipway-example-app.git"
  operations:
    - operation: test
      path: "git@github.com:slipway-gitops/slipway-example-app.git//kustomize/base"
      weight: 3
      optype: pull
      transformers:
        - type: namespace
          value: pull
```

It tells Slipway to create a hash operation for every active pull request.  It runs the namespace 
transformer on every pull request replacing the namespace name with pull-#.

First lets tear down any existing gitrepo-sample
```bash
$ kubectl delete gitrepo gitrepo-sample
```

Now apply the pulls.yaml
```bash
$ kubectl apply -f pulls.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample created
```

If we look at the namespaces in the cluster we will see two new namespaces.

```bash
$ kubectl get ns
NAME              STATUS   AGE
default           Active   17d
pull-3            Active   89s
pull-5            Active   91s
slipway-system    Active   138m
```

These namespaces coincide with only the
[active pull requests](https://github.com/slipway-gitops/slipway-example-app/pulls?q=is%3Apr)
in your repo and will be removed if a branch gets merged or deleted.  You will have to experiment
with this yourself on a private repo.

You can see all objects were deployed in both namespaces.
```
$ kubectl get all -n pull-3
NAME                                  READY   STATUS    RESTARTS   AGE
pod/the-deployment-7fd7749979-4cb9r   1/1     Running   0          8m27s
pod/the-deployment-7fd7749979-npbsv   1/1     Running   0          8m27s
pod/the-deployment-7fd7749979-xz22h   1/1     Running   0          8m27s

NAME                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/the-service   LoadBalancer   10.103.134.204   <pending>     8666:32378/TCP   8m27s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/the-deployment   3/3     3            3           8m27s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/the-deployment-7fd7749979   3         3         3       8m27s
```

If you were to add a "hashpath: true" the kustomize would run against the merged pull request
meaning you could possibly run any kustomize, so always trust the source.

You can tear it down.
```bash
$ kubectl delete gitrepo gitrepo-sample
```

