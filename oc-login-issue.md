## Simptoms
* `oc login` is success, but `oc get node` shows `error: You must be logged in to the server (Unauthorized)`

## Result
* This is successful
`oc login -u admin -p P@ssw0rd --kubeconfig=/home/newgen/ocp43/install/auth/kubeconfig`

* These are unsuccessful
`oc login -u admin -p P@ssw0rd`
`oc login -u admin -p P@ssw0rd --certificate-authority=$INGRESSCERT`


