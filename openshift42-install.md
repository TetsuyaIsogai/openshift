## Prerequisite
- ESX 6.7 U2 or later
- Server resource
  - 1 bootstrap machine
  - 3 Master node
  - 3 worker node


## Download
https://cloud.redhat.com/openshift/install/vsphere/user-provisioned
OR
https://access.redhat.com/downloads/content/290/ver=4.2/rhel---8/4.2.0/x86_64/product-software

- RHCOS OVA (700MB)
- pull-secret
- openshift-client.tar.gz
- openshift-install.tar.gz

## Machine Parameters
- Hostname/IP/Mac
/etc/dnsmasq.conf
```
port=53
domain-needed
bogus-priv
resolv-file=/etc/resolv.dnsmasq
no-poll
local=/tetsuya.local/
address=/apps.oc4cluster.tetsuya.local/10.0.1.10
user=dnsmasq
group=dnsmasq
no-dhcp-interface=ens224
expand-hosts
domain=oc4cluster.tetsuya.local
dhcp-range=10.0.1.100,10.0.1.200,255.255.0.0,12h
dhcp-host=00:50:56:b6:09:c1,bootstrap,10.0.1.200
dhcp-host=00:50:56:b6:09:c2,master-0,10.0.1.100
dhcp-host=00:50:56:b6:09:c3,master-1,10.0.1.101
dhcp-host=00:50:56:b6:09:c4,master-2,10.0.1.102
dhcp-host=00:50:56:b6:09:c5,worker-0,10.0.1.110
dhcp-host=00:50:56:b6:09:c6,worker-1,10.0.1.111
dhcp-host=00:50:56:b6:09:c7,worker-2,10.0.1.112
dhcp-option=option:dns-server,10.0.1.10
dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases
srv-host=_etcd-server-ssl._tcp.oc4cluster.tetsuya.local,etcd-0.oc4cluster.tetsuya.local,2380,0,10
srv-host=_etcd-server-ssl._tcp.oc4cluster.tetsuya.local,etcd-1.o
cac4cluster.tetsuya.local,2380,0,10
srv-host=_etcd-server-ssl._tcp.oc4cluster.tetsuya.local,etcd-2.oc4cluster.tetsuya.local,2380,0,10
log-dhcp
log-facility=/var/log/dnsmasq.log
conf-dir=/etc/dnsmasq.d,.rpmnew,.rpmsave,.rpmorig
```
** Acceptable Mac address in vSphere is 
00:05:69:00:00:00 to 00:05:69:FF:FF:FF
00:0c:29:00:00:00 to 00:0c:29:FF:FF:FF
00:1c:14:00:00:00 to 00:1c:14:FF:FF:FF
00:50:56:00:00:00 to 00:50:56:FF:FF:FF

- DNS entries
```
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# for OCP4
15.210.188.93   api
10.0.1.10       api-int
10.0.1.100      etcd-0
10.0.1.101      etcd-1
10.0.1.102      etcd-2

#api.oc4cluster.tetsuya.local    15.210.188.93
#api-int.oc4cluster.tetsuya.local       10.0.1.10
*.apps.oc4cluster.tetsuya.local 10.0.1.10
etcd-0.oc4cluster.tetsuya.local 10.0.1.100
etcd-1.oc4cluster.tetsuya.local 10.0.1.101
etcd-2.oc4cluster.tetsuya.local 10.0.1.102
```

