Instractor:
 bthong@redhat.com  
 jase.bkthong@gmail.com  

https://github.com/RedHatTraining/

## Lab info
workstation  
student/student  
https://api.ocp-qyeppkgjxzzvqgj200616.do280.rht-ap.nextcle.com:6443  
eLUie-2d2dL-Hmg9e-4xBLL  
ed9d51a9-f548-4dd3-a91a-3f8cdb25d1e7  

api.ocp-jzjjpoonkkqwynp200618.do280.rht-ap.nextcle.com:  

## Concept
- 高可用性
- 軽量OS
- ロードバランシング
- スケ―リングの自動化
- ログとモニタリング
- サービス検出
- ストレージ
- アプリケーション管理
- クラスターの拡張性

## Conpornent
Static Pod: etcd, kube-scheduler, kube-controller-manager, kube-apiserver
(Directory kubelet runs)

Router:  
*  allow ingress traffic into the cluster
*  Maps DNS to svc  
   svc locate to the backedn pods

Volume:
*  `pv` is cluster wide resource
*  `pvc` is defined by Developers

pod:
*   smallest scheduling unit in k8s
*   1 or more containers
*   shares same IP
*   external storage can use the storage

dc:
*   pod settings
*   Num of replicas
*   triggers
*   dc --> replication controller --> pod

deployment:
*   deployment --> replicaset --> pod

bc:
*   code + builder image -> final image
*   buld process
*   end result is a new container image

ImageStream(is):
*   contains image stream tags
*   `oc get all -n openshift`
*   used by `BuildConfig` and `DeploymentConfig`

Example:
```
    oc create deployment --> deployment
    oc new-app --> deploymentconfig

    oc new-app -S php
    oc new-app -S --template=ruby
    oc new-app -S --image-stream=mysql
    oc new-app -S --docker-image=python
    oc new-app https://github.com/openshift/ruby-hello-world#beta4
```

Operator:
*  Similar to controllers(Observ->Act->)
*  Program to manage another program
*  User defines `CR` defined by `CRD`
*  Operator Framework provide:  
    Practice  
    Distribute  
    Packaging as container image  
  * Most Operator reside in their own dedicated projects `openshift-*-operator`  
* Cluster Operator:
    ```
    • dns
    • network
    • ingress
    • storage
    • authentication
    • console
    • monitoring
    • image-registry
    • autoscale
    • openshift-apiserver
    • openshift-dns
    • openshift-controller-manager
    • dns
    • cloud-credential
    ```

Example:  
 Change CR  
 `oc patch configs.imageregistry.operator.openshift.io/cluster ...`  
 https://docs.openshift.com/container-platform/4.1/registry/configuring-registry-operator.html

- Supported feature by community K8s can be used in OpenShift

## Install
* deploy process
  1. boot bootstrap machine
  2. boot controll plane machine
  3. run Etcd cluster
  4. temprary k8s cluster up
  5. deploy controll plane on the machines
  6. temporary k8s cluster get down
  7. Pass the new controll plane its role from temporary
  8. bootstrap inject OpenShift component to the new control plane
  9. destroy bootstrap node

## Troubleshooting
```
oc adm top node
oc describe node <nodename>   #Capacity/Allocatable/Allocated resources
oc cluster-info
oc get clusterversion
oc describe clusterversion
oc get clusteroperators
oc adm node-logs <nodename> --tail 3
oc adm node-logs <nodename> -u crio
oc adm node-logs <nodename> -u kubelet
# login node
oc debug node/worker-1
chroot /host
systemctl status kubelet
crictl ps --name <podname>
oc get events # events in current namespace
skopeo inspect

oc debug deployment/my-deployment-name --as-root
```

## Indentitiy Management
- User
- Identity
- Service Account
- Group
- Role

* IDP
  - HTPasswd
  - Keystone
  - LDAP
  - GitHub
  - OpenID Connect

* Authenticate for Cluster Admin
  - OAuth(user/password)
  - X.509 certificate (kubeconfig)
  - Virtual User #Set up during cluster install

