https://docs.sysdig.com/en/openshift-agent-installation-steps.html

## Prerequisite
```
$ oc get node
NAME       STATUS   ROLES    AGE   VERSION
master-0   Ready    master   15d   v1.14.6+c07e432da
master-1   Ready    master   15d   v1.14.6+c07e432da
master-2   Ready    master   15d   v1.14.6+c07e432da
worker-0   Ready    worker   15d   v1.14.6+c07e432da
worker-1   Ready    worker   15d   v1.14.6+c07e432da
worker-2   Ready    worker   15d   v1.14.6+c07e432da
```
## Agent Install Procedure
1. get trial access key from sysdig website
1. create Project, SA, Node Label
```
oc adm new-project sysdig-agent --node-selector='app=sysdig-agent'
oc label node --all "app=sysdig-agent"
oc project sysdig-agent
oc create serviceaccount sysdig-agent
oc adm policy add-scc-to-user privileged -n sysdig-agent -z sysdig-agent
oc adm policy add-cluster-role-to-user cluster-reader -n sysdig-agent -z sysdig-agent
```
1. create Secret
```
kubectl create secret generic sysdig-agent --from-literal=access-key=b933e079-5aff-41dd-932b-dfe97be738ff -n sysdig-agent
```
1. download `configmap` and `daemonset` yaml from sysdig github  
https://raw.githubusercontent.com/draios/sysdig-cloud-scripts/master/agent_deploy/kubernetes/sysdig-agent-daemonset-v2.yaml
https://raw.githubusercontent.com/draios/sysdig-cloud-scripts/master/agent_deploy/kubernetes/sysdig-agent-configmap.yaml
1. modify `sysdig-agent-configmap.yaml` described here 
```
$ cat sysdig-agent-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sysdig-agent
data:
  dragent.yaml: |
    configmap: true
    ### Agent tags
    # tags: linux:ubuntu,dept:dev,local:nyc

    #### Sysdig Software related config ####

    # Sysdig collector address
    collector: 192.168.1.1

    # Collector TCP port
    collector_port: 443

    # Whether collector accepts ssl
    ssl: true

    # collector certificate validation
    ssl_verify_certificate: false

    #######################################
    # new_k8s: true
    # k8s_cluster_name: production
    security:
      k8s_audit_server_url: 0.0.0.0
      k8s_audit_server_port: 7765
```
1. apply modified yaml files
```
oc create -f sysdig-agent-configmap.yaml
oc create -f sysdig-agent-daemonset-v2.yaml
```
```
$ oc get all
NAME                     READY   STATUS    RESTARTS   AGE
pod/sysdig-agent-2nkb4   1/1     Running   0          49m
pod/sysdig-agent-46ckj   1/1     Running   0          49m
pod/sysdig-agent-6n5wk   1/1     Running   0          49m
pod/sysdig-agent-82vtt   1/1     Running   0          49m
pod/sysdig-agent-cxkdq   1/1     Running   0          49m
pod/sysdig-agent-qjbv4   1/1     Running   0          49m
```

## Web Interface install
https://docs.sysdig.com/en/getting-started.html

