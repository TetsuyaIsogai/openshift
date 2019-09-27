## Prerequisite
- use same machine to execute 

## Procedure
1. Add dhcp entry in dnsmasq

1. create virtual machine for worker node you want to add in cluster
1. install RHEL7.6
  - Lang: English
  - TZ: UTC 00:00 # allign with existing cluster
  - the other options are default
1. Enable `ens192`
1. Subscription manager register
```
[root@worker-2 ~]# subscription-manager register --username=tty1028 --password=FU62iNc-iiK679u
登録中: subscription.rhsm.redhat.com:443/subscription
このシステムは、次の ID で登録されました: b372a5c7-e93a-40b3-88ad-62668a71e95e
登録したシステム名: worker-2
```
1. subscription manager refresh
```
[root@worker-2 ~]# subscription-manager refresh
ローカルデータがすべて更新されました
```
1. subscription manager poolid
#pool id is on the `https://access.redhat.com/management/subscriptions/`
```
subscription-manager attach --pool=<pool_id>
```
1. 



Links
赤帽エンジニアブログ https://rheb.hatenablog.com/entry/openshift41-baremetal-upi

Add RHEL 7 worker node
https://docs.openshift.com/container-platform/4.1/machine_management/adding-rhel-compute.html