* kubeadmin
  `kubeadmin` user is virtual user  

* Procedure of `htpasswd` ID provider in OpenShift
  1. Create OAuth custom resource
  2. Export existing custom resource
     `oc get -oyaml oauth cluster > oauth.yaml`  
  3. apply 


* htpasswd
  -B  bcrypt encryption
  -b  password input from command line
  -c  create
  -D  delete

* Tips
  * Find the current IDP settings in the cluster  
    `oc get oauth cluster -oyaml`  
  * Where Secret files stored for htpasswd  
    `oc get secret/htpass-secret -n openshift-config`  
  * Extract htpaswd secret to STDOUT  
    `oc extract secret/htpass-secret -n openshift-config --to -`  
  * get IDs
    `oc get identity` / `oc get users`  


## How to find CR
oc get crd | grep \<what you find keyword\>  
oc get \<name\>  
oc describe \<crd-name\> \<cr-name\>

## RBAC
* Clusterrole
  * Cluster role -> cluster/local role binding -> user/group/sa
  * * Namespace Self provison
    clousterrolebinding for `self-provisioner` cluster role  
    * Add cluster role to user  
      `oc adm policy add-cluster-role-to-user cluster-role username`  
  * populer cluster-role
    * cluster-admin, cluster-status

* Local role
  * namespace scope
  * Local role -> local role binding -> user/group/sa
    oc adm policy add-

* Tips
  * Add priviledge for user to a new project  
    `oc policy add-role-to-user edit isogai -n isogai-prj`  
  * Confirm which role applied in a project
    `oc get rolebindings`  
    `oc get rolebindings edit -oyaml`

## Secret
* create secret  
  `oc create secret generic iso-sec-1 --from-literal=user=isogai --from-literal=pass=password`  
* Set secret into Pod
  `oc set env --from=secret/mysecret dc/myapp`  #As env  
  `oc set volume deploymentconfig.apps.openshift.io/mysql --add --type=secret --secret-name=mysql --mount-path=/app-secrets` #As file

## Configmap

## Secret
`oc create secret generic mysql --from-literal user=myuser --from-literal password=redhat123 --from-literal database=test_secrets --from-literal hostname=mysql`  
`oc new-app --name mysql --docker-image registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47`  
`oc new-app --name quotes --docker-image quay.io/redhattraining/famous-quotes:1.0`  
`curl http://quotes-authorization-secrets.apps.cluster.domain.example.com/env`  
`curl http://quotes-authorization-secrets.apps.cluster.domain.example.com/status`  


## Security Context (SCC)
リソースのアクセスは制限するがOpenShiftの操作は制限しない  
OCP prevent to run as root, also defend by SELINUX  
default is `restricted`  
check when pod start to run

* OpenShift's SCC
```
  • anyuid
  • hostaccess
  • hostmount-anyuid
  • hostnetwork
  • node-exporter
  • nonroot
  • privileged
  • restricted
```

* Confirm security context requirement which the pods requests
  ```shell
  $ oc get pods mysql-3-2rw8r -oyaml | oc adm policy scc-subject-review -f -
  RESOURCE            ALLOWED BY
  Pod/mysql-3-2rw8r   anyuid
  ```

* Lab
`oc new-app --name gitlab gitlab/gitlab-ce:8.4.3-ce.0`  
`oc create sa gitlab-sa`  
`oc adm policy add-scc-to-user anyuid -z gitlab-sa`  


## Network

* Network Type
  * externalname  : クラスタの外側にあるURLをクラスタ内と同じルールで名前解決するために使う(ex. google.com -> mygoogle)
  * NodePort
  * ClusterIP

* DNS
  * CoreDNS
    * Create default cluster name(cluster.local)
    * assgin dns name for `svc` (db.backend.cluster.local)
    * assign dns name for `pod` (db001.backend.cluster.local)
  * Service DNS name
    * A record
    * SRV record (discover port number directry)

