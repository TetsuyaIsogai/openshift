1. Log in to OpenShift Console
1. Create Namespace
- Name: odh
- Display name: "Open Data Hub"
1. Find `Open Data Hub` operator in the `OperatorHub`
1. Click `Open Data Hub` operator. When you see warning, click `proceed`
1. Click `install` button and click `subscribe` (wait about 10 minuts)
1. Go to `installed operator` and confirm the operator status is `InstallSucceeded`
1. Click `Create New` 

*If you don't have any PersistentVolume and deploy using default YAML settings, you see the errors below.*
```
NAME                                    READY   STATUS    RESTARTS   AGE
grafana-85c66bdc6d-2pf4w                0/2     Pending   0          20m
jupyterhub-1-deploy                     0/1     Error     0          25m
jupyterhub-db-1-deploy                  0/1     Error     0          25m
opendatahub-operator-7464f66749-spl5t   1/1     Running   0          16h
prometheus-0                            4/4     Running   0          25m
spark-operator-7d45c6cb98-rhgrj         1/1     Running   0          25m
```


