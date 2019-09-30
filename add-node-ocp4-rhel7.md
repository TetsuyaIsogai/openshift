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
1. subscription manager attach poolid
#pool id is on the `https://access.redhat.com/management/subscriptions/`
```
subscription-manager attach --pool=<pool_id>
```
#both Redhat Developers subscription & OpenShift subscription are needed  
#Confirmation
```  
[root@worker-2 ~]# subscription-manager list
+-------------------------------------------+
    インストール済み製品のステータス
+-------------------------------------------+
製品名:           Red Hat OpenShift Container Platform
製品 ID:          290
バージョン:       4.1
アーキテクチャー: x86_64
状態:             サブスクライブ済み
状態の詳細:
開始:             2019年09月30日
終了:             2020年09月30日

製品名:           Red Hat Enterprise Linux Server
製品 ID:          69
バージョン:       7.6
アーキテクチャー: x86_64
状態:             サブスクライブ済み
状態の詳細:
開始:             2019年07月31日
終了:             2020年09月30日
```
1. Enable repos
```
[root@worker-2 ~]# subscription-manager repos \
>     --enable="rhel-7-server-rpms" \
>     --enable="rhel-7-server-extras-rpms" \
>     --enable="rhel-7-server-ansible-2.7-rpms" \
>     --enable="rhel-7-server-ose-4.1-rpms"
リポジトリー 'rhel-7-server-rpms' は、このシステムに対して有効になりました。
リポジトリー 'rhel-7-server-ansible-2.7-rpms' は、このシステムに対して有効になりました。
リポジトリー 'rhel-7-server-ose-4.1-rpms' は、このシステムに対して有効になりました。
リポジトリー 'rhel-7-server-extras-rpms' は、このシステムに対して有効になりました。
```
1. Install Client tools
```
yum install openshift-ansible openshift-clients jq
```
#openshift-clients might be error, install from https://cloud.redhat.com/openshift/install


Links
赤帽エンジニアブログ https://rheb.hatenablog.com/entry/openshift41-baremetal-upi

Add RHEL 7 worker node
https://docs.openshift.com/container-platform/4.1/machine_management/adding-rhel-compute.html
