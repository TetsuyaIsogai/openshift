## Links
https://medium.com/@karansingh010/rook-ceph-deployment-on-openshift-4-2b34dfb6a442
https://blog.openshift.com/rook-container-native-storage-openshift/
https://github.com/rook/rook/blob/master/Documentation/openshift.md

## Installation Steps
*on worker node*
* mkdir /var/lib/rook
* mkdir /var/pv
* chomod 777 /var/lib/rook
* chmod 777 /var/pv

*on bation node*
1. Clone Repo
```
$ cd ~
$ git clone https://github.com/rook/rook.git
$ cd /rook/cluster/examples/kubernetes/ceph
```
1. create project, serviceaccounts, rbac, etc  
`oc create -f common.yaml`
1. modify `operator-openshift.yaml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rook-ceph-operator
  namespace: rook-ceph
  labels:
    operator: rook
    storage-backend: ceph
spec:
  selector:
    matchLabels:
      app: rook-ceph-operator
  replicas: 1
  template:
    metadata:
      labels:
        app: rook-ceph-operator
    spec:
      serviceAccountName: rook-ceph-system
      containers:
      - name: rook-ceph-operator
        image: rook/ceph:master
        args: ["ceph", "operator"]
        volumeMounts:
        - mountPath: /var/lib/rook
          name: rook-config
        - mountPath: /etc/ceph
          name: default-config-dir
        env:
        - name: ROOK_CURRENT_NAMESPACE_ONLY
          value: "true"
        - name: FLEXVOLUME_DIR_PATH
          value: "/etc/kubernetes/kubelet-plugins/volume/exec"
        - name: ROOK_ALLOW_MULTIPLE_FILESYSTEMS
          value: "false"
        - name: ROOK_LOG_LEVEL
          value: "INFO"
        - name: ROOK_MON_HEALTHCHECK_INTERVAL
          value: "45s"
        - name: ROOK_MON_OUT_TIMEOUT
          value: "600s"
        - name: ROOK_DISCOVER_DEVICES_INTERVAL
          value: "60m"
        - name: ROOK_HOSTPATH_REQUIRES_PRIVILEGED
          value: "true"
        - name: ROOK_ENABLE_SELINUX_RELABELING
          value: "true"
        - name: ROOK_ENABLE_FSGROUP
          value: "true"
        - name: ROOK_DISABLE_DEVICE_HOTPLUG
          value: "false"
        - name: ROOK_ENABLE_FLEX_DRIVER
          value: "false"
        - name: ROOK_ENABLE_DISCOVERY_DAEMON
          value: "true"
        - name: ROOK_ENABLE_MACHINE_DISRUPTION_BUDGET
          value: "false"
        - name: ROOK_CSI_ENABLE_CEPHFS
          value: "true"
        - name: ROOK_CSI_ENABLE_RBD
          value: "true"
        - name: ROOK_CSI_ENABLE_GRPC_METRICS
          value: "true"
        - name: ROOK_HOSTPATH_REQUIRES_PRIVILEGED
          value: "true"
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      volumes:
      - name: rook-config
        hostPath:                 #modify emptydir -> hostPath
          path: /var/pv/rook-config #modify emptydir -> hostPath
      - name: default-config-dir
        hostPath:                 #modify emptydir -> hostPath
          path: /var/pv/rook-config-dir #modify emptydir -> hostPath
```
1. create operator  
`oc create -f operator-openshift.yaml`
--confirm
```
[newgen@bastion01 ceph]$ oc get po
NAME                                 READY   STATUS    RESTARTS   AGE
rook-ceph-operator-dcbc78bcb-ddtbc   1/1     Running   0          23m
rook-discover-g8g97                  1/1     Running   0          23m
rook-discover-nqcdk                  1/1     Running   0          23m
rook-discover-tf2b8                  1/1     Running   0          23m
```
1. create object storage  
`oc create -f object-openshift.yaml`
