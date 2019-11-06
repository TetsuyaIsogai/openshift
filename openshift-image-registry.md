## THIS LAB IS STILL PENDING 


## Prerequesite
- openshift 4.2 cluster

## Before
```
$ oc get configs.imageregistry.operator.openshift.io -oyaml
apiVersion: v1
items:
- apiVersion: imageregistry.operator.openshift.io/v1
  kind: Config
  metadata:
    creationTimestamp: "2019-10-30T10:21:35Z"
    finalizers:
    - imageregistry.operator.openshift.io/finalizer
    generation: 2
    name: cluster
    resourceVersion: "891689"
    selfLink: /apis/imageregistry.operator.openshift.io/v1/configs/cluster
    uid: 0d5d4812-faff-11e9-b81e-005056b609c2
  spec:
    defaultRoute: false
    httpSecret: 870c33867a8e29c625038345b60f4c1aced0903828d7f1625355314a20215adc01fb6328ecc848343d6d903694e6485b827db6e1c4263d7da2b1f6f924b26605
    logging: 2
    managementState: Managed
    proxy:
      http: ""
      https: ""
      noProxy: ""
    readOnly: false
    replicas: 1
    requests:
      read:
        maxInQueue: 0
        maxRunning: 0
        maxWaitInQueue: 0s
      write:
        maxInQueue: 0
        maxRunning: 0
        maxWaitInQueue: 0s
    storage:
      emptyDir: {}
  status:
    conditions:
    - lastTransitionTime: "2019-10-30T12:03:56Z"
      message: The registry is ready
      reason: Ready
      status: "True"
      type: Available
    - lastTransitionTime: "2019-11-01T06:41:47Z"
      message: The registry is ready
      reason: Ready
      status: "False"
      type: Progressing
    - lastTransitionTime: "2019-10-30T11:54:01Z"
      status: "False"
      type: Degraded
    - lastTransitionTime: "2019-10-30T10:21:35Z"
      status: "False"
      type: Removed
    - lastTransitionTime: "2019-10-30T11:54:01Z"
      message: EmptyDir storage successfully created
      reason: Creation Successful
      status: "True"
      type: StorageExists
    observedGeneration: 2
    readyReplicas: 0
    storage:
      emptyDir: {}
    storageManaged: false
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
  ```
  
  ## After
  $ oc get configs.imageregistry.operator.openshift.io -oyaml
apiVersion: v1
items:
- apiVersion: imageregistry.operator.openshift.io/v1
  kind: Config
  metadata:
    creationTimestamp: "2019-10-30T10:21:35Z"
    finalizers:
    - imageregistry.operator.openshift.io/finalizer
    generation: 4
    name: cluster
    resourceVersion: "4085993"
    selfLink: /apis/imageregistry.operator.openshift.io/v1/configs/cluster
    uid: 0d5d4812-faff-11e9-b81e-005056b609c2
  spec:
    defaultRoute: false
    httpSecret: 870c33867a8e29c625038345b60f4c1aced0903828d7f1625355314a20215adc01fb6328ecc848343d6d903694e6485b827db6e1c4263d7da2b1f6f924b26605
    logging: 2
    managementState: Managed
    proxy:
      http: ""
      https: ""
      noProxy: ""
    readOnly: false
    replicas: 1
    requests:
      read:
        maxInQueue: 0
        maxRunning: 0
        maxWaitInQueue: 0s
      write:
        maxInQueue: 0
        maxRunning: 0
        maxWaitInQueue: 0s
    storage:
      pvc:
        claim: image-registry-storage
  status:
    conditions:
    - lastTransitionTime: "2019-10-30T12:03:56Z"
      message: The registry has minimum availability
      reason: MinimumAvailability
      status: "True"
      type: Available
    - lastTransitionTime: "2019-11-06T03:57:21Z"
      message: The deployment has not completed
      reason: DeploymentNotCompleted
      status: "True"
      type: Progressing
    - lastTransitionTime: "2019-10-30T11:54:01Z"
      status: "False"
      type: Degraded
    - lastTransitionTime: "2019-10-30T10:21:35Z"
      status: "False"
      type: Removed
    - lastTransitionTime: "2019-11-06T03:57:21Z"
      reason: PVC Exists
      status: "True"
      type: StorageExists
    observedGeneration: 4
    readyReplicas: 0
    storage:
      emptyDir: {}
      pvc:
        claim: image-registry-storage
    storageManaged: true
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
  
  -> Create pvc automatically
  
