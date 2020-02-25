* gitlab-ci.yaml
```
deploy:
  stage: deploy
  script:
    - oc version --client
    - oc --kubeconfig=/var/gitlab/cert/kubeconfig get node
    - oc new-app 10.0.0.131:5000/gs-spring-boot-docker:0.1 --insecure-registry=true --kubeconfig=/var/gitlab/cert/kubeconfig

test:
  stage: test
  script:
    - oc get dc gs-spring-boot-docker --kubeconfig=/var/gitlab/cert/kubeconfig
    - oc rollout status dc/gs-spring-boot-docker --kubeconfig=/var/gitlab/cert/kubeconfig
    - oc expose dc/gs-spring-boot-docker --port=8080 --kubeconfig=/var/gitlab/cert/kubeconfig
    - oc expose svc/gs-spring-boot-docker  --kubeconfig=/var/gitlab/cert/kubeconfig
    - oc get route gs-spring-boot-docker --kubeconfig=/var/gitlab/cert/kubeconfig
```
