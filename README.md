# Cluster API Provider for Managed Bare Metal Hardware

This repository will contain a Machine actuator implementation for the
Kubernetes [Cluster API](https://github.com/kubernetes-sigs/cluster-api/).  It
is currently still under active development and is not yet functional.

For more information about this actuator and related repositories, see
[metalkube.org](http://metalkube.org/).

## Dev Workflow

You can run this component manually against a running Kubernetes cluster for
development purposes.  The current version of this code is built against the
OpenShift fork of the Machine API, so the instructions assume an OpenShift
cluster, and the use of the OpenShift client.

### Stop the machine-api-operator

If this is an OpenShift 4 based cluster, there is a Machine controller
already via the machine-api-operator (MAO). We start by stopping the MAO. This
operator is managed by the Cluster Version Operator (CVO), so we need to make
sure the CVO will not undo our custom changes first.

One method is to specifically tell the CVO to stop managing the Machine API
Operator. There are some docs on this process
[here](https://github.com/openshift/cluster-version-operator/blob/master/docs/dev/clusterversion.md#setting-objects-unmanaged).
(If you work out the exact steps, please submit them as a PR!)

A brute force approach is to just stop the CVO completely.

```bash
$ oc scale deployment cluster-version-operator -n openshift-cluster-version --replicas=0

$ oc get deployment cluster-version-operator -n openshift-cluster-version
NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
cluster-version-operator   0         0         0            0           6d
```

Then you can stop the Machine API Operator by scaling it down, as well.

```bash
$ oc scale deployment machine-api-operator -n openshift-machine-api --replicas=0

$ oc get deployment machine-api-operator -n openshift-machine-api
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
machine-api-operator   0         0         0            0           6d
```

### Create CRDs if necessary

If you start with a new enough OpenShift 4 cluster, this will not be necessary.
It's also possible to test this against any Kubernetes cluster.  In that case,
you will need to create a namespace and add the expected CRDs to the system first.

```bash
oc create namespace openshift-machine-api

git clone https://github.com/openshift/machine-api-operator
cd machine-api-operator

oc create -n openshift-machine-api -f test/integration/manifests/0000_30_machine-api-operator_02_machine.crd.yaml
oc create -n openshift-machine-api -f test/integration/manifests/0000_30_machine-api-operator_03_machineset.crd.yaml
oc create -n openshift-machine-api -f test/integration/manifests/0000_30_machine-api-operator_04_machinedeployment.crd.yaml
oc create -n openshift-machine-api -f test/integration/manifests/0000_30_machine-api-operator_05_cluster.crd.yaml
```

### Run our machine controller

Now you can manually run our own machine controller.

```bash
cd cluster-api-provider-baremetal
make build
export KUBECONFIG=...
./bin/machine-controller-manager
```