- Load Balancer (HAProxy)
** Add entry in `/etc/haproxy.cfg`  
```
frontend K8s-api
    bind *:6443
    option tcplog
    mode tcp
    default_backend     api-6443

frontend Machine-config
    bind *:22623
    option tcplog
    mode tcp
    default_backend     config-22623

frontend Ingress-http
    bind *:80
    option tcplog
    mode tcp
    default_backend http-80

frontend Ingress-https
    bind *:443
    option tcplog
    mode tcp
    default_backend     https-443


backend api-6443
    mode tcp
    balance     roundrobin
    option  ssl-hello-chk
    server  bootstrap bootstrap.oc4cluster.tetsuya.local:6443 check
    server  master-0 master-0.oc4cluster.tetsuya.local:6443 check
    server  master-1 master-1.oc4cluster.tetsuya.local:6443 check
    server  master-2 master-2.oc4cluster.tetsuya.local:6443 check

backend config-22623
    mode tcp
    balance     roundrobin
    server  bootstrap bootstrap.oc4cluster.tetsuya.local:22623 check
    server  master-0 master-0.oc4cluster.tetsuya.local:22623 check
    server  master-1 master-1.oc4cluster.tetsuya.local:22623 check
    server  master-2 master-2.oc4cluster.tetsuya.local:22623 check

backend http-80
    mode tcp
    balance     roundrobin
    server  worker-0 worker-0.oc4cluster.tetsuya.local:80 check
    server  worker-1 worker-1.oc4cluster.tetsuya.local:80 check

backend https-443
    mode tcp
    balance     roundrobin
    option      ssl-hello-chk
    server  worker-0 worker-0.oc4cluster.tetsuya.local:443 check
    server  worker-1 worker-1.oc4cluster.tetsuya.local:443 check
```
## install-config.yaml
```
apiVersion: v1
baseDomain: tetsuya.local
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: oc4cluster
platform:
  vsphere:
    vcenter: 15.210.188.92
    username: administrator
    password: P@ssW0rd
    datacenter: Hewlett-Packard
    defaultDatastore: datastore1
pullSecret: '{"auths": ...'   # ' is mandatory, don't forget
sshKey: 'ssh-rsa AAAA ...'    # ' is mandatory. don't forget

```
Files after executing `./openshift-install create manifests --dir=/home/newgen/ocp42`  
```
04-openshift-machine-config-operator.yaml  cluster-network-02-config.yml     etcd-host-service.yaml                 etcd-signer-secret.yaml
cloud-provider-config.yaml                 cluster-proxy-01-config.yaml      etcd-metric-client-secret.yaml         kube-cloud-config.yaml
cluster-config.yaml                        cluster-scheduler-02-config.yml   etcd-metric-serving-ca-configmap.yaml  kube-system-configmap-root-ca.yaml
cluster-dns-02-config.yml                  cvo-overrides.yaml                etcd-metric-signer-secret.yaml         machine-config-server-tls-secret.yaml
cluster-infrastructure-02-config.yml       etcd-ca-bundle-configmap.yaml     etcd-namespace.yaml                    openshift-config-secret-pull-secret.yaml
cluster-ingress-02-config.yml              etcd-client-secret.yaml           etcd-service.yaml
cluster-network-01-crd.yml                 etcd-host-service-endpoints.yaml  etcd-serving-ca-configmap.yaml
```

Directories after executing `./openshift-install create ignition-configs --dir=/home/newgen/ocp42`  
```
.
├── README.md
├── auth
│   ├── kubeadmin-password
│   └── kubeconfig
├── bootstrap.ign
├── master.ign
├── metadata.json
├── openshift-client-linux-4.2.0.tar.gz
├── openshift-install
├── openshift-install-linux-4.2.0.tar.gz
├── pull-secret.txt
└── worker.ign
```

## append-bootstrap.ign
```
$ cat append-bootstrap.ign
{
  "ignition": {
    "config": {
      "append": [
        {
          "source": "http://10.0.1.10:8008/ocp/rhcos/ignitions/bootstrap.ign",
          "verification": {}
        }
      ]
    },
    "timeouts": {},
    "version": "2.1.0"
  },
  "networkd": {},
  "passwd": {},
  "storage": {},
  "systemd": {}
}
```

## OVA 
1. Deploy OVA file
1. Clone OVA to Virtual Machine
1. Edit Virtual Hardware Options, add parameter (base64)
1. Edit Memory, Disk, CPU size appropriatly each machine
   Note: Memory reservation is needed
1. Edit MAC address adjust with `dnsmasq.conf`
1. Boot the virtual Machines



## Install openshift client
$ tar xvf openshift-client-linux-4.2.0.tar.gz
$ sudo mv oc /usr/bin/oc
$ oc version
Client Version: openshift-clients-4.2.0-201910041700

## Create Cluster
$ openshift-install --dir=/home/newgen/ocp42 wait-for bootstrap-complete --log-level=info
INFO Waiting up to 30m0s for the Kubernetes API at https://api.oc4cluster.tetsuya.local:6443...

## Troubleshooting
$ ssh core@bootstrap -i .ssh/ocp42_rsa
[core@bootstrap ~]$ journalctl -b -f -u bootkube.service



