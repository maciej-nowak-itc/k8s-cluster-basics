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

Initiate a local cluster.

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

## ArgoCD

TODO: Provide summary note.