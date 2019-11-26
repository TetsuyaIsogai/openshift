## Link
https://access.redhat.com/documentation/ja-jp/openshift_container_platform/4.1/html/logging/efk-logging-deploying

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
$ oc get packagemanifest elasticsearch-operator -n openshift-marketplace -o jsonpath='{.status.channels[].currentCSV}'
elasticsearch-operator.4.2.5-201911121709
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
  source: "elasticsearch"
  sourceNamespace: "openshift-operators-redhat"
  name: "elasticsearch-operator"
  startingCSV: "elasticsearch-operator.4.2.5-201911121709" 
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
1. Confirm
```
$ oc project openshift-logging
Now using project "openshift-logging" on server "https://api.oc4cluster.tetsuya.local:6443".
$ oc get all
NAME                                            READY   STATUS              RESTARTS   AGE
pod/cluster-logging-operator-7b8c88b586-kg6vx   0/1     ContainerCreating   0          27s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cluster-logging-operator   0/1     1            0           28s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/cluster-logging-operator-7b8c88b586   1         1         0       27s
```
**Wait until status is healty on Admin Console
```
$ oc get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/cluster-logging-operator-7b8c88b586-kg6vx   1/1     Running   0          4m26s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cluster-logging-operator   1/1     1            1           4m27s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/cluster-logging-operator-7b8c88b586   1         1         1       4m26s
```
1. Create Cluster logging via Admin Console -> `Custom Resource Definitions` -> `Cluster Logging` -> `Actions` -> `View Instance` -> `Create Cluster Logging`  by creating `CR` below
```
apiVersion: "logging.openshift.io/v1"
kind: "ClusterLogging"
metadata:
  name: "instance" 
  namespace: "openshift-logging"
spec:
  managementState: "Managed"  
  logStore:
    type: "elasticsearch"  
    elasticsearch:
      nodeCount: 3 
      storage:
        storageClassName: gp2
        size: 200G
      redundancyPolicy: "SingleRedundancy"
  visualization:
    type: "kibana"  
    kibana:
      replicas: 1
  curation:
    type: "curator"  
    curator:
      schedule: "30 3 * * *"
  collection:
    logs:
      type: "fluentd"  
      fluentd: {}
```
1. Click `Create`

