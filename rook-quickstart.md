## Links
https://medium.com/@karansingh010/rook-ceph-deployment-on-openshift-4-2b34dfb6a442
https://blog.openshift.com/rook-container-native-storage-openshift/
https://github.com/rook/rook/blob/master/Documentation/openshift.md

## Ceph Operator Installation Steps
*on worker node*
1. add 100GB disk each node on vmware vspere  
```
before  
[core@worker-0 ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  120G  0 disk
|-sda1   8:1    0    1M  0 part
|-sda2   8:2    0    1G  0 part /boot
`-sda3   8:3    0  119G  0 part /sysroot
sr0     11:0    1 1024M  0 rom
```
```
after  
[core@worker-0 ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  120G  0 disk
|-sda1   8:1    0    1M  0 part
|-sda2   8:2    0    1G  0 part /boot
`-sda3   8:3    0  119G  0 part /sysroot
sdb      8:16   0  100G  0 disk
sr0     11:0    1 1024M  0 rom
```

*on bation node*
1. Clone Repo
```
$ cd ~
$ git clone https://github.com/rook/rook.git
$ cd /rook/cluster/examples/kubernetes/ceph
```

1. create CRD/RBAC/SA 
`oc create -f common.yaml`
1. create operator and Security Constraints
`oc create -f operator-openshift.yaml`
wait for creating operator
```
$ oc get all
NAME                                      READY   STATUS    RESTARTS   AGE
pod/rook-ceph-operator-579b654999-ngc77   1/1     Running   0          31m
pod/rook-discover-2ddrw                   1/1     Running   0          19m
pod/rook-discover-4z6s4                   1/1     Running   0          19m
pod/rook-discover-6bbjg                   1/1     Running   0          19m

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/rook-discover   3         3         3       3            3           <none>          19m

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rook-ceph-operator   1/1     1            1           31m

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/rook-ceph-operator-579b654999   1         1         1       31m
```

1. create cluster
`oc create -f cluster.yaml`
```
$ oc get all
NAME                                                READY   STATUS      RESTARTS   AGE
pod/csi-cephfsplugin-j6v95                          3/3     Running     0          20m
pod/csi-cephfsplugin-jkjg6                          3/3     Running     0          20m
pod/csi-cephfsplugin-p959d                          3/3     Running     0          20m
pod/csi-cephfsplugin-provisioner-78d9994b5d-54xqp   4/4     Running     0          20m
pod/csi-cephfsplugin-provisioner-78d9994b5d-bxf6t   4/4     Running     0          20m
pod/csi-rbdplugin-8cwcp                             3/3     Running     0          20m
pod/csi-rbdplugin-gpkls                             3/3     Running     0          20m
pod/csi-rbdplugin-provisioner-9d99b58f6-blbq4       5/5     Running     0          20m
pod/csi-rbdplugin-provisioner-9d99b58f6-xs55m       5/5     Running     0          20m
pod/csi-rbdplugin-x5hzq                             3/3     Running     0          20m
pod/rook-ceph-mgr-a-7c9864f9b8-hxltx                1/1     Running     0          14m
pod/rook-ceph-mon-a-57785bb7fb-nxw5r                1/1     Running     0          18m
pod/rook-ceph-mon-b-6cf8f7cfdb-7v4qw                1/1     Running     0          17m
pod/rook-ceph-mon-c-8549499ccf-hd4xm                1/1     Running     0          17m
pod/rook-ceph-operator-579b654999-ngc77             1/1     Running     0          52m
pod/rook-ceph-osd-0-6986dd69dd-nt6jg                1/1     Running     0          13m
pod/rook-ceph-osd-1-6967b4b59c-9gp8b                1/1     Running     0          13m
pod/rook-ceph-osd-2-f885db75-89f6p                  1/1     Running     0          13m
pod/rook-ceph-osd-prepare-worker-0-jb5pw            0/1     Completed   0          14m
pod/rook-ceph-osd-prepare-worker-1-bn2s5            0/1     Completed   0          14m
pod/rook-ceph-osd-prepare-worker-2-q8xq7            0/1     Completed   0          14m
pod/rook-ceph-tools-5f5dc75fd5-jhkc8                1/1     Running     0          12m
pod/rook-discover-2ddrw                             1/1     Running     0          40m
pod/rook-discover-4z6s4                             1/1     Running     0          40m
pod/rook-discover-6bbjg                             1/1     Running     0          40m

NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/csi-cephfsplugin-metrics   ClusterIP   172.30.133.52    <none>        8080/TCP,8081/TCP   20m
service/csi-rbdplugin-metrics      ClusterIP   172.30.28.95     <none>        8080/TCP,8081/TCP   20m
service/rook-ceph-mgr              ClusterIP   172.30.54.215    <none>        9283/TCP            14m
service/rook-ceph-mgr-dashboard    ClusterIP   172.30.93.0      <none>        8443/TCP            14m
service/rook-ceph-mon-a            ClusterIP   172.30.167.156   <none>        6789/TCP,3300/TCP   18m
service/rook-ceph-mon-b            ClusterIP   172.30.65.55     <none>        6789/TCP,3300/TCP   17m
service/rook-ceph-mon-c            ClusterIP   172.30.145.16    <none>        6789/TCP,3300/TCP   17m

NAME                              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/csi-cephfsplugin   3         3         3       3            3           <none>          20m
daemonset.apps/csi-rbdplugin      3         3         3       3            3           <none>          20m
daemonset.apps/rook-discover      3         3         3       3            3           <none>          40m

NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/csi-cephfsplugin-provisioner   2/2     2            2           20m
deployment.apps/csi-rbdplugin-provisioner      2/2     2            2           20m
deployment.apps/rook-ceph-mgr-a                1/1     1            1           14m
deployment.apps/rook-ceph-mon-a                1/1     1            1           18m
deployment.apps/rook-ceph-mon-b                1/1     1            1           17m
deployment.apps/rook-ceph-mon-c                1/1     1            1           17m
deployment.apps/rook-ceph-operator             1/1     1            1           52m
deployment.apps/rook-ceph-osd-0                1/1     1            1           13m
deployment.apps/rook-ceph-osd-1                1/1     1            1           13m
deployment.apps/rook-ceph-osd-2                1/1     1            1           13m
deployment.apps/rook-ceph-tools                1/1     1            1           12m

NAME                                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/csi-cephfsplugin-provisioner-78d9994b5d   2         2         2       20m
replicaset.apps/csi-rbdplugin-provisioner-9d99b58f6       2         2         2       20m
replicaset.apps/rook-ceph-mgr-a-7c9864f9b8                1         1         1       14m
replicaset.apps/rook-ceph-mon-a-57785bb7fb                1         1         1       18m
replicaset.apps/rook-ceph-mon-b-6cf8f7cfdb                1         1         1       17m
replicaset.apps/rook-ceph-mon-c-8549499ccf                1         1         1       17m
replicaset.apps/rook-ceph-operator-579b654999             1         1         1       52m
replicaset.apps/rook-ceph-osd-0-6986dd69dd                1         1         1       13m
replicaset.apps/rook-ceph-osd-1-6967b4b59c                1         1         1       13m
replicaset.apps/rook-ceph-osd-2-f885db75                  1         1         1       13m
replicaset.apps/rook-ceph-tools-5f5dc75fd5                1         1         1       12m

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/rook-ceph-osd-prepare-worker-0   1/1           36s        14m
job.batch/rook-ceph-osd-prepare-worker-1   1/1           36s        14m
job.batch/rook-ceph-osd-prepare-worker-2   1/1           41s        14m
```
## Confirm Ceph Cluster
### Install toolbox
`oc create -f toolbox.yaml`  

### Confrim Health State
```
$ oc rsh pod/rook-ceph-tools-5f5dc75fd5-jhkc8
sh-4.2#
sh-4.2# ceph -s
  cluster:
    id:     26231080-d9c5-4027-bd61-397d5e8c191c
    health: HEALTH_WARN
            clock skew detected on mon.b, mon.c

  services:
    mon: 3 daemons, quorum a,b,c (age 4m)
    mgr: a(active, since 3m)
    osd: 3 osds: 3 up (since 2m), 3 in (since 2m)

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   3.0 GiB used, 114 GiB / 117 GiB avail
    pgs:
```
## Create StraogeClass
/rook/cluster/examples/kubernetes/ceph/csi/rbd/storageclass.yaml
```
$ cat storageclass.yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-block
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
    # clusterID is the namespace where the rook cluster is running
    # If you change this namespace, also change the namespace below where the secret namespaces are defined
    clusterID: rook-ceph

    # Ceph pool into which the RBD image shall be created
    pool: replicapool

    # RBD image format. Defaults to "2".
    imageFormat: "2"

    # RBD image features. Available for imageFormat: "2". CSI RBD currently supports only `layering` feature.
    imageFeatures: layering

    # The secrets contain Ceph admin credentials. These are generated automatically by the operator
    # in the same namespace as the cluster.
    csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
    csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
    csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
    csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
    # Specify the filesystem type of the volume. If not specified, csi-provisioner
    # will set default as `ext4`.
    csi.storage.k8s.io/fstype: ext4
# uncomment the following to use rbd-nbd as mounter on supported nodes
#mounter: rbd-nbd
reclaimPolicy: Delete

```
```
$ oc create -f storageclass.yaml
cephblockpool.ceph.rook.io/replicapool created
storageclass.storage.k8s.io/rook-ceph-block created
$ oc get sc
NAME              PROVISIONER                    AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com     27s
```

## Confirm PVC
```
$ oc project sample1
Now using project "sample1" on server "https://api.oc4cluster.tetsuya.local:6443".
** Change namespace 

$ cat pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: samplepvc
spec:
  storageClassName: rook-ceph-block
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 200Mi
```
```
$ oc get pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
samplepvc   Bound    pvc-b9dae451-fbb5-11e9-aa53-005056b609c2   200Mi      RWO            rook-ceph-block   4s
```


## Confirm from Pod
mysql.yaml
```
$ cat mysql.yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: rook-ceph-block
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
    tier: mysql
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: changeme
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

1. Claim from pod
`oc create -f mysql.yaml`
1. confirm
`oc get pods`
```
$ oc get pods
NAME                                   READY   STATUS      RESTARTS   AGE
rook-ceph-mgr-a-5b59cb7458-2q2gc       1/1     Running     0          103m
rook-ceph-mon-d-6bf94599d6-pk5rr       1/1     Running     0          52m
rook-ceph-mon-f-578c956465-w5rgv       1/1     Running     0          104m
rook-ceph-mon-g-766c586468-hjd7d       1/1     Running     0          52m
rook-ceph-osd-0-9bcf6d645-h8stn        1/1     Running     0          66m
rook-ceph-osd-1-69bc54c4b5-5hqws       1/1     Running     0          52m
rook-ceph-osd-2-6db8496845-whc5l       1/1     Running     0          101m
rook-ceph-osd-prepare-worker-2-ptv2f   0/2     Completed   0          103m
rook-ceph-tools                        1/1     Running     0          96m
wordpress-mysql-6887bf844f-x6hwp       1/1     Running     0          3m45s
```

`oc get pvc`
```
$ oc get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
mysql-pv-claim   Bound    pvc-3e574ed7-e908-11e9-b9b7-005056b609c3   20Gi       RWO            rook-ceph-block   5m9s
```

`oc get pv`
```
$ oc get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS      REASON   AGE
pvc-3e574ed7-e908-11e9-b9b7-005056b609c3   20Gi       RWO            Delete           Bound    rook-ceph/mysql-pv-claim   rook-ceph-block            4m15s
```
## Create Filesystem
Before
```
$ oc rsh pod/rook-ceph-tools-5f5dc75fd5-jhkc8
sh-4.2#
sh-4.2# ceph -s
  cluster:
    id:     26231080-d9c5-4027-bd61-397d5e8c191c
    health: HEALTH_WARN
            too few PGs per OSD (4 < min 30)
            clock skew detected on mon.b, mon.c

  services:
    mon: 3 daemons, quorum a,b,c (age 3w)
    mgr: a(active, since 2w)
    osd: 6 osds: 6 up (since 2w), 6 in (since 2w)

  data:
    pools:   1 pools, 8 pgs
    objects: 109 objects, 288 MiB
    usage:   6.9 GiB used, 407 GiB / 414 GiB avail
    pgs:     8 active+clean

sh-4.2#
sh-4.2# ceph df
RAW STORAGE:
    CLASS     SIZE        AVAIL       USED        RAW USED     %RAW USED
    hdd       414 GiB     407 GiB     894 MiB      6.9 GiB          1.66
    TOTAL     414 GiB     407 GiB     894 MiB      6.9 GiB          1.66

POOLS:
    POOL            ID     STORED      OBJECTS     USED        %USED     MAX AVAIL
    replicapool      1     263 MiB         109     794 MiB      0.20       127 GiB
```
Create filesystem
```
$ cd ./cluster/examples/kubernetes/ceph
$ oc create -f filesystem.yaml
```
Progress
```
$ oc get pod -l app=rook-ceph-mds
NAME                                    READY   STATUS     RESTARTS   AGE
rook-ceph-mds-myfs-a-6b8f55bbd-qw66c    0/1     Init:0/1   0          5s
rook-ceph-mds-myfs-b-55696b9dd5-r5fbx   0/1     Init:0/1   0          4s
```
