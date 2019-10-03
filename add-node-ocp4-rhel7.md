## Prerequisite
- use same machine to execute 

## Procedure
### Outside of the cluster
1. Add dhcp entry in dnsmasq

## Bastion node
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
yum update
yum install openshift-ansible openshift-clients jq
```
<details>
  
```
[root@bastion01 ~]# yum install openshift-ansible
読み込んだプラグイン:product-id, search-disabled-repos, subscription-manager
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ openshift-ansible.noarch 0:4.1.18-201909201709.git.162.ccd0d87.el7 を インストール
--> 依存性の処理をしています: ansible >= 2.7.8 のパッケージ: openshift-ansible-4.1.18-201909201709.git.162.ccd0d87.el7.noarch
--> 依存性の処理をしています: openshift-clients のパッケージ: openshift-ansible-4.1.18-201909201709.git.162.ccd0d87.el7.noarch
--> トランザクションの確認を実行しています。
---> パッケージ ansible.noarch 0:2.8.4-1.el7 を インストール
--> 依存性の処理をしています: PyYAML のパッケージ: ansible-2.8.4-1.el7.noarch
--> 依存性の処理をしています: python-httplib2 のパッケージ: ansible-2.8.4-1.el7.noarch
--> 依存性の処理をしています: python-jinja2 のパッケージ: ansible-2.8.4-1.el7.noarch
--> 依存性の処理をしています: python-paramiko のパッケージ: ansible-2.8.4-1.el7.noarch
--> 依存性の処理をしています: python2-cryptography のパッケージ: ansible-2.8.4-1.el7.noarch
--> 依存性の処理をしています: python2-jmespath のパッケージ: ansible-2.8.4-1.el7.noarch
--> 依存性の処理をしています: sshpass のパッケージ: ansible-2.8.4-1.el7.noarch
---> パッケージ openshift-clients.x86_64 0:4.1.18-201909201709.git.0.12ee15d.el7 を インストール
--> トランザクションの確認を実行しています。
---> パッケージ PyYAML.x86_64 0:3.10-11.el7 を インストール
--> 依存性の処理をしています: libyaml-0.so.2()(64bit) のパッケージ: PyYAML-3.10-11.el7.x86_64
---> パッケージ python-httplib2.noarch 0:0.9.2-1.el7 を インストール
---> パッケージ python-jinja2.noarch 0:2.7.2-4.el7 を インストール
--> 依存性の処理をしています: python-babel >= 0.8 のパッケージ: python-jinja2-2.7.2-4.el7.noarch
--> 依存性の処理をしています: python-markupsafe のパッケージ: python-jinja2-2.7.2-4.el7.noarch
---> パッケージ python-paramiko.noarch 0:2.1.1-9.el7 を インストール
--> 依存性の処理をしています: python2-pyasn1 のパッケージ: python-paramiko-2.1.1-9.el7.noarch
---> パッケージ python2-cryptography.x86_64 0:1.7.2-2.el7 を インストール
--> 依存性の処理をしています: python-cffi >= 1.4.1 のパッケージ: python2-cryptography-1.7.2-2.el7.x86_64
--> 依存性の処理をしています: python-idna >= 2.0 のパッケージ: python2-cryptography-1.7.2-2.el7.x86_64
--> 依存性の処理をしています: python-enum34 のパッケージ: python2-cryptography-1.7.2-2.el7.x86_64
---> パッケージ python2-jmespath.noarch 0:0.9.0-4.el7ae を インストール
---> パッケージ sshpass.x86_64 0:1.06-2.el7 を インストール
--> トランザクションの確認を実行しています。
---> パッケージ libyaml.x86_64 0:0.1.4-11.el7_0 を インストール
---> パッケージ python-babel.noarch 0:0.9.6-8.el7 を インストール
---> パッケージ python-cffi.x86_64 0:1.6.0-5.el7 を インストール
--> 依存性の処理をしています: python-pycparser のパッケージ: python-cffi-1.6.0-5.el7.x86_64
---> パッケージ python-enum34.noarch 0:1.0.4-1.el7 を インストール
---> パッケージ python-idna.noarch 0:2.4-1.el7 を インストール
---> パッケージ python-markupsafe.x86_64 0:0.11-10.el7 を インストール
---> パッケージ python2-pyasn1.noarch 0:0.1.9-7.el7 を インストール
--> トランザクションの確認を実行しています。
---> パッケージ python-pycparser.noarch 0:2.14-1.el7 を インストール
--> 依存性の処理をしています: python-ply のパッケージ: python-pycparser-2.14-1.el7.noarch
--> トランザクションの確認を実行しています。
---> パッケージ python-ply.noarch 0:3.4-11.el7 を インストール
--> 依存性解決を終了しました。

依存性を解決しました

================================================================================================================
 Package               アーキテクチャー
                               バージョン                                 リポジトリー                     容量
================================================================================================================
インストール中:
 openshift-ansible     noarch  4.1.18-201909201709.git.162.ccd0d87.el7    rhel-7-server-ose-4.1-rpms       38 k
