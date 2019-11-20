```
[newgen@bastion01 sql-nginx]$ oc get pods
NAME                                 READY   STATUS      RESTARTS   AGE
polling-app-mysql-77bb768fb-xqng5    1/1     Running     0          20m
polling-app-server-6c8674777-9kvck   0/1     OOMKilled   5          14m
[newgen@bastion01 sql-nginx]$
[newgen@bastion01 sql-nginx]$ oc describe pod polling-app-server-6c8674777-9kvck
Name:               polling-app-server-6c8674777-9kvck
Namespace:          mysql-spring-react
Priority:           0
PriorityClassName:  <none>
Node:               worker-0/10.0.1.110
Start Time:         Wed, 20 Nov 2019 16:34:12 +0900
Labels:             app=polling-app-server
                    pod-template-hash=6c8674777
Annotations:        k8s.v1.cni.cncf.io/networks-status:
                      [{
                          "name": "openshift-sdn",
                          "interface": "eth0",
                          "ips": [
                              "10.130.0.48"
                          ],
                          "default": true,
                          "dns": {}
                      }]
                    openshift.io/scc: restricted
Status:             Running
IP:                 10.130.0.48
Controlled By:      ReplicaSet/polling-app-server-6c8674777
Containers:
  polling-app-server:
    Container ID:   cri-o://1a7077fc2f8297ee230831d5f458a97ac1012b52a367f50c060497911f10a6e6
    Image:          callicoder/polling-app-server:1.0.0
    Image ID:       docker.io/callicoder/polling-app-server@sha256:0c8b14e2ba78bf07c7a4bb6b69eeb94003994b3854dc3c60eefd08c038ef1969
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    137
      Started:      Wed, 20 Nov 2019 16:47:18 +0900
      Finished:     Wed, 20 Nov 2019 16:48:47 +0900
    Ready:          False
    Restart Count:  5
    Limits:
      cpu:     200m
      memory:  200Mi
    Requests:
      cpu:     200m
      memory:  200Mi
    Environment:
      SPRING_DATASOURCE_USERNAME:  <set to the key 'username' in secret 'mysql-user-pass'>  Optional: false
      SPRING_DATASOURCE_PASSWORD:  <set to the key 'password' in secret 'mysql-user-pass'>  Optional: false
      SPRING_DATASOURCE_URL:       <set to the key 'url' in secret 'mysql-db-url'>          Optional: false
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v5l68 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-v5l68:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-v5l68
    Optional:    false
QoS Class:       Guaranteed
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/memory-pressure:NoSchedule
                 node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  16m                  default-scheduler  Successfully assigned mysql-spring-react/polling-app-server-6c8674777-9kvck to worker-0
  Normal   Pulling    16m                  kubelet, worker-0  Pulling image "callicoder/polling-app-server:1.0.0"
  Normal   Pulled     14m                  kubelet, worker-0  Successfully pulled image "callicoder/polling-app-server:1.0.0"
  Normal   Pulled     6m17s (x4 over 12m)  kubelet, worker-0  Container image "callicoder/polling-app-server:1.0.0" already present on machine
  Normal   Created    6m16s (x5 over 14m)  kubelet, worker-0  Created container polling-app-server
  Normal   Started    6m16s (x5 over 14m)  kubelet, worker-0  Started container polling-app-server
  Warning  BackOff    65s (x17 over 11m)   kubelet, worker-0  Back-off restarting failed container

```
