# "Buenos Aires": Kubernetes Pod Crashing

**Description:** There are two brothers (pods) `logger` and `logshipper` living in the default namespace. Unfortunately, `logshipper` has an issue (crashlooping) and is forbidden to see what `logger` is trying to say. Could you help fix Logshipper? You can check the status of the pods with `kubectl get pods`.  

Do not change the K8S definition of the `logshipper` pod. Use "sudo".

**Test:** `kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].state.waiting.reason // .status.phase)"'` returns `Running`.  


---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 4440
drwxr-xr-x 7 admin admin    4096 Oct 23 04:02 .
drwxr-xr-x 3 root  root     4096 Sep 17 16:44 ..
drwx------ 3 admin admin    4096 Sep 20 15:52 .ansible
-rw------- 1 admin admin      30 Oct 23 04:25 .bash_history
-rw-r--r-- 1 admin admin     220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin    3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin    4096 Sep 20 15:56 .config
drwxr-x--- 3 admin admin    4096 Oct 23 04:02 .kube
-rw-r--r-- 1 admin admin     807 Aug  4  2021 .profile
drwx------ 2 admin admin    4096 Sep 17 16:44 .ssh
drwxr-xr-x 2 admin root     4096 Oct 23 04:24 agent
-rw-r--r-- 1 root  root  4501504 Oct 23 02:40 busybox.tar
```

`cat agent/check.sh`  
```console
#!/usr/bin/bash
res=$(sudo kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].state.waiting.reason // .status.phase)"')
res=$(echo $res|tr -d '\r')

if [[ "$res" != "Running" ]]
then
  echo -n "NO"
  exit
fi

res=$(sudo k3s kubectl get pods -l app=logshipper --no-headers -o custom-columns=":.spec.serviceAccountName")
res=$(echo $res|tr -d '\r')