依存性関連でのインストールをします:
 PyYAML                x86_64  3.10-11.el7                                rhel-7-server-rpms              153 k
 ansible               noarch  2.8.4-1.el7                                epel                             15 M
 libyaml               x86_64  0.1.4-11.el7_0                             rhel-7-server-rpms               55 k
 openshift-clients     x86_64  4.1.18-201909201709.git.0.12ee15d.el7      rhel-7-server-ose-4.1-rpms       16 M
 python-babel          noarch  0.9.6-8.el7                                rhel-7-server-rpms              1.4 M
 python-cffi           x86_64  1.6.0-5.el7                                rhel-7-server-rpms              218 k
 python-enum34         noarch  1.0.4-1.el7                                rhel-7-server-rpms               52 k
 python-httplib2       noarch  0.9.2-1.el7                                rhel-7-server-extras-rpms       115 k
 python-idna           noarch  2.4-1.el7                                  rhel-7-server-rpms               94 k
 python-jinja2         noarch  2.7.2-4.el7                                rhel-7-server-rpms              519 k
 python-markupsafe     x86_64  0.11-10.el7                                rhel-7-server-rpms               25 k
 python-paramiko       noarch  2.1.1-9.el7                                rhel-7-server-rpms              269 k
 python-ply            noarch  3.4-11.el7                                 rhel-7-server-rpms              123 k
 python-pycparser      noarch  2.14-1.el7                                 rhel-7-server-rpms              105 k
 python2-cryptography  x86_64  1.7.2-2.el7                                rhel-7-server-rpms              503 k
 python2-jmespath      noarch  0.9.0-4.el7ae                              rhel-7-server-ansible-2.7-rpms   39 k
 python2-pyasn1        noarch  0.1.9-7.el7                                rhel-7-server-rpms              100 k
 sshpass               x86_64  1.06-2.el7                                 rhel-7-server-extras-rpms        21 k

トランザクションの要約
================================================================================================================
インストール  1 パッケージ (+18 個の依存関係のパッケージ)

合計容量: 35 M
総ダウンロード容量: 635 k
インストール容量: 175 M
Is this ok [y/d/N]: y
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/5): PyYAML-3.10-11.el7.x86_64.rpm                                                     | 153 kB  00:00:05
(2/5): libyaml-0.1.4-11.el7_0.x86_64.rpm                                                 |  55 kB  00:00:07
(3/5): python-cffi-1.6.0-5.el7.x86_64.rpm                                                | 218 kB  00:00:07
(4/5): python-idna-2.4-1.el7.noarch.rpm                                                  |  94 kB  00:00:08
(5/5): python-httplib2-0.9.2-1.el7.noarch.rpm                                            | 115 kB  00:00:13
----------------------------------------------------------------------------------------------------------------
合計                                                                             34 kB/s | 635 kB  00:00:18
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  インストール中          : python2-pyasn1-0.1.9-7.el7.noarch                                              1/19
  インストール中          : python-httplib2-0.9.2-1.el7.noarch                                             2/19
  インストール中          : sshpass-1.06-2.el7.x86_64                                                      3/19
  インストール中          : python2-jmespath-0.9.0-4.el7ae.noarch                                          4/19
  インストール中          : python-babel-0.9.6-8.el7.noarch                                                5/19
  インストール中          : openshift-clients-4.1.18-201909201709.git.0.12ee15d.el7.x86_64                 6/19
  インストール中          : python-markupsafe-0.11-10.el7.x86_64                                           7/19
  インストール中          : python-jinja2-2.7.2-4.el7.noarch                                               8/19
  インストール中          : python-enum34-1.0.4-1.el7.noarch                                               9/19
  インストール中          : python-ply-3.4-11.el7.noarch                                                  10/19
  インストール中          : python-pycparser-2.14-1.el7.noarch                                            11/19
  インストール中          : python-cffi-1.6.0-5.el7.x86_64                                                12/19
  インストール中          : libyaml-0.1.4-11.el7_0.x86_64                                                 13/19
  インストール中          : PyYAML-3.10-11.el7.x86_64                                                     14/19
  インストール中          : python-idna-2.4-1.el7.noarch                                                  15/19
  インストール中          : python2-cryptography-1.7.2-2.el7.x86_64                                       16/19
  インストール中          : python-paramiko-2.1.1-9.el7.noarch                                            17/19
  インストール中          : ansible-2.8.4-1.el7.noarch                                                    18/19
  インストール中          : openshift-ansible-4.1.18-201909201709.git.162.ccd0d87.el7.noarch              19/19
  検証中                  : python-idna-2.4-1.el7.noarch                                                   1/19
  検証中                  : libyaml-0.1.4-11.el7_0.x86_64                                                  2/19
  検証中                  : python-ply-3.4-11.el7.noarch                                                   3/19
  検証中                  : python-enum34-1.0.4-1.el7.noarch                                               4/19
  検証中                  : python-paramiko-2.1.1-9.el7.noarch                                             5/19
  検証中                  : python-markupsafe-0.11-10.el7.x86_64                                           6/19
  検証中                  : openshift-clients-4.1.18-201909201709.git.0.12ee15d.el7.x86_64                 7/19
  検証中                  : python-babel-0.9.6-8.el7.noarch                                                8/19
  検証中                  : openshift-ansible-4.1.18-201909201709.git.162.ccd0d87.el7.noarch               9/19
  検証中                  : python2-jmespath-0.9.0-4.el7ae.noarch                                         10/19
  検証中                  : ansible-2.8.4-1.el7.noarch                                                    11/19
  検証中                  : python-pycparser-2.14-1.el7.noarch                                            12/19
  検証中                  : sshpass-1.06-2.el7.x86_64                                                     13/19
  検証中                  : python-jinja2-2.7.2-4.el7.noarch                                              14/19
  検証中                  : python2-pyasn1-0.1.9-7.el7.noarch                                             15/19
  検証中                  : PyYAML-3.10-11.el7.x86_64                                                     16/19
  検証中                  : python-httplib2-0.9.2-1.el7.noarch                                            17/19
  検証中                  : python-cffi-1.6.0-5.el7.x86_64                                                18/19
  検証中                  : python2-cryptography-1.7.2-2.el7.x86_64                                       19/19
