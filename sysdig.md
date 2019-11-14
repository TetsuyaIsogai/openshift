https://docs.sysdig.com/en/openshift-agent-installation-steps.html

1. get trial access key from sysdig website
1. oc new-project sysdig
1. install sysdig agent from `OperatorHub`
1. install begins automatically
```
$ oc get pods -n sysdig
NAME                               READY   STATUS              RESTARTS   AGE
sysdig-operator-5b895f4b9d-zwvxt   0/1     ContainerCreating   0          74s
```
then running
```
$ oc get pods
NAME                               READY   STATUS    RESTARTS   AGE
sysdig-operator-5b895f4b9d-zwvxt   1/1     Running   0          118s
```
1. Click Installed Operators -> sysdig Agent Operators
1. Click Sysdig Agent daemonSet -> Create Instance
* Modify YAML file (This sample use `ebpf and secure` model)
* Paste access Key 
```
apiVersion: sysdig.com/v1alpha1
kind: SysdigAgent
metadata:
  name: agent-with-ebpf-and-secure
spec:
  ebpf:
    enabled: true
  secure:
    enabled: true
  sysdig:
    accessKey: XXX
```


```
access key: b933e079-5aff-41dd-932b-dfe97be738ff
```
