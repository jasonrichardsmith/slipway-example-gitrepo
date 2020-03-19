# Regex

Slipway allows for Regex matching on your Github references as well.

We will now see how that works.

If we look at the [regex-branch.yaml](regex-branch.yaml) file we can see the Reference field now
has a regex expression.

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
```
Let's deploy this

```
$ kubectl apply -f regex-branch.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample created
```

We will jump straight to the hash

```
$ kubectl get hash
NAME                                       AGE
75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda   61s

$ kubectl describe hash 75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
Name:         75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  git.gitops.slipway.org/v1
Kind:         Hash
Metadata:
  Creation Timestamp:  2020-03-18T13:13:54Z
  Generation:          1
  Owner References:
    API Version:           git.gitops.slipway.org/v1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  GitRepo
    Name:                  gitrepo-sample
    UID:                   12f61798-9715-4537-89e4-92e8b7412a3f
  Resource Version:        2394333
  Self Link:               /apis/git.gitops.slipway.org/v1/hashes/75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
  UID:                     261ff24d-2d00-4753-b864-02cdebfb5991
Spec:
  Gitrepo:  gitrepo-sample
  Operations:
    Hashpath:        false
    Operation:       test
    Optype:          branch
    Path:            git@github.com:slipway-gitops/slipway-example-app.git//kustomize/base
    Reference:       m[a-z]+r
    Referencetitle:  master
    Transformers:
    Weight:  3
Status:
  Active:
    API Version:       v1
    Kind:              ConfigMap
    Name:              the-map
    Namespace:         default
    Resource Version:  2394278
    UID:               81e2991c-27a2-43c3-aeb7-199d0ced7c49
    API Version:       v1
    Kind:              Service
    Name:              the-service
    Namespace:         default
    Resource Version:  2394281
    UID:               0ad174e1-95e7-49ba-ac2b-6deaddb35c1c
    API Version:       apps/v1
    Kind:              Deployment
    Name:              the-deployment
    Namespace:         default
    Resource Version:  2394332
    UID:               894125cc-2263-4e7f-a805-755ae6cf53a7
Events:                <none>
```
If you notice we have the same deployment as the original basic example.

You will also notice Referencetitle: refers to the branch that got deployed.

If multiple hash references matched the regex, all would have been deployed under the same
hash.  each would have the same operation with a different refrence title. 
More on this in later chapters.

Let's tear this down and try with tags.

```
$ kubectl delete gitrepo gitrepo-sample
gitrepo.git.gitops.slipway.org "gitrepo-sample" deleted
```

Now let's look at the [regex-tag.yaml](regex-tag.yaml) file.

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
```
We will now deploy this as well.

```
$ kubectl apply -f regex-tag.yaml
gitrepo.git.gitops.slipway.org/gitrepo-sample created
```
Once again lets get the hash

```
$ kubectl get hash
NAME                                       AGE
75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda   61s

$ kubectl describe hash 75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda

Name:         75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  git.gitops.slipway.org/v1
Kind:         Hash
Metadata:
  Creation Timestamp:  2020-03-19T07:06:07Z
  Generation:          1
  Owner References:
    API Version:           git.gitops.slipway.org/v1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  GitRepo
    Name:                  gitrepo-sample
    UID:                   a83671c2-7b81-4365-b4ce-c8505718bf43
  Resource Version:        2430944
  Self Link:               /apis/git.gitops.slipway.org/v1/hashes/75f4b6e33ebbf8e0977dca4062c8ced9d49eeeda
  UID:                     b0b25bfd-949e-4db1-aeb6-9e72745e6708
Spec:
  Gitrepo:  gitrepo-sample
  Operations:
    Hashpath:        false
    Operation:       test
    Optype:          tag
    Path:            git@github.com:slipway-gitops/slipway-example-app.git//kustomize/base
    Reference:       v1.[0-9].[0-9]
    Referencetitle:  v1.1.0
    Transformers:
        Weight:  3
Status:
  Active:
    API Version:       v1
    Kind:              ConfigMap
    Name:              the-map
    Namespace:         default
    Resource Version:  2430885
    UID:               cf5eb414-98cb-445c-983a-98053f7ee7e9
    API Version:       v1
    Kind:              Service
    Name:              the-service
    Namespace:         default
    Resource Version:  2430889
    UID:               544dee3e-1bc0-4c84-8609-3e6a45fd01bf
    API Version:       apps/v1
    Kind:              Deployment
    Name:              the-deployment
    Namespace:         default
    Resource Version:  2430942
    UID:               2927d8f1-9882-41c4-a30a-5d082e82b07d
Events:                <none>
```
The first thing you may notice is that the Hash is exactly the same.
This is because your tag references the same commit as the branch.
You will also notice the Referencetitle is set to "v1.1.0" which is reference
name it matches.

I hope this gives an understanding n how to perform regex git references

Pull requests does not support regex.  If your optype is "pull" it launches every
active pull request.