rhel-7-server-ansible-2.7-rpms/x86_64/productid                                          | 2.1 kB  00:00:00
rhel-7-server-extras-rpms/x86_64/productid                                               | 2.1 kB  00:00:00
rhel-7-server-ose-4.1-rpms/x86_64/productid                                              | 2.1 kB  00:00:00

インストール:
  openshift-ansible.noarch 0:4.1.18-201909201709.git.162.ccd0d87.el7

依存性関連をインストールしました:
  PyYAML.x86_64 0:3.10-11.el7                 ansible.noarch 0:2.8.4-1.el7
  libyaml.x86_64 0:0.1.4-11.el7_0             openshift-clients.x86_64 0:4.1.18-201909201709.git.0.12ee15d.el7
  python-babel.noarch 0:0.9.6-8.el7           python-cffi.x86_64 0:1.6.0-5.el7
  python-enum34.noarch 0:1.0.4-1.el7          python-httplib2.noarch 0:0.9.2-1.el7
  python-idna.noarch 0:2.4-1.el7              python-jinja2.noarch 0:2.7.2-4.el7
  python-markupsafe.x86_64 0:0.11-10.el7      python-paramiko.noarch 0:2.1.1-9.el7
  python-ply.noarch 0:3.4-11.el7              python-pycparser.noarch 0:2.14-1.el7
  python2-cryptography.x86_64 0:1.7.2-2.el7   python2-jmespath.noarch 0:0.9.0-4.el7ae
  python2-pyasn1.noarch 0:0.1.9-7.el7         sshpass.x86_64 0:1.06-2.el7

完了しました!
[root@bastion01 ~]#
```

</details>

### Worker node
1. create virtual machine for worker node you want to add in cluster
1. install RHEL7.6
  - Lang: English
  - TZ: UTC 00:00 # allign with existing cluster
  - the other options are default
1. Boot and login to the new worker node
1. Enable `ens192` & confirm ssh login
1. subscription-manager register
1. subscription-manager attach --pool=<pool-id>
1. disable repo
  1. subscription-manager repos --disable="*"
  1. Enable repos
```
    subscription-manager repos \
    --enable="rhel-7-server-rpms" \
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-ose-4.1-rpms"
```
```
    リポジトリー 'rhel-7-server-rpms' は、このシステムに対して有効になりました。
    リポジトリー 'rhel-7-server-ose-4.1-rpms' は、このシステムに対して有効になりました。
    リポジトリー 'rhel-7-server-extras-rpms' は、このシステムに対して有効になりました。
```
### Add Worker node
#### on bastion node
1. configure ssh public key access bastion to worker
    From Worker node: `mkdir ~/.ssh; chmod 700 .ssh`
    From bastion node: `scp <public-key> root@<new-worker>:~/.ssh/authorized_keys`
1.1. make sure to connnect bastion -> worker node without passsword
    #confirm accessing to `root` account
1. create pull secret file
    `oc -n openshift-config get -o jsonpath='{.data.\.dockerconfigjson}' secret pull-secret | base64 -d | jq . |tee pull-secret.txt`
1. create hosts file
    ```
    [all:vars]
    ansible_user=root
    #ansible_become=True

    openshift_kubeconfig_path="/home/newgen/bare-metal/auth/kubeconfig"
    openshift_pull_secret_path="/home/newgen/add-rhel-worker/pull-secret.txt"

    [new_workers]
    worker-2.oc4cluster.tetsuya.local
    [new_workers:vars]
    ansible_ssh_private_key_file=<private-key-location>
    ```
    make sure to access to Worker node via ansible
    `ansible new_workers -i inventory/hosts -m ping1`
    
1. exec playbook
```
$ cd /usr/share/ansible/openshift-ansible
$ ansible-playbook -i /<path>/inventory/hosts playbooks/scaleup.yml 
```


Links
赤帽エンジニアブログ https://rheb.hatenablog.com/entry/openshift41-baremetal-upi

Add RHEL 7 worker node
https://docs.openshift.com/container-platform/4.1/machine_management/adding-rhel-compute.html
