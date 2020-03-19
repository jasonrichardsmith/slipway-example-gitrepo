# Transformers

Slipway allows you to transform using standard Kustomize transformers.

This is only intended to transform based on the context of the commit you are
currently working in, all other transformations should be captured in the 
kustomize manifests in your repo.

There are only three values to each transformer.
```yaml
        - type: annotations
          value: branch
          key: branch-name
```
- The "type" of transformer
- The "value" to be derived from the commit or just a plain string
- The "key" this is the key value for annotations and labels.  It is also used to name the image you want to transform

## Annotations Transformer

The annotations transformer behaves just like the
[commonAnnotations plugin](https://kubectl.docs.kubernetes.io/pages/reference/kustomize.html#commonannotations)

To see it work we can deploy the annotate.yaml
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
      optype: branch
      reference: m[a-z]+r
      transformers:
        - type: annotations
          value: branch
          key: branch-name
```
This will add the annotation of "branch-name: master" because the regex will match the master branch name.
Remove the gitrepo-sample if it exists already
```bash
$ kubectl delete gitrepo gitrepo-sample
```
And create the gitrepo-sample
```bash
$ kubectl apply -f annotate.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample created
```

We can all of our objects have been created.  If you check any of the created objects you can 
see they have annotations added.

```bash
$ kubectl get service the-service -o yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    branch-name: master
...
```

## Labels Transformer

The labels transformer behaves just like the
[commonLabels plugin](https://kubectl.docs.kubernetes.io/pages/reference/kustomize.html#commonlabels)

To see it work we can deploy the label.yaml
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
      optype: tag
      reference: v1.[0-9].[0-9]
      transformers:
        - type: labels
          value: tag
          key: commit-tag

```
This will add the label of "commit-hash: thefullhashopfthecommit".
Remove the gitrepo-sample if it exists already
```bash
$ kubectl delete gitrepo gitrepo-sample
```
And create the gitrepo-sample
```bash
$ kubectl apply -f label.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample created
```

We can see all of our objects have been created.  If you check any of the created objects you can 
see they have labels added.

```bash
$ kubectl get service the-service -o yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello
    commit-hash: 75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
...
```

## Prefix or Suffix Transformer

The prefix and suffix transformers behave just like the
[namePrefix plugin](https://kubectl.docs.kubernetes.io/pages/reference/kustomize.html#nameprefix)
and
[nameSuffix plugin](https://kubectl.docs.kubernetes.io/pages/reference/kustomize.html#namesuffix)

***Warning*** kubernetes objects names have to follow certain patterns so "prefix" with hash can be
problematic, and using version tags are also a problem.

### Prefix

To see it work we can deploy the prefix.yaml
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
      optype: branch
      reference: path
      transformers:
        - type: prefix
          value: branch

```


This will prefix the name of every object with the branch name.
Remove the gitrepo-sample if it exists already
```bash
$ kubectl delete gitrepo gitrepo-sample
```
And create the gitrepo-sample
```bash
$ kubectl apply -f prefix.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample created
```

We can see all of our objects have been created.
```bash
$ kubectl get all
NAME                                       READY   STATUS              RESTARTS   AGE
pod/path-the-deployment-7fd7749979-4sjwc   0/1     ContainerCreating   0          0s
pod/path-the-deployment-7fd7749979-lq2b8   0/1     ContainerCreating   0          0s
pod/path-the-deployment-7fd7749979-wwxzw   0/1     ContainerCreating   0          0s

NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/path-the-service   LoadBalancer   10.107.201.63   <pending>     8666:30084/TCP   0s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/path-the-deployment   0/3     3            0           0s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/path-the-deployment-7fd7749979   3         3         0       0s
```
Notice all objects have been prefixed with "path"


### Suffix

To see it work we can deploy the suffix.yaml
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
      optype: branch
      reference: m[a-z]+r
      transformers:
        - type: suffix
          value: hash

```


This will suffix the name of every object with the commit hash.
Remove the gitrepo-sample if it exists already
```bash
$ kubectl delete gitrepo gitrepo-sample
```
And create the gitrepo-sample
```bash
$ kubectl apply -f suffix.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample created
```

We can see all of our objects have been created.
```bash
$ kubectl get all                                                      2303  12:05:26 
NAME                                                                  READY   STATUS                       RESTARTS   AGE
pod/the-deployment-75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda-7fnb8cx   0/1     CreateContainerConfigError   0          6s
pod/the-deployment-75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda-7fqxq8m   0/1     CreateContainerConfigError   0          6s
pod/the-deployment-75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda-7fzln69   0/1     CreateContainerConfigError   0          6s

NAME                                                           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/the-service-75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda   LoadBalancer   10.110.85.163   <pending>     8666:30825/TCP   6s

NAME                                                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/the-deployment-75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda   0/3     3            0           6s

NAME                                                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/the-deployment-75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda-7fd7749979   3         3         0       6s

```
Notice all objects have been suffixed with the commit hash

## Namespace Transformer

The namespace transformer behaves just like the
[namespace plugin](https://kubectl.docs.kubernetes.io/pages/reference/kustomize.html#namespace)

To see it work we can deploy the namespace.yaml
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
      optype: tag
      reference: v1.[0-9].[0-9]
      transformers:
        - type: namespace
          value: hash
```
This will add all the objects to the namespace named after the hash.
If the namespace does not exists it will be created.

Remove the gitrepo-sample if it exists already
```bash
$ kubectl delete gitrepo gitrepo-sample
```
And create the gitrepo-sample
```bash
$ kubectl apply -f namespace.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample created
```
If we run a get command in the default namespace we will not have any resources.

```bash
$ kubectl get all
No resources found in default namespace.
```

If we check the namespaces
```
$ kubectl get ns
NAME                                       STATUS   AGE
75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda   Active   2m57s
default                                    Active   16d
slipway-system                             Active   44h
...
```

We can see we have a new namespace with the hash value as the name.
We will now check the hash namespace.
```
$ kubectl get all -n 75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
NAME                                  READY   STATUS    RESTARTS   AGE
pod/the-deployment-7fd7749979-62bkt   1/1     Running   0          4m56s
pod/the-deployment-7fd7749979-dmngd   1/1     Running   0          4m56s
pod/the-deployment-7fd7749979-snxtt   1/1     Running   0          4m56s

NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/the-service   LoadBalancer   10.104.10.101   <pending>     8666:32646/TCP   4m56s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/the-deployment   3/3     3            3           4m56s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/the-deployment-7fd7749979   3         3         3       4m56s
```
We can see all the objects were created there.




## Image Transformer

The image transformer behaves just like the
[images plugin](https://kubectl.docs.kubernetes.io/pages/reference/kustomize.html#images)
It only supports tag overwriting.

To see it work we can deploy the image.yaml
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
      optype: tag
      reference: v1.[0-9].[0-9]
      transformers:
        - type: image
          value: foo
          key: monopole/hello
```
This will assign the tag "foo" to the image that has a name "monopole/hello".

We will expect the pods to fail pulling this image because it does not exist.

Remove the gitrepo-sample if it exists already
```bash
$ kubectl delete gitrepo gitrepo-sample
```
And create the gitrepo-sample
```bash
$ kubectl apply -f image.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample created
```

If we check the pods we should see an image pull issue
```
$ kubectl get pods
NAME                              READY   STATUS         RESTARTS   AGE
the-deployment-84d47f9d56-b7gsj   0/1     ErrImagePull   0          13s
the-deployment-84d47f9d56-drw8x   0/1     ErrImagePull   0          13s
the-deployment-84d47f9d56-fhknn   0/1     ErrImagePull   0          13s
```

If we get the events we can see it is trying to pull a tag monopole/hello:foo that does not exist,
but our tag manipulation worked as expected.

```
$ kubectl get events
...
8s          Normal    Pulling             pod/the-deployment-84d47f9d56-fhknn                                             Pulling image "monopole/hello:foo"
6s          Warning   Failed              pod/the-deployment-84d47f9d56-fhknn                                             Failed to pull image "monopole/hello:foo": rpc error: code = Unknown desc = Error response from daemon: manifest for monopole/hello:foo not found: manifest unknown: manifest unknown
6s          Warning   Failed              pod/the-deployment-84d47f9d56-fhknn                                             Error: ErrImagePull
22s         Normal    BackOff             pod/the-deployment-84d47f9d56-fhknn                                             Back-off pulling image "monopole/hello:foo"
22s         Warning   Failed              pod/the-deployment-84d47f9d56-fhknn                                             Error: ImagePullBackOff
...
```

You can tear down now.

```
$ kubectl delete gitrepo gitrepo-sample 
```

