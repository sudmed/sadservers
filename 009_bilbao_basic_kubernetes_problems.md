# "Bilbao": Basic Kubernetes Problems

**Description:** There's a Kubernetes Deployment with an Nginx pod and a Load Balancer declared in the manifest.yml file. The pod is not coming up. Fix it so that you can access the Nginx container through the Load Balancer.  

There's no "sudo" (root) access.  

**Test:** Running curl 10.43.216.196 returns the default Nginx Welcome page.  

See `/home/admin/agent/check.sh` for the test that "Check My Solution" runs.  


---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 44
drwxr-xr-x 6 admin admin 4096 Jan 18 17:11 .
drwxr-xr-x 3 root  root  4096 Sep 17 16:44 ..
drwx------ 3 admin admin 4096 Sep 17 17:15 .ansible
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin 4096 Jan 18 17:11 .config
-rw-r--r-- 1 admin admin  807 Aug  4  2021 .profile
drwx------ 2 admin admin 4096 Sep 17 16:44 .ssh
-rw-r--r-- 1 admin admin  833 Jan 17 23:55 README.txt
drwxr-xr-x 2 admin root  4096 Jan 17 23:55 agent
-rw-r--r-- 1 admin admin  755 Jan 17 23:54 manifest.yml
```

`cat manifest.yml`  
```console
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: localhost:5000/nginx
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: 2000Mi
            cpu: 100m
          requests:
            cpu: 100m
            memory: 2000Mi
      nodeSelector:
        disk: ssd

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  clusterIP: 10.43.216.196
  type: LoadBalancer
```

`cd ~/agent && cat check.sh`  
```console
#!/usr/bin/bash
# DO NOT MODIFY THIS FILE ("Check My Solution" will fail)

res=$(kubectl get pods -l app=nginx --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].state.waiting.reason // .status.phase)"')
res=$(echo $res|tr -d '\r')

if [[ "$res" != "Running" ]]
then
  echo -n "NO"
  exit 1
fi

res=$(curl -s 10.43.216.196:80|grep -c Welcome)
if [[ "$res" -eq 2 ]]
then
  echo -n "OK"
else
  echo "NO"
fi
```

`curl 10.43.216.196`  
```console
curl: (7) Failed to connect to 10.43.216.196 port 80: Connection refused
```


#### 2. Check Kubernetes objects
`kubectl cluster-info`  
```console
Kubernetes control plane is running at https://127.0.0.1:6443
CoreDNS is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

`kubectl get no`  
```console
NAME                  STATUS     ROLES                  AGE   VERSION
node1                 Ready      control-plane,master   17h   v1.28.5+k3s1
i-02f8e6680f7d5e616   NotReady   control-plane,master   17h   v1.28.5+k3s1
```

`kubectl get po`  
```console
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6c8857bf89-dc82j   0/1     Pending   0          17h
```

`kubectl get services`  
```console
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
kubernetes      ClusterIP      10.43.0.1       <none>          443/TCP        17h
nginx-service   LoadBalancer   10.43.216.196   172.31.40.252   80:31577/TCP   17h
```

`kubectl get deployments`  
```console
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/1     1            0           17h
```

`kubectl get ns`  
```console
NAME              STATUS   AGE
kube-system       Active   17h
kube-public       Active   17h
kube-node-lease   Active   17h
default           Active   17h
```

`kubectl get all -o wide`  
```console
NAME                                    READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
pod/nginx-deployment-6c8857bf89-dc82j   0/1     Pending   0          17h   <none>   <none>   <none>           <none>

NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE   SELECTOR
service/kubernetes      ClusterIP      10.43.0.1       <none>          443/TCP        17h   <none>
service/nginx-service   LoadBalancer   10.43.216.196   172.31.39.199   80:31577/TCP   17h   app=nginx

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                 SELECTOR
deployment.apps/nginx-deployment   0/1     1            0           17h   nginx        localhost:5000/nginx   app=nginx

NAME                                          DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                 SELECTOR
replicaset.apps/nginx-deployment-67699598cc   1         1         0       17h   nginx        localhost:5000/nginx   app=nginx,pod-template-hash=67699598cc
```