if [[ "$res" = "logshipper-sa" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`sudo -i`  
`kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].state.waiting.reason // .status.phase)"'`  
```console
CrashLoopBackOff
```

`kubectl get no`  
```console
NAME    STATUS   ROLES                  AGE    VERSION
node1   Ready    control-plane,master   101d   v1.27.6+k3s1
```

`kubectl get service`  
```console
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   101d
```

`kubectl get deploy`  
```console
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
logger       1/1     1            1           101d
logshipper   0/1     1            0           101d
```

`kubectl get configmap`  
```console
NAME               DATA   AGE
kube-root-ca.crt   1      101d
```

`kubectl describe deploy`  
```console
Name:                   logger
Namespace:              default
CreationTimestamp:      Mon, 23 Oct 2023 02:40:44 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=logger
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=logger
  Containers:
   logger:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      /bin/sh
      -c
      --
    Args:
      while true; do sleep 1; echo 'Hi, My brother logshipper cannot see what I am saying. Can you fix him?' ; done;
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   logger-574bd75fd9 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  101d  deployment-controller  Scaled up replica set logger-574bd75fd9 to 1


Name:                   logshipper
Namespace:              default
CreationTimestamp:      Mon, 23 Oct 2023 02:40:44 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 3
Selector:               app=logshipper
Replicas:               1 desired | 1 updated | 1 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app=logshipper
  Annotations:      kubectl.kubernetes.io/restartedAt: 2023-10-23T02:50:49Z
  Service Account:  logshipper-sa
  Containers:
   logshipper:
    Image:      logshipper:v3
    Port:       <none>
    Host Port:  <none>
    Command:
      /usr/bin/python3
      logshipper.py
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      False   MinimumReplicasUnavailable
OldReplicaSets:  logshipper-7794979fdc (0/0 replicas created), logshipper-c6fd8ff89 (0/0 replicas created)
NewReplicaSet:   logshipper-7d97c8659 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  101d  deployment-controller  Scaled up replica set logshipper-7794979fdc to 1
  Normal  ScalingReplicaSet  101d  deployment-controller  Scaled up replica set logshipper-c6fd8ff89 to 1
  Normal  ScalingReplicaSet  101d  deployment-controller  Scaled down replica set logshipper-7794979fdc to 0 from 1
  Normal  ScalingReplicaSet  101d  deployment-controller  Scaled up replica set logshipper-7d97c8659 to 1
  Normal  ScalingReplicaSet  101d  deployment-controller  Scaled down replica set logshipper-c6fd8ff89 to 0 from 1
```

`find / -type f -name "logshipper.py"`  
```console
/var/lib/rancher/k3s/agent/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/54/fs/home/ubuntu/logshipper.py
```

`cat /var/lib/rancher/k3s/agent/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/54/fs/home/ubuntu/logshipper.py`  
```console
from kubernetes import client, config
from kubernetes.client.rest import ApiException
import time,sys 

# Configs can be set in Configuration class directly or using helper utility
config.load_incluster_config()

#def list_pods():
#    kv1 = client.CoreV1Api()
#    #v1 = client.AppsV1Api()
#    ret = kv1.list_namespaced_pod(namespace="default")
#    for i in ret.items:
#        print(i.metadata.name , i.metadata.namespace)


def read_log():
    api_instance = client.CoreV1Api()
    deployment_name = "logger" # str | name of the deploy,ent
    namespace = "default" # str | object name and auth scope, such as for teams and projects
    label_selector = "app=logger"
    container = "logger" # str | The container for which to stream logs. Defaults to only container if there is one container in the pod. (optional)
    follow = True # bool | Follow the log stream of the pod. Defaults to false. (optional)
    insecure_skip_tls_verify_backend = True # bool | insecureSkipTLSVerifyBackend indicates that the apiserver should not confirm the validity of the serving certificate of the backend it is connecting to.  This will make the HTTPS connection between the apiserver and the backend insecure. This means the apiserver cannot verify the log data it is receiving came from the real kubelet.  If the kubelet is configured to verify the apiserver's TLS credentials, it does not mean the connection to the real kubelet is vulnerable to a man in the middle attack (e.g. an attacker could not intercept the actual log data coming from the real kubelet). (optional)
    limit_bytes = 200 # int | If set, the number of bytes to read from the server before terminating the log output. This may not display a complete final line of logging, and may return slightly more or slightly less than the specified limit. (optional)
    pretty = 'pretty_example' # str | If 'true', then the output is pretty printed. (optional)
    previous = False # bool | Return previous terminated container logs. Defaults to false. (optional)
    since_seconds = 1 # int | A relative time in seconds before the current time from which to show logs. If this value precedes the time a pod was started, only logs since the pod start will be returned. If this value is in the future, no logs will be returned. Only one of sinceSeconds or sinceTime may be specified. (optional)
    tail_lines = 1 # int | If set, the number of lines from the end of the logs to show. If not specified, logs are shown from the creation of the container or sinceSeconds or sinceTime (optional)
    timestamps = False # bool | If true, add an RFC3339 or RFC3339Nano timestamp at the beginning of every line of log output. Defaults to false. (optional)

    try:
        pods = api_instance.list_namespaced_pod(namespace, label_selector=label_selector)
        if pods.items:
            pod = pods.items[0]
            api_response = api_instance.read_namespaced_pod_log(
                    pod.metadata.name,
                    namespace,
                    container="logger",
                    since_seconds=1
                )
            print("Thanks for fixing me! Logger is saying:")
            print(api_response)
        else:
            print("No pod matching the label selector found.")
    except ApiException as e:
        print("Exception when calling CoreV1Api->read_namespaced_pod_log: %s\n" % e)
        sys.exit(1)

while(True):
    read_log()
```

`kubectl get po`  
```console
NAME                         READY   STATUS             RESTARTS        AGE 
logger-574bd75fd9-wpg4w      1/1     Running            5 (4m18s ago)   101d
logshipper-7d97c8659-npscd   0/1     CrashLoopBackOff   23 (36s ago)    101d
```

`kubectl logs logshipper-7d97c8659-npscd`  
```console
<no output>
```

`kubectl logs logger-574bd75fd9-wpg4w`  
```console
Hi, My brother logshipper cannot see what I am saying. Can you fix him?
Hi, My brother logshipper cannot see what I am saying. Can you fix him?
Hi, My brother logshipper cannot see what I am saying. Can you fix him?
<...>
Hi, My brother logshipper cannot see what I am saying. Can you fix him?
```

`kubectl get pod --show-labels`  
```console
NAME                         READY   STATUS             RESTARTS        AGE    LABELS
logger-574bd75fd9-wpg4w      1/1     Running            5 (4m18s ago)   101d   app=logger,pod-template-hash=574bd75fd9
logshipper-7d97c8659-npscd   0/1     CrashLoopBackOff   23 (36s ago)    101d   app=logshipper,pod-template-hash=7d97c8659
```

`kubectl get ClusterRole`  
```console
NAME                                                                   CREATED AT
cluster-admin                                                          2023-10-23T02:34:11Z
system:discovery                                                       2023-10-23T02:34:11Z
system:monitoring                                                      2023-10-23T02:34:11Z
system:basic-user                                                      2023-10-23T02:34:11Z
system:public-info-viewer                                              2023-10-23T02:34:11Z
system:aggregate-to-admin                                              2023-10-23T02:34:11Z
system:aggregate-to-edit                                               2023-10-23T02:34:11Z
system:aggregate-to-view                                               2023-10-23T02:34:11Z
system:heapster                                                        2023-10-23T02:34:11Z
system:node                                                            2023-10-23T02:34:11Z
system:node-problem-detector                                           2023-10-23T02:34:11Z
system:kubelet-api-admin                                               2023-10-23T02:34:11Z
system:node-bootstrapper                                               2023-10-23T02:34:11Z
system:auth-delegator                                                  2023-10-23T02:34:11Z
system:kube-aggregator                                                 2023-10-23T02:34:11Z
system:kube-controller-manager                                         2023-10-23T02:34:11Z
system:kube-dns                                                        2023-10-23T02:34:11Z
system:persistent-volume-provisioner                                   2023-10-23T02:34:11Z
system:certificates.k8s.io:certificatesigningrequests:nodeclient       2023-10-23T02:34:11Z
system:certificates.k8s.io:certificatesigningrequests:selfnodeclient   2023-10-23T02:34:11Z
system:volume-scheduler                                                2023-10-23T02:34:11Z
system:certificates.k8s.io:legacy-unknown-approver                     2023-10-23T02:34:11Z
system:certificates.k8s.io:kubelet-serving-approver                    2023-10-23T02:34:11Z
system:certificates.k8s.io:kube-apiserver-client-approver              2023-10-23T02:34:11Z
system:certificates.k8s.io:kube-apiserver-client-kubelet-approver      2023-10-23T02:34:11Z
system:service-account-issuer-discovery                                2023-10-23T02:34:11Z
system:node-proxier                                                    2023-10-23T02:34:11Z
system:kube-scheduler                                                  2023-10-23T02:34:11Z
system:controller:attachdetach-controller                              2023-10-23T02:34:11Z
system:controller:clusterrole-aggregation-controller                   2023-10-23T02:34:11Z
system:controller:cronjob-controller                                   2023-10-23T02:34:11Z
system:controller:daemon-set-controller                                2023-10-23T02:34:11Z
system:controller:deployment-controller                                2023-10-23T02:34:12Z
system:controller:disruption-controller                                2023-10-23T02:34:12Z
system:controller:endpoint-controller                                  2023-10-23T02:34:12Z
system:controller:endpointslice-controller                             2023-10-23T02:34:12Z
system:controller:endpointslicemirroring-controller                    2023-10-23T02:34:12Z
system:controller:expand-controller                                    2023-10-23T02:34:12Z
system:controller:ephemeral-volume-controller                          2023-10-23T02:34:12Z
system:controller:generic-garbage-collector                            2023-10-23T02:34:12Z
system:controller:horizontal-pod-autoscaler                            2023-10-23T02:34:12Z
system:controller:job-controller                                       2023-10-23T02:34:12Z
system:controller:namespace-controller                                 2023-10-23T02:34:12Z
system:controller:node-controller                                      2023-10-23T02:34:12Z
system:controller:persistent-volume-binder                             2023-10-23T02:34:12Z
system:controller:pod-garbage-collector                                2023-10-23T02:34:12Z
system:controller:replicaset-controller                                2023-10-23T02:34:12Z
system:controller:replication-controller                               2023-10-23T02:34:12Z
system:controller:resourcequota-controller                             2023-10-23T02:34:12Z
system:controller:route-controller                                     2023-10-23T02:34:12Z
system:controller:service-account-controller                           2023-10-23T02:34:12Z
system:controller:service-controller                                   2023-10-23T02:34:12Z
system:controller:statefulset-controller                               2023-10-23T02:34:12Z
system:controller:ttl-controller                                       2023-10-23T02:34:12Z
system:controller:certificate-controller                               2023-10-23T02:34:12Z
system:controller:pvc-protection-controller                            2023-10-23T02:34:12Z
system:controller:pv-protection-controller                             2023-10-23T02:34:12Z
system:controller:ttl-after-finished-controller                        2023-10-23T02:34:12Z
system:controller:root-ca-cert-publisher                               2023-10-23T02:34:12Z
k3s-cloud-controller-manager                                           2023-10-23T02:34:22Z
system:coredns                                                         2023-10-23T02:34:24Z
local-path-provisioner-role                                            2023-10-23T02:34:25Z
system:aggregated-metrics-reader                                       2023-10-23T02:34:26Z
system:metrics-server                                                  2023-10-23T02:34:26Z
clustercidrs-node                                                      2023-10-23T02:34:27Z
system:k3s-controller                                                  2023-10-23T02:34:27Z
view                                                                   2023-10-23T02:34:11Z
edit                                                                   2023-10-23T02:34:11Z
admin                                                                  2023-10-23T02:34:11Z
traefik-kube-system                                                    2023-10-23T02:35:28Z
logshipper-cluster-role                                                2023-10-23T02:40:43Z
```

`kubectl get ClusterRole logshipper-cluster-role`  
```console
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    descripton: Think about what verbs you need to add.
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"rbac.authorization.k8s.io/v1","kind":"ClusterRole","metadata":{"annotations":{"descripton":"Think about what verbs you need to add."},"name":"logshipper-cluster-role"},"rules":[{"apiGroups":[""
  creationTimestamp: "2023-10-23T02:40:43Z"
  name: logshipper-cluster-role
  resourceVersion: "2851"
  uid: eb5541f8-870c-40e0-ac57-17f7c4d24fd5
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  - pods
  - pods/log
  verbs:
  - list
```


### 2. AUTHOR'S SOLUTION
##### Edit the `logshipper-cluster-role` role (include the `get` and `watch` verbs) and restart the `logshipper` pod
#### 2.1. Edit the role
`kubectl edit ClusterRole logshipper-cluster-role`    
```console
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    descripton: Think about what verbs you need to add.
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"rbac.authorization.k8s.io/v1","kind":"ClusterRole","metadata":{"annotations":{"descripton":"Think about what verbs you need to add."},"name":"logshipper-cluster-role"},"rules":[{"apiGroups":[""
  creationTimestamp: "2023-10-23T02:40:43Z"
  name: logshipper-cluster-role
  resourceVersion: "2851"
  uid: eb5541f8-870c-40e0-ac57-17f7c4d24fd5
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  - pods
  - pods/log
  verbs:
  - list
  - watch
  - get
```
```console
clusterrole.rbac.authorization.k8s.io/logshipper-cluster-role edited
```

#### 2.2. Restart the pod
`kubectl rollout restart deployment logshipper`  
```console
deployment.apps/logshipper restarted
```

`kubectl get po`  
```console
NAME                         READY   STATUS    RESTARTS        AGE
logger-574bd75fd9-wpg4w      1/1     Running   5 (23s ago)     101d
logshipper-7d97c8659-npscd   1/1     Running   20 (101d ago)   101d
```


#### 3. Validate the task
`cd /home/admin/agent/ && ./check.sh`  
```console
OK
```


`kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].state.waiting.reason // .status.phase)"'`  
```console
Running
```
