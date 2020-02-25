# Registry Setup
## Objective
Set up container registry on bation node

## Reference
https://blog.openshift.com/openshift-4-2-disconnected-install/

## Procedure
1. Install `podman`, `httpd`, and `httpd-tools`
```
$ sudo yum -y install podman httpd httpd-tools
```
1. Create registry's directry
```
$ sudo mkdir -p /opt/registry/{auth,certs,data}
```
1. Generate an SSL certificate
```
$ sudo openssl req -newkey rsa:4096 -nodes -sha256 -keyout domain.key -x509 -days 365 -out domain.crt
```
```
Country Name (2 letter code) [XX]:
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:registry.oc4cluster.tetsuya.local  #Registry FQDN
Email Address []:
```
1. Generate a username and password for access to the registry
User: admin
passwd: password
```
$ sudo htpasswd -bBc /opt/registry/auth/htpasswd admin password
```
1. Open Firewall Port
```
]$ sudo -i
[root@bastion01 ~]# firewall-cmd --add-port=5000/tcp --zone=internal --permanent
success
[root@bastion01 ~]# firewall-cmd --add-port=5000/tcp --zone=public   --permanent
success
[root@bastion01 ~]# firewall-cmd --add-service=http  --permanent
success
[root@bastion01 ~]# firewall-cmd --reload
success
```
1. (Option) Set MTU size
```
# ip link set dev ens192 mtu 1300
# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1300 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b6:c7:7f brd ff:ff:ff:ff:ff:ff
3: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:b6:9c:8b brd ff:ff:ff:ff:ff:ff
```
1. (Option) Add dns entry for resolving FQDN of registry
1. Run the registry
```
podman run --name poc-registry -p 5000:5000 \
-v /opt/registry/data:/var/lib/registry:z \
-v /opt/registry/auth:/auth:z \
-e "REGISTRY_AUTH=htpasswd" \
-e "REGISTRY_AUTH_HTPASSWD_REALM=Registry" \
-e "REGISTRY_HTTP_SECRET=ALongRandomSecretForRegistry" \
-e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
-v /opt/registry/certs:/certs:z \
-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
-e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
docker.io/library/registry:2
```
* Run regsitry process foreground *
1. Confirm access to registry
```
curl -u admin:password -k https://registry.oc4cluster.tetsuya.local:5000/v2/_catalog
```

## How to Use Insecure registry (under proxy)
- Write registry server info into `/etc/docker/daemon.json`
```
{
  "insecure-registries" : ["http://<docker-host>:5000"]
}
```
- Proxy info into `/etc/systemd/system/docker.service.d/http_proxy.conf`
```
[Service]
Environment="HTTP_PROXY=http://<proxy-server>:3128/"
Environment="HTTPS_PROXY=http://<proxy-server>:3128/"
Environment="NO_PROXY=localhost,127.0.0.1,<registry-server>"
```

## Tips
- Stop registry
```
podman stop poc-regstry
```
- Start registry
```
podman start poc-regstry
```
- Check status
```
podman info
```
