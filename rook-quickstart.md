## Links
https://medium.com/@karansingh010/rook-ceph-deployment-on-openshift-4-2b34dfb6a442
https://blog.openshift.com/rook-container-native-storage-openshift/
https://github.com/rook/rook/blob/master/Documentation/openshift.md

## Ceph Operator Installation Steps
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
$ oc get all
NAME                                     READY   STATUS    RESTARTS   AGE
pod/rook-ceph-operator-dcbc78bcb-t6tkb   1/1     Running   0          47s
pod/rook-discover-9c6f8                  1/1     Running   0          37s
pod/rook-discover-cs24z                  1/1     Running   0          37s
pod/rook-discover-snrh5                  1/1     Running   0          37s

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/rook-discover   3         3         3       3            3           <none>          37s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rook-ceph-operator   1/1     1            1           26s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/rook-ceph-operator-dcbc78bcb   1         1         1       47s
```

## create cluster
`$ oc create -f cluster.yaml`

```
$ oc get all
NAME                                     READY   STATUS    RESTARTS   AGE
pod/csi-cephfsplugin-dx4d9               3/3     Running   0          53s
pod/csi-cephfsplugin-gmqvt               3/3     Running   0          53s
pod/csi-cephfsplugin-gxrbq               3/3     Running   0          53s
pod/csi-cephfsplugin-provisioner-0       4/4     Running   0          53s
pod/csi-rbdplugin-n7wsh                  3/3     Running   0          53s
pod/csi-rbdplugin-pm4tf                  3/3     Running   0          53s
pod/csi-rbdplugin-provisioner-0          5/5     Running   0          53s
pod/csi-rbdplugin-zdcz8                  3/3     Running   0          53s
pod/rook-ceph-operator-dcbc78bcb-t6tkb   1/1     Running   0          2m21s
pod/rook-discover-9c6f8                  1/1     Running   0          2m11s
pod/rook-discover-cs24z                  1/1     Running   0          2m11s
pod/rook-discover-snrh5                  1/1     Running   0          2m11s

NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/csi-cephfsplugin-metrics   ClusterIP   172.30.73.188   <none>        8080/TCP,8081/TCP   53s
service/csi-rbdplugin-metrics      ClusterIP   172.30.180.11   <none>        8080/TCP,8081/TCP   53s

NAME                              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/csi-cephfsplugin   3         3         3       3            3           <none>          53s
daemonset.apps/csi-rbdplugin      3         3         3       3            3           <none>          54s
daemonset.apps/rook-discover      3         3         3       3            3           <none>          2m11s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rook-ceph-operator   1/1     1            1           2m

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/rook-ceph-operator-dcbc78bcb   1         1         1       2m21s

NAME                                            READY   AGE
statefulset.apps/csi-cephfsplugin-provisioner   1/1     53s
statefulset.apps/csi-rbdplugin-provisioner      1/1     54s
```

