## Prerequisites
- OCP4.1 or higer cluster is ready on VMware
- bootstap node(?)
- install jq


## Procedures
1. Create RHCOS woker node (not powered on)
* Choose 'BIOS' for Boot Option (allign with ISO image)
* Coose 'CoreOS Linux(64bit)'
1. DHCP settings
```
--- /etc/dnsmasq.conf on bation01
[newgen@bastion01 rhcos]$ cat /etc/dnsmasq.conf
port=53
...
dhcp-host=00:50:56:b6:39:f4,worker-2,10.0.1.112 #add for new worker node
```
2. 
1. Boot new worker node from vSphere
1. Enter boot image / ignition file URL
1. 

## Links
赤帽エンジニアブログ
https://rheb.hatenablog.com/entry/openshift41-baremetal-upi

一応
https://docs.openshift.com/container-platform/4.1/machine_management/adding-rhel-compute.html