* Cluster Network Operator
  * ClusterNetwork/ServiceNetwork
    `oc get network cluster -oyaml`  

* Network Mode
  * Multitenant: isolate project level with each vlan id provided
  * Subnet  : All flat network accross project, tenant
  * NetworkPolicy (Default) : Apply network policy to Pod level. Default is no network policy


* External Access
  * NodePort
  * K8s Ingress
    * K8s Ingress = nginx
  * Route
    * Route = HAProxy
    * `oc expose svc <svcName> 
  * LoadBalancer
    * `oc expose svc loadbalancer <svcname> --port=80:8080`
  * ExternalPort
    * Specify the one of the worker node IP

* Router Options
  * Create self signed certificate  
    1. create CSR
    2. create certificate signed by CA
  * Insecure Route  
    `oc expose service/wordpress`  
  * Secure route
    * Edge  : router that uses edge TLS termination  
    `oc create route edge -h`  
    `oc create route edge [NAME] --service=SERVICE [flags]`  
    Example: `oc create route edge php-https-edge --service=php-http --port 8080 --ca-cert myCA.pem --key php.key --cert php.crt`  
    * Paththrough : router that uses passthrough TLS termination  
      * Container decript the TLS session
      * Use secret resource  
    `oc create route passthrough -h`  
    `oc create route passthrough [NAME] --service=SERVICE [flags]`  
    * reencript : reencrypt TLS termination  
      * If you unsecured routes, `oc expose svc `
    `oc create route reencrypt -h`    
    `oc create route reencrypt [NAME] --dest-ca-cert=FILENAME --service=SERVICE [flags]`  
    * Example
      * `oc create route edge --service <svcName> --key api.key --cert api.crt`  

### Tips: Create CA certificate
https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/
* Becoming CA
  ```
  openssl genrsa -des3 -out myCA.key 2048
  openssl req -x509 -new -nodes -key myCA.key -sha256 -days 1825 -out myCA.pem
  ```
  Enter these information:
  - Country Name:
  - State:
  - Locality Name:
  - Organization Natme:
  - Organizational Unit Name:
  - Common Name:
  - Email Address:
  -> You get `myCA.key`(Private Key) and `myCA.pem`(root Certificate)  

* Create CA-Signed Certificate
  1. Create private key
   `openssl genrsa -out dev.deliciousbrains.com.key 2048`  
  2. Create CSR
   `openssl req -new -key dev.deliciousbrains.com.key -out dev.deliciousbrains.com.csr`  
  3. Create Certificate
  `openssl x509 -req -in dev.deliciousbrains.com.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial -out dev.deliciousbrains.com.crt -days 825 -sha256 -extfile dev.deliciousbrains.com.ext`  



## Scheduling
* Node Selector
  * `deployment.spec.template.spec.nodeSelector`

## Resource limit/requests
* Limits
  * `deployment.spec.template.spec.containers.resources.limits`  
* Requests
  * `deployment.spec.template.spec.containers.resources.requests`  

## Quota
* Object Count
  * Pods, replicationcontrollers, services, secrets, persistentVolumeClaims
* Resources
  * cpu, memory, storage

## Cluster Scaling
* manual scaling
oc get machinesets -n openshift-machine-api
oc scale machinesets -n openshift-machine-api <machineset-name> --replicas=3

* Auto scaling
Cluster AutoScaler -> MachineAutoScaler(AZ)
1. Create ClusterAutoscaler
2. Create MachineAutoscaler


## Upgrade
oc get clusterversion
can skip z-stream versions (x.y.z, ex: 4.1.12)



## Final Assesment
oc new-app --name hello-world-nginx https://github.com/RedhatTraining/DO280-apps --context-dir hello-world-nginx

registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7
  secret/mysql=MYSQL_ROOT_PASSWORD=r3dh4t123
docker.io/library/wordpress:5.3.0
  WORDPRESS_DB_HOST=mysql
  WORDPRESS_DB_NAME=wordpress
  WORDPRESS_DB_PASSWORD=secret/mysql



## Commands
```
systemctl status kubelet
systemctl status crio
journalctl -u crio, journalctl -f -u kubelet