`kubectl describe pod/nginx-deployment-6c8857bf89-dc82j`  
```console
Name:             nginx-deployment-6c8857bf89-dc82j
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           app=nginx
                  pod-template-hash=67699598cc
Annotations:      <none>
Status:           Pending
IP:               
IPs:              <none>
Controlled By:    ReplicaSet/nginx-deployment-67699598cc
Containers:
  nginx:
    Image:      localhost:5000/nginx
    Port:       80/TCP
    Host Port:  0/TCP
    Limits:
      cpu:     100m
      memory:  2000Mi
    Requests:
      cpu:        100m
      memory:     2000Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mslhc (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  kube-api-access-mslhc:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              disk=ssd
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  17h   default-scheduler  0/1 nodes are available: 1 node(s) didn't match Pod's node affinity/selector, 1 node(s) had untolerated taint {node.kubernetes.io/unreachable: }. preemption: 0/1 nodes are available: 2 Preemption is not helpful for scheduling..
```


#### 3. As we can see in `kubectl describe pod` command, Pod's node affinity/selector doesn't match. Let's add the missing label to the node
`kubectl label nodes node1 disk=ssd`  
```console
node/node1 labeled
```


#### 4. Restart the deployment
`kubectl delete deploy nginx-deployment`  
```console
deployment.apps "nginx-deployment" deleted
```

`kubectl apply -f manifest.yml`  
```console
deployment.apps/nginx-deployment created
service/nginx-service unchanged
```

`kubectl get po`  
```console
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-67699598cc-vpsdn   0/1     Pending   0          8s
```

`kubectl describe pod nginx-deployment-67699598cc-vpsdn`  
```console
Name:             nginx-deployment-67699598cc-vpsdn
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           app=nginx
                  pod-template-hash=67699598cc
Annotations:      <none>
Status:           Pending
IP:               
IPs:              <none>
Controlled By:    ReplicaSet/nginx-deployment-67699598cc
Containers:
  nginx:
    Image:      localhost:5000/nginx
    Port:       80/TCP
    Host Port:  0/TCP
    Limits:
      cpu:     100m
      memory:  2000Mi
    Requests:
      cpu:        100m
      memory:     2000Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zd29h (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  kube-api-access-zd29h:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              disk=ssd
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  36s   default-scheduler  0/2 nodes are available: 1 Insufficient memory, 1 node(s) had untolerated taint {node.kubernetes.io/unreachable: }. preemption: 0/2 nodes are available: 1 No preemption victims found for incoming pod, 1 Preemption is not helpful for scheduling..
```


#### 5. As we can see in `kubectl describe pod` command, there is `Insufficient memory` warning. Let's change memory limits for the pod
`vi manifest.yml`  
```console
<...>
        resources:
          limits:
            memory: 200Mi
            cpu: 100m
          requests:
            cpu: 100m
            memory: 200Mi
<...>
```

`kubectl get po`  
```console
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-748f7447d5-bdsk5   1/1     Running   0          50s
admin@i-04d5c03c68726d634:~$ kubectl describe pod nginx-deployment-748f7447d5-bdsk5
Name:             nginx-deployment-748f7447d5-bdsk5
Namespace:        default
Priority:         0
Service Account:  default
Node:             node1/172.31.32.137
Start Time:       Thu, 18 Jan 2024 18:14:25 +0000
Labels:           app=nginx
                  pod-template-hash=748f7447d5
Annotations:      <none>
Status:           Running
IP:               10.42.1.14
IPs:
  IP:           10.42.1.14
Controlled By:  ReplicaSet/nginx-deployment-748f7447d5
Containers:
  nginx:
    Container ID:   containerd://37cd08a166fcd240b0025b08faf21968c4612701f80a004d6cbbf3724fb64cda
    Image:          localhost:5000/nginx
    Image ID:       localhost:5000/nginx@sha256:9ceac672f55cdad7316651fcced0d6a1ba44e87454ec86236cb3bca647e419f6
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 18 Jan 2024 18:14:38 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     100m
      memory:  200Mi
    Requests:
      cpu:        100m
      memory:     200Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wbxpk (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-wbxpk:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              disk=ssd
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  72s   default-scheduler  Successfully assigned default/nginx-deployment-748f7447d5-bdsk5 to node1
  Normal  Pulling    72s   kubelet            Pulling image "localhost:5000/nginx"
  Normal  Pulled     61s   kubelet            Successfully pulled image "localhost:5000/nginx" in 11.537s (11.537s including waiting)
  Normal  Created    60s   kubelet            Created container nginx
  Normal  Started    60s   kubelet            Started container nginx
```


#### 6. Validate the task
`curl -I 10.43.216.196`  
```console
HTTP/1.1 200 OK
Server: nginx/1.25.3
Date: Thu, 18 Jan 2024 18:16:05 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 24 Oct 2023 13:46:47 GMT
Connection: keep-alive
ETag: "6537cac7-267"
Accept-Ranges: bytes
```

`cd ~/agent && ./check.sh`  
```console
OK
```

`curl 10.43.216.196`  
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
