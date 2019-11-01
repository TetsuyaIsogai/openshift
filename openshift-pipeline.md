## Reference
https://github.com/tektoncd/pipeline/blob/master/docs/install.md

## Prerequisites
- Install OCP4.2 Cluster
- login as user with `cluster-admin` privileges
```
$ oc whoami
system:admin
```

## Procedures
1. Create project
```
$ oc new-project tekton-pipelines
$ oc project
Using project "tekton-pipelines" on server "https://api.oc4cluster.tetsuya.local:6443".
```
```
$ oc adm policy add-scc-to-user anyuid -z tekton-pipelines-controller
securitycontextconstraints.security.openshift.io/anyuid added to: ["system:serviceaccount:tekton-pipelines:tekton-pipelines-controller"]
```
```
$ oc apply --filename https://storage.googleapis.com/tekton-releases/latest/release.yaml
```
1. Confirmation
```
$ oc get pods -n tekton-pipelines
```
