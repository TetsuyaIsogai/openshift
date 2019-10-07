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
#```
#$ cd ~
#$ git clone https://github.com/rook/rook.git
#$ cd /rook/cluster/examples/kubernetes/ceph
#```
```
git clone https://github.com/ksingh7/ocp4-rook.git
cd ocp4-rook/ceph/
```

1. create SecurityContextConstraints 
`oc create -f scc.yaml`
1. create operator
`oc create -f operator.yaml`
1. create cluster
`oc create -f cluster.yaml`

## Confirmation Ceph Cluster
```
$ oc exec -it rook-ceph-tools bash
bash: warning: setlocale: LC_CTYPE: cannot change locale (en_US.UTF-8): No such file or directory
bash: warning: setlocale: LC_COLLATE: cannot change locale (en_US.UTF-8): No such file or directory
bash: warning: setlocale: LC_MESSAGES: cannot change locale (en_US.UTF-8): No such file or directory
bash: warning: setlocale: LC_NUMERIC: cannot change locale (en_US.UTF-8): No such file or directory
bash: warning: setlocale: LC_TIME: cannot change locale (en_US.UTF-8): No such file or directory
[root@rook-ceph-tools /]#
[root@rook-ceph-tools /]# ceph -s
  cluster:
    id:     aaf57b47-271b-4904-8ecc-2cc5d38fda4b
    health: HEALTH_WARN
            clock skew detected on mon.d, mon.g

  services:
    mon: 3 daemons, quorum f,d,g
    mgr: a(active)
    osd: 3 osds: 3 up, 3 in

  data:
    pools:   1 pools, 100 pgs
    objects: 0  objects, 0 B
    usage:   3.0 GiB used, 297 GiB / 300 GiB avail
    pgs:     100 active+clean
```

## Confirmation from Pod
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

