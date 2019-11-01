## Reference
https://github.com/openshift/pipelines-tutorial

## Prerequisite
- OpenShift 4.2 Cluster
- install tekton cli on bastion node
- set default storageClass for dinamic provisioning from tekton pipelines

## Procedures
1. Install OpenShift Pipelines
follow this procedure via OpenShift Admin Console -> OperatorHub
https://github.com/openshift/pipelines-tutorial/blob/master/install-operator.md

## Deploy Sample Application
1. create new pipelines
1. OpenShift Pipelines add `serviceaccount` - `pipeline`
```
$ oc get serviceaccounts
NAME       SECRETS   AGE
builder    2         8s
default    2         8s
deployer   2         8s
pipeline   2         8s
```
1. apply manifest
```
$ curl -O https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/petclinic/manifests.yaml
```
<details><div>
  
```
---
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  labels:
    app: spring-petclinic
  name: spring-petclinic
---
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  labels:
    app: spring-petclinic
  name: spring-petclinic
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    app: spring-petclinic
    deploymentconfig: spring-petclinic
  strategy:
    activeDeadlineSeconds: 21600
    resources: {}
    rollingParams:
      intervalSeconds: 1
      maxSurge: 25%
      maxUnavailable: 25%
      timeoutSeconds: 600
      updatePeriodSeconds: 1
    type: Rolling
  template:
    metadata:
      labels:
        app: spring-petclinic
        deploymentconfig: spring-petclinic
    spec:
      containers:
      - image: spring-petclinic:latest
        imagePullPolicy: Always
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 45
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        name: spring-petclinic
        ports:
        - containerPort: 8080
          protocol: TCP
        - containerPort: 8443
          protocol: TCP
        - containerPort: 8778
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 45
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
  test: false
  triggers:
  - imageChangeParams:
      containerNames:
      - spring-petclinic
      from:
        kind: ImageStreamTag
        name: spring-petclinic:latest
        namespace: pipelines-tutorial
    type: ImageChange
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: spring-petclinic
  name: spring-petclinic
spec:
  ports:
  - name: 8080-tcp
    port: 8080
    protocol: TCP
    targetPort: 8080
  - name: 8443-tcp
    port: 8443
    protocol: TCP
    targetPort: 8443
  - name: 8778-tcp
    port: 8778
    protocol: TCP
    targetPort: 8778
  selector:
    app: spring-petclinic
    deploymentconfig: spring-petclinic
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app: spring-petclinic
  name: spring-petclinic
spec:
  port:
    targetPort: 8080-tcp
  to:
    kind: Service
    name: spring-petclinic
    weight: 100
```
  
</div></details>

## Install Tasks
- task manifest
```
apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: maven-build
spec:
  inputs:
    resources:
    - name: workspace-git
      targetPath: /
      type: git
  steps:
  - name: build
    image: maven:3.6.0-jdk-8-slim
    command:
    - /usr/bin/mvn
    args:
    - install
```
1. task install
```
$ oc create -f https://raw.githubusercontent.com/openshift/tektoncd-catalog/release-v0.7/openshift-client/openshift-client-task.yaml
$ oc create -f https://raw.githubusercontent.com/openshift/pipelines-catalog/release-v0.7/s2i-java-8/s2i-java-8-task.yaml
```
## Create Pipeline
```
apiVersion: tekton.dev/v1alpha1
kind: Pipeline
metadata:
  name: petclinic-deploy-pipeline
spec:
  resources:
  - name: app-git
    type: git
  - name: app-image
    type: image
  tasks:
  - name: build
    taskRef:
      name: s2i-java-8
    params:
      - name: TLSVERIFY
        value: "false"
    resources:
      inputs:
      - name: source
        resource: app-git
      outputs:
      - name: image
        resource: app-image
  - name: deploy
    taskRef:
      name: openshift-client
    runAfter:
      - build
    params:
    - name: ARGS
      value: "rollout latest spring-petclinic"
```
1. Confirm created pipeline
```
$ tkn pipeline list
NAME                        AGE             LAST RUN   STARTED   DURATION   STATUS
petclinic-deploy-pipeline   9 seconds ago   ---        ---       ---        ---
```
## Trigger pipeline
1. create pipeline resource
```
---
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: petclinic-image
spec:
  type: image
  params:
  - name: url
    value: image-registry.openshift-image-registry.svc:5000/pipelines-tutorial/spring-petclinic
---
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: petclinic-git
spec:
  type: git
  params:
  - name: url
    value: https://github.com/spring-projects/spring-petclinic
```
```
$ tkn  resource ls
NAME              TYPE    DETAILS
petclinic-git     git     url: https://github.com/spring-projects/spring-petclinic
petclinic-image   image   url: image-registry.openshift-image-registry.svc:5000/pipelines-tutorial/spring-petclinic
```

## Start pipeline
```
tkn pipeline start petclinic-deploy-pipeline \
        -r app-git=petclinic-git \
        -r app-image=petclinic-image \
        -s pipeline
```
Confrim the pipeline started
```
$ tkn pipeline ls
NAME                        AGE             LAST RUN                              STARTED          DURATION   STATUS
petclinic-deploy-pipeline   7 minutes ago   petclinic-deploy-pipeline-run-8mg5c   50 seconds ago   ---        Running
```
Check progress as it runs
```
$ tkn pipelinerun logs petclinic-deploy-pipeline-run-8mg5c -f -n pipelines-tutorial
```