## Modify pvc
### Before
```
$ oc get pvc -n openshift-image-registry
NAME                     STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS      AGE
image-registry-storage   Pending                                      rook-ceph-block   91s
```
- StorageClass is set automatically
- You need to modify `volumeMode` from `FileSystem` to `Block:x
`
```
$ oc get pvc -n openshift-image-registry image-registry-storage -oyaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    imageregistry.openshift.io: "true"
    volume.beta.kubernetes.io/storage-provisioner: rook-ceph.rbd.csi.ceph.com
  creationTimestamp: "2019-11-06T03:57:20Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: image-registry-storage
  namespace: openshift-image-registry
  resourceVersion: "4085969"
  selfLink: /api/v1/namespaces/openshift-image-registry/persistentvolumeclaims/image-registry-storage
  uid: 8830b9a9-0049-11ea-9441-005056b609c2
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: rook-ceph-block
  volumeMode: Filesystem
status:
  phase: Pending
```
```
$  oc get pvc -n openshift-image-registry image-registry-storage -oyaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    imageregistry.openshift.io: "true"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{"imageregistry.openshift.io":"true","volume.beta.kubernetes.io/storage-provisioner":"rook-ceph.rbd.csi.ceph.com"},"creationTimestamp":"2019-11-06T03:57:20Z","finalizers":["kubernetes.io/pvc-protection"],"name":"image-registry-storage","namespace":"openshift-image-registry","resourceVersion":"4085969","selfLink":"/api/v1/namespaces/openshift-image-registry/persistentvolumeclaims/image-registry-storage","uid":"8830b9a9-0049-11ea-9441-005056b609c2"},"spec":{"accessModes":["ReadWriteMany"],"resources":{"requests":{"storage":"100Gi"}},"storageClassName":"rook-ceph-block","volumeMode":"Block"},"status":{"phase":"Pending"}}
    pv.kubernetes.io/bind-completed: "yes"
    pv.kubernetes.io/bound-by-controller: "yes"
    volume.beta.kubernetes.io/storage-provisioner: rook-ceph.rbd.csi.ceph.com
  creationTimestamp: "2019-11-06T04:11:14Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: image-registry-storage
  namespace: openshift-image-registry
  resourceVersion: "4092395"
  selfLink: /api/v1/namespaces/openshift-image-registry/persistentvolumeclaims/image-registry-storage
  uid: 79103b7c-004b-11ea-b435-005056b609c3
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: rook-ceph-block
  volumeMode: Block
  volumeName: pvc-79103b7c-004b-11ea-b435-005056b609c3
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 100Gi
  phase: Pending
```

- delete pvc
- delete image-registry pod
- after that, `pvc` will bound and image-registry Pod will bring up automatically

```
$ oc get all -n openshift-image-registry
NAME                                                   READY   STATUS              RESTARTS   AGE
pod/cluster-image-registry-operator-696db565bb-rzkrj   2/2     Running             0          4d21h
pod/image-registry-74cdf6f7f9-8tz4l                    0/1     ContainerCreating   0          6m22s
pod/image-registry-fd465ddf5-7g4ww                     1/1     Running             0          4d21h
pod/node-ca-66g5h                                      1/1     Running             0          6d16h
pod/node-ca-ccrcr                                      1/1     Running             0          6d16h
pod/node-ca-dcxjr                                      1/1     Running             0          6d16h
pod/node-ca-mqcnj                                      1/1     Running             0          6d16h
pod/node-ca-p65w6                                      1/1     Running             0          6d16h
pod/node-ca-qqhdk                                      1/1     Running             0          6d16h

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/image-registry   ClusterIP   172.30.199.232   <none>        5000/TCP   6d16h

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/node-ca   6         6         6       6            6           kubernetes.io/os=linux   6d16h

NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cluster-image-registry-operator   1/1     1            1           6d18h
deployment.apps/image-registry                    1/1     1            1           6d16h

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/cluster-image-registry-operator-696db565bb   1         1         1       4d21h
replicaset.apps/cluster-image-registry-operator-7c4b9db98f   0         0         0       6d18h
replicaset.apps/image-registry-56f566f8f9                    0         0         0       6d16h
replicaset.apps/image-registry-6fbbc5fb9b                    0         0         0       6d16h
replicaset.apps/image-registry-74cdf6f7f9                    1         1         0       18m
replicaset.apps/image-registry-cff895f7b                     0         0         0       6d16h
replicaset.apps/image-registry-fd465ddf5                     1         1         1       4d21h
```