oc get pod -A -l app=xxx
host <url>
oc get pv

oc explain dc.spec
oc get clusteroperator
oc adm node-log <node>
oc adm node-log -u kubelet
oc adm node-log -u crio
oc debug node/<node>

curl -k -v -XHEAD 
oc debug deployment/my-deployment-name --as-root
oc port-forward <pod> local-port:remote-port
oc get pod --log-level 6
oc logs --tail 3 -n <namespace> 
#in CoreOS
crictl ps --name openvswitch
oc status
oc get events
skopeo inspect docker://registry.access.redhat.com/rhscl/postgresq-96-rhel7:1

oc get identity
openssl rand -hex 15
oc extract secret/htp-secret -n openshift-config --to - > temp2

oc rollout latest dc/abc
oc rollout cancel dc/abc

oc get clusterrolebinding
oc describe clusterrolebinding <rolebinding-name>
grep -E "NAME|self-provisoner"


oc project |grep -i auth
oc get pods -n openshift-authentication-operator
oc adm policy add-cluster-role-to-user cluster-admin student
oc adm policy remove-scc-from-user aaa -z <sa-account>
oc adm policy scc-subject-review #same as describe pod

oc new-app --name httpd httpd:2.4

oc get clusterrole # show pre-defined roles
oc adm policy
  oc adm policy who-can <verv> <resource>
oc set env
oc set volume

oc new-app
oc new-app --search
oc new-app --image-stream
ps axo pid,euid,egid,comm
```

## YAML Basic
- No tabs
- Lists (array)
  - users
    - alice
    - bob
  * allginment is important
- Dictionaries
  - key: value

Regarding PV, When we specify `fstype` in `pod.spec.volumes.iscsi` Who does format the filesystem of PV?
container runtime(crio), or container itself?

My lab console doesn't show login screen yet.
Could you check it, should I reset it, or wait more time?

It's bit difficult question though, do you have a recommendation which clusterroles should be assign to a Developer?
-> edit

## Sample docker images
| name | path | Comment |
| --- | --- | --- |
| mysql | registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47 | Set user and pass in secret `--from-literal=user=myuser --from-literal=password=r3dh4t123 --from-literal=database=test_secrets --from-literal=hostname=mysql` |
| php | quay.io/redhattraining/php-ssl:v1.0 | Returm access method as http |
| php | quay.io/redhattraining/php-ssl:v1.1 | Return access method as https |
| nginx | quay.io/redhattraining/hello-world-nginx:v1.0 | Return hello world |
| php | quay.io/redhattraining/scaling:v1.0 | Return Pod's IP |



## 謎

```
[newgen@bastion01 hello-world-nginx]$ oc rsh mysql-2-r47f8
sh-4.2$ mysql -u myuser --password=r3dh4t123 test_secrets -e 'show databases;'
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| information_schema |
| test_secrets       |
+--------------------+
sh-4.2$
sh-4.2$ exit
exit
[newgen@bastion01 hello-world-nginx]$ oc debug pod mysql-2-r47f8
Error from server (NotFound): pods "pod" not found
[newgen@bastion01 hello-world-nginx]$ oc get pods
NAME             READY   STATUS      RESTARTS   AGE
mysql-2-deploy   0/1     Completed   0          3m48s
mysql-2-r47f8    1/1     Running     0          3m45s
[newgen@bastion01 hello-world-nginx]$
[newgen@bastion01 hello-world-nginx]$ oc debug pod mysql-2-r47f8
Error from server (NotFound): pods "pod" not found
[newgen@bastion01 hello-world-nginx]$ oc debug mysql-2-r47f8
Starting pod/mysql-2-r47f8-debug, command was: container-entrypoint run-mysqld
Pod IP: 10.129.1.73
If you don't see a command prompt, try pressing enter.
sh-4.2$ mysql -u myuser --password=r3dh4t123 test_secrets -e 'show databases;'
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
```
