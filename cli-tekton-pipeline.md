# Install tekton pipelines
## Reference
https://github.com/tektoncd/pipeline/blob/master/docs/install.md  
tekton cli  
https://github.com/tektoncd/cli

## Prerequisites
- Install OCP4.2 Cluster
- login as user with `cluster-admin` privileges
```
$ oc whoami
system:admin
```
## tekton cli install (Optional)
1. Download `tkn` binary
```
curl -LO https://github.com/tektoncd/cli/releases/download/v0.4.0/tkn_0.4.0_Linux_x86_64.tar.gz
```
1. move it to `/usr/local/bin`

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
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-controller-669dfb5dd4-jld8p   1/1     Running   0          96s
tekton-pipelines-webhook-5669b89c9c-dl9t4      1/1     Running   0          96s

```

# Tutorial 1 - Hello World -
1. Create YAML file `task` and `taskrun`
```
$ cat <<EOF > task.yaml
> apiVersion: tekton.dev/v1alpha1
> kind: Task
> metadata:
>   name: echo-hello-world
> spec:
>   steps:
>     - name: echo
>       image: ubuntu
>       command:
>         - echo
>       args:
>         - "hello world"
> EOF
```
```
$ cat <<EOF > taskrun.yaml
> apiVersion: tekton.dev/v1alpha1
> kind: TaskRun
> metadata:
>   name: echo-hello-world-task-run
> spec:
>   taskRef:
>     name: echo-hello-world
> EOF
```
1. apply YAML files (task)
```
$ oc create -f task.yaml
task.tekton.dev/echo-hello-world created
$ tkn task list
NAME               AGE
echo-hello-world   41 seconds ago
```
1. apply YAML files (taskrun)
```
$ oc create -f taskrun.yaml
taskrun.tekton.dev/echo-hello-world-task-run created
$ tkn taskrun list
NAME                        STARTED          DURATION   STATUS
echo-hello-world-task-run   27 seconds ago   ---        Running(Pending)
```


