## Link
https://docs.openshift.com/container-platform/4.2/logging/cluster-logging-deploying.html


## Prerequisite
OCP4.2

## Procedure
1. Create Namespace for `openshift-operators`  
```
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-operators-redhat
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-logging: "true"
    openshift.io/cluster-monitoring: "true"
```
1. create Namespace for `openshift-logging`
```
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-logging
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-logging: "true"
    openshift.io/cluster-monitoring: "true"
```
1. Create Operator group object
```
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-operators-redhat
  namespace: openshift-operators-redhat
spec: {}
```
1. Create `CatalogSourceConfig` object
```
apiVersion: "operators.coreos.com/v1"
kind: "CatalogSourceConfig"
metadata:
  name: "elasticsearch"
  namespace: "openshift-marketplace"
spec:
  targetNamespace: "openshift-operators-redhat"
  packages: "elasticsearch-operator"
  source: "elasticsearch"
```
1. confirm operator status
```
$ oc get packagemanifest elasticsearch-operator -n openshift-marketplace -o jsonpath='{.status.channels[].name}'
4.2
```
1. Create Subscription object
```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  generateName: "elasticsearch-"
  namespace: "openshift-operators-redhat" 
spec:
  channel: "4.2" 
  installPlanApproval: "Automatic"
  source: "redhat-operators"
  sourceNamespace: "openshift-marketplace"
  name: "elasticsearch-operator"
```
1. Change to the `openshift-operators-redhat` project
```
$ oc project openshift-operators-redhat
Already on project "openshift-operators-redhat" on server "https://api.oc4cluster.tetsuya.local:6443
```
1. create RBAC object
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: prometheus-k8s
  namespace: openshift-operators-redhat
rules:
- apiGroups:
  - ""
  resources:
  - services
  - endpoints
  - pods
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prometheus-k8s
  namespace: openshift-operators-redhat
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-k8s
subjects:
- kind: ServiceAccount
  name: prometheus-k8s
namespace: openshift-operators-redhat
```
1. Install `Cluster Logging Operator` via Admin console  
**Specify namespace `cluster-logging`
**`ElasticSearch`Operator will be installed in `openshift-operators-redhat` namespace
1. Confirm
```
$ oc project
Using project "openshift-operators-redhat" on server "https://api.oc4cluster.tetsuya.local:6443".
$ oc get all
NAME                                          READY   STATUS    RESTARTS   AGE
pod/elasticsearch-operator-57f6bdc955-58gb7   1/1     Running   0          3m35s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/elasticsearch-operator   1/1     1            1           3m37s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/elasticsearch-operator-57f6bdc955   1         1         1       3m35s
$ oc get all -n openshift-logging
NAME                                            READY   STATUS    RESTARTS   AGE
pod/cluster-logging-operator-7b8c88b586-2kckl   1/1     Running   0          10m

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cluster-logging-operator   1/1     1            1           10m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/cluster-logging-operator-7b8c88b586   1         1         1       10m
```
**Now You can see both `Cluster Logging` Operator and `Elasticsearch` Operator in the `openshift-logging` namespace

1. Create `Cluster Logging` via Admin Console -> `Installed Operators` -> `Cluster Logging` -> `Create Instance`
**Change storageClassName appropriately that your cluster have
```
apiVersion: logging.openshift.io/v1
kind: ClusterLogging
metadata:
  name: instance
  namespace: openshift-logging
spec:
  managementState: Managed
  logStore:
    type: elasticsearch
    elasticsearch:
      nodeCount: 3
      redundancyPolicy: SingleRedundancy
      storage:
        storageClassName: rook-ceph-block
        size: 20G
  visualization:
    type: kibana
    kibana:
      replicas: 1
  curation:
    type: curator
    curator:
      schedule: 30 3 * * *
  collection:
    logs:
      type: fluentd
      fluentd: {}
```
1. Click `Create`

