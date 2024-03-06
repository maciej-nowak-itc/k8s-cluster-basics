# Local cluster set up

Local cluster spin up.

## General notes

Some of the subprojects might require addtional resuorces to be delivered to cluster, such as secrets.
By default `.gitignore` is set to ignore any file that starts with `secret` literal. 

## Kind

Kind based local cluster. Usege of kind is not obligatory, treat it only as an example. Other clusters managers might be added later.

### Prequisities/ assumptions:
* docker, see [Docker install](https://docs.docker.com/engine/install/)
* kubectl, see [kubectl install](https://kubernetes.io/docs/tasks/tools/)
* kind, see [kind install](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

All of the above istalled and available from CLI/bash.


### Installation

#### Initiate a local cluster.

Note that cluster config is using single node set up - a control-plane node that will host both the control plane and workloads.

Single cluster:
```
kind create cluster --config ./kind/kind-cluster.yaml [--name=your-name]
```

where _--name=your-name_ is optional name for your cluster. If ommitted, your cluster will be named _kind-kind_, when name is provided it will be dubbed _kind-\<your-name>_.

Multi cluster. To set up multicluster environment on single device use:
```
kind create cluster --config ./kind/kind-cluster.yaml --name=prod
kind create cluster --config ./kind/kind-cluster-dev.yaml --name=non-prod
```
Treat the names used above as an example only.

You can set up as many clusters as necessary, just be sure to avoid collision on hostPorts.

#### Ingress
```
kubectl apply -k ./kind/
```

## ArgoCD

### Namespace and CRD

Assuming standard NS for ArgoCd = `argocd`.

The script includes ArogCDs CRDs.

Note that kustomize accounts for presence of `secret.oidc.azure.yaml` in `.\argocd` folder, which is Git immune, so one needs to provision it separatly.

```
kubectl apply -k ./argocd/_pre-install
```

### ArgoCD with Kind IngressController

```
kubectl apply -k ./argocd/overlays/kind
```

### (opt) Finalizers

Get default admin password, login and change it.

Snippet below assumes local host mapping of `argocd.kind.localhost`.
```
#Unix based
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
#Win CMD
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | wsl base64 -d

# On prompt sepcify user 'admin' and acquired password
argocd login argocd.kind.localhost --insecure --grpc-web

# On prompt specify inital admin password, than the new one.
argocd account update-password
```

Need to create apps outside of `argocd` NS?
```
kubectl apply -f ./argocd/_post-install/default.appproject-patch.yaml
```

Want to set template for private repo authorization?

Add `repo-creds` flagged `Secret` to your `argocd` NS. Below you have an example, based on SSH. But you can use HTTPS user/pass based too.
```
apiVersion: v1
kind: Secret
metadata:
  name: my-repo-secret
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repo-creds
stringData:
  type: git
  url: ssh://git@bitbucket.myenterpriseinstance.com/
  sshPrivateKey: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    ....
    -----END OPENSSH PRIVATE KEY-----
```

Also to allow for SSH handshake you should register your enterprise repo SSH public key. Note that all keys sits inder one property of single CM (argocd-ssh-known-hosts-cm) so there is no nice and clean way to simply add your entries in declarative way (unless you use some extra scripting that would allow you to preserve known hosts provided upfront). To keep it simple just use UI or CLI. For refernce see [Unknown SSH Hosts](https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/#unknown-ssh-hosts)