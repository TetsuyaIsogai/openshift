## Link
https://docs.openshift.com/container-platform/4.2/logging/cluster-logging-deploying.html
https://docs.openshift.com/container-platform/4.1/logging/config/efk-logging-elasticsearch.html


## Prerequisite
OCP4.2

## Procedure
1. Create Namespace for `openshift-operators-redhat`  
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
      resources:
        limits:
          cpu: "4000m"
          memory: "4Gi"
        requests:
          cpu: "100m"
          memory: "1Gi"
      nodeCount: 3
      redundancyPolicy: ZeroRedundancy
      storage: {}
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
```
[newgen@bastion01 cluster-logging]$ oc project openshift-logging
Now using project "openshift-logging" on server "https://api.oc4cluster.tetsuya.local:6443".
[newgen@bastion01 cluster-logging]$
[newgen@bastion01 cluster-logging]$
[newgen@bastion01 cluster-logging]$
[newgen@bastion01 cluster-logging]$ oc get all
NAME                                                READY   STATUS              RESTARTS   AGE
pod/cluster-logging-operator-f5c55f79c-lc6f9        1/1     Running             0          8m47s
pod/elasticsearch-cdm-usj0hoy0-1-dcbcf5969-cpw64    0/2     ContainerCreating   0          5m8s
pod/elasticsearch-cdm-usj0hoy0-2-8fc7dbd75-vxv68    0/2     ContainerCreating   0          4m6s
pod/elasticsearch-cdm-usj0hoy0-3-86c85998ff-bz49m   0/2     ContainerCreating   0          3m5s
pod/fluentd-8pqgn                                   0/1     ContainerCreating   0          5m7s
pod/fluentd-9kxfp                                   0/1     ContainerCreating   0          5m7s
pod/fluentd-bkx95                                   0/1     ContainerCreating   0          5m7s
pod/fluentd-cjjcb                                   0/1     ContainerCreating   0          5m7s
pod/fluentd-sj5l6                                   0/1     ContainerCreating   0          5m7s
pod/fluentd-xr2cd                                   0/1     ContainerCreating   0          5m7s
pod/kibana-84cdbf9cbd-mgnxh                         0/2     ContainerCreating   0          5m8s

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/elasticsearch           ClusterIP   172.30.50.241    <none>        9200/TCP    5m9s
service/elasticsearch-cluster   ClusterIP   172.30.199.88    <none>        9300/TCP    5m9s
service/elasticsearch-metrics   ClusterIP   172.30.11.84     <none>        60000/TCP   5m8s
service/fluentd                 ClusterIP   172.30.239.58    <none>        24231/TCP   5m7s
service/kibana                  ClusterIP   172.30.235.115   <none>        443/TCP     5m8s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/fluentd   6         6         0       6            0           kubernetes.io/os=linux   5m7s

NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cluster-logging-operator       1/1     1            1           8m49s
deployment.apps/elasticsearch-cdm-usj0hoy0-1   0/1     1            0           5m8s
deployment.apps/elasticsearch-cdm-usj0hoy0-2   0/1     1            0           4m6s
deployment.apps/elasticsearch-cdm-usj0hoy0-3   0/1     1            0           3m5s
deployment.apps/kibana                         0/1     1            0           5m8s

NAME                                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/cluster-logging-operator-f5c55f79c        1         1         1       8m47s
replicaset.apps/elasticsearch-cdm-usj0hoy0-1-dcbcf5969    1         1         0       5m8s
replicaset.apps/elasticsearch-cdm-usj0hoy0-2-8fc7dbd75    1         1         0       4m6s
replicaset.apps/elasticsearch-cdm-usj0hoy0-3-86c85998ff   1         1         0       3m5s
replicaset.apps/kibana-84cdbf9cbd                         1         1         0       5m8s

NAME                    SCHEDULE     SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/curator   30 3 * * *   False     0        <none>          5m8s

NAME                              HOST/PORT                                                PATH   SERVICES   PORT    TERMINATION          WILDCARD
route.route.openshift.io/kibana   kibana-openshift-logging.apps.oc4cluster.tetsuya.local          kibana     <all>   reencrypt/Redirect   None

```

↓Fail
```
$ oc get clusterlogging instance -oyaml
apiVersion: logging.openshift.io/v1
kind: ClusterLogging
metadata:
  creationTimestamp: "2019-11-26T07:37:21Z"
  generation: 1
  name: instance
  namespace: openshift-logging
  resourceVersion: "17567630"
  selfLink: /apis/logging.openshift.io/v1/namespaces/openshift-logging/clusterloggings/instance
  uid: 9496426c-101f-11ea-aaa2-005056b609c3
spec:
  collection:
    logs:
      fluentd: {}
      type: fluentd
  curation:
    curator:
      schedule: 30 3 * * *
    type: curator
  logStore:
    elasticsearch:
      nodeCount: 3
      redundancyPolicy: ZeroRedundancy
      storage: {}
    type: elasticsearch
  managementState: Managed
  visualization:
    kibana:
      replicas: 1
    type: kibana
```

↓Fail
```
$ oc get pods
NAME                                            READY   STATUS        RESTARTS   AGE
cluster-logging-operator-f5c55f79c-2d5d7        0/1     Terminating   0          25m
cluster-logging-operator-f5c55f79c-55mx6        1/1     Running       0          19m
elasticsearch-cdm-usj0hoy0-1-dcbcf5969-7j7lt    0/2     Terminating   0          25m
elasticsearch-cdm-usj0hoy0-1-dcbcf5969-fjrc4    1/2     Running       0          19m
elasticsearch-cdm-usj0hoy0-2-8fc7dbd75-7jzjg    1/2     Terminating   0          114m
elasticsearch-cdm-usj0hoy0-2-8fc7dbd75-b52xq    2/2     Terminating   0          43m
elasticsearch-cdm-usj0hoy0-2-8fc7dbd75-ct6hp    0/2     Pending       0          10m
elasticsearch-cdm-usj0hoy0-3-86c85998ff-l48z5   1/2     Terminating   0          114m
elasticsearch-cdm-usj0hoy0-3-86c85998ff-zlk6j   1/2     Running       0          25m
fluentd-8pqgn                                   1/1     Running       0          3h2m
fluentd-9kxfp                                   1/1     Running       0          3h2m
fluentd-bkx95                                   1/1     Running       0          3h2m
fluentd-cjjcb                                   1/1     Running       0          3h2m
fluentd-sj5l6                                   1/1     Running       0          3h2m
fluentd-xr2cd                                   1/1     Running       0          3h2m
kibana-84cdbf9cbd-bc6z9                         2/2     Running       0          25m
```
