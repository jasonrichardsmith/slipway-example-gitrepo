# Basic Deployment

A Basic Deployment

```
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
      optype: branch
      reference: master
```

Here we can see we are just deploying the [repo's base kustomization](https://github.com/slipway-gitops/slipway-example-app/blob/master/kustomize/base/kustomization.yaml).

```bash
$ kubectl apply -f basic.yaml
```
Now lets see what we deployed.


```bash
$ kubectl get gitrepo
NAME             AGE
gitrepo-sample   87s
```

```
$ kubectl describe gitrepo gitrepo-sample
Name:         gitrepo-sample
Namespace:
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"git.gitops.slipway.org/v1","kind":"GitRepo","metadata":{"annotations":{},"name":"gitrepo-sample"},"spec":{"operations":[{"o...
API Version:  git.gitops.slipway.org/v1
Kind:         GitRepo
Metadata:
  Creation Timestamp:  2020-03-18T11:55:33Z
  Generation:          1
  Resource Version:    2381498
  Self Link:           /apis/git.gitops.slipway.org/v1/gitrepoes/gitrepo-sample
  UID:                 419f1e47-51a8-4a41-8cf9-54b711520cb3
Spec:
  Operations:
    Operation:  test
    Optype:     branch
    Path:       git@github.com:slipway-gitops/slipway-example-app.git//kustomize/base
    Reference:  master
    Weight:     3
  Uri:          git@github.com:slipway-gitops/slipway-example-app.git
Status:
  Sha:
    API Version:       git.gitops.slipway.org/v1
    Kind:              Hash
    Name:              75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
    Resource Version:  2381496
    UID:               dd4c5523-20f0-4a42-8192-3e733603df56
Events:
  Type    Reason   Age                  From                Message
  ----    ------   ----                 ----                -------
  Normal  created  2m59s                gitrepo-controller  Repo created hash 75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
```
We have set the GitRepo and we can see we have a child Hash.  Let's see what is in the child Hash.

```bash
$ kubectl get hash 75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
NAME                                       AGE
75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda   5m27s
```
```bash
$ kubectl describe hash 75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
Name:         75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  git.gitops.slipway.org/v1
Kind:         Hash
Metadata:
  Creation Timestamp:  2020-03-18T11:55:34Z
  Generation:          1
  Owner References:
    API Version:           git.gitops.slipway.org/v1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  GitRepo
    Name:                  gitrepo-sample
    UID:                   419f1e47-51a8-4a41-8cf9-54b711520cb3
  Resource Version:        2381496
  Self Link:               /apis/git.gitops.slipway.org/v1/hashes/75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
  UID:                     dd4c5523-20f0-4a42-8192-3e733603df56
Spec:
  Gitrepo:  gitrepo-sample
  Operations:
    Hashpath:        false
    Operation:       test
    Optype:          branch
    Path:            git@github.com:slipway-gitops/slipway-example-app.git//kustomize/base
    Reference:       master
    Referencetitle:  master
    Transformers:
    Weight:  3
Status:
  Active:
    API Version:       v1
    Kind:              ConfigMap
    Name:              the-map
    Namespace:         default
    Resource Version:  2381435
    UID:               85956a46-8142-4c11-bc89-38c2638c4311
    API Version:       v1
    Kind:              Service
    Name:              the-service
    Namespace:         default
    Resource Version:  2381438
    UID:               5cfa03d7-8746-4b93-897c-e7719ba7e8d7
    API Version:       apps/v1
    Kind:              Deployment
    Name:              the-deployment
    Namespace:         default
    Resource Version:  2381491
    UID:               e092330d-2cdd-4864-aec1-03eb621b6dc0
Events:                <none>
```
Here we can see we have a Hash that is deployed and under the Status we have a series of child objects.

All of the objects are under the default namespace so lets see if they exist.

```bash
$ kubectl get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/the-deployment-7fd7749979-l2tsl   1/1     Running   0          10m
pod/the-deployment-7fd7749979-wthrz   1/1     Running   0          10m
pod/the-deployment-7fd7749979-xbws7   1/1     Running   0          10m

NAME                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/the-service   LoadBalancer   10.108.228.176   <pending>     8666:31734/TCP   10m

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/the-deployment   3/3     3            3           10m

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/the-deployment-7fd7749979   3         3         3       10m
```
Let's delete the Hash and see what happens

```
$ kubectl delete hash 75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
hash.git.gitops.slipway.org "75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda" deleted
```
Let's see what is running
```
$ kubectl get all
NAME                                  READY   STATUS        RESTARTS   AGE
pod/the-deployment-7fd7749979-l2tsl   0/1     Terminating   0          11m
pod/the-deployment-7fd7749979-wthrz   0/1     Terminating   0          11m
pod/the-deployment-7fd7749979-xbws7   0/1     Terminating   0          11m

```

Depending on how quickly you ran that command will determine what you see.

You may see this, or you may have to wait 2-3 minutes and it will be back.
```
$ kubectl get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/the-deployment-7fd7749979-bgngw   1/1     Running   0          117s
pod/the-deployment-7fd7749979-j5gjg   1/1     Running   0          117s
pod/the-deployment-7fd7749979-zkspr   1/1     Running   0          117s

NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/the-service   LoadBalancer   10.101.28.147   <pending>     8666:32179/TCP   117s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/the-deployment   3/3     3            3           117s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/the-deployment-7fd7749979   3         3         3       117s
```

If we check for the Hash we will see it is back as well.
```
$ kubectl get hash
NAME                                       AGE
75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda   4m25s
```
The hash has returned.  This is because the GitRepo is constantly scanning the GitRepo to insure
what is being referenced in the Operations is always in your cluster.

Now lets cleanup and see what happens.

```
$ kubectl delete gitrepo gitrepo-sample
gitrepo.git.gitops.slipway.org "gitrepo-sample" deleted
```
```
$ kubectl get hash
No resources found in default namespace.
```

```
$ k get all
No resources found in default namespace.
```

All the resources have been removed.
