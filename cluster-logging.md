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


