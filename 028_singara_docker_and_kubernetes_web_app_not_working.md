# "Singara": Docker and Kubernetes web app not working.

**Description:** There's a k3s Kubernetes install you can access with kubectl. The Kubernetes YAML manifests under /home/admin have been applied. The objective is to access from the host the "webapp" web server deployed and find what message it serves (it's a name of a town or city btw). In order to pass the check, the webapp Docker container should not be run separately outside Kubernetes as a shortcut.

**Test:** curl localhost:8888 returns a value from the webapp deployed Kubernetes pod.


---


### Solution:
#### 1. Reconnaissance on the server
`sudo -i && cd /home/admin/ && ls -la`  
```console
total 52
drwxr-xr-x 6 admin admin 4096 Sep 18  2022 .
drwxr-xr-x 3 root  root  4096 Aug 31  2022 ..
-rw------- 1 admin admin    5 Sep 18  2022 .bash_history
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3561 Sep 17  2022 .bashrc
drwxr-x--- 3 admin admin 4096 Sep 17  2022 .kube
drwxr-xr-x 3 admin admin 4096 Sep 17  2022 .local
-rw-r--r-- 1 admin admin  807 Aug  4  2021 .profile
drwx------ 2 admin admin 4096 Aug 31  2022 .ssh
drwxr-xr-x 2 admin admin 4096 Sep 18  2022 agent
-rw-r--r-- 1 admin admin  366 Sep 18  2022 deployment.yml
-rw-r--r-- 1 root  root    54 Sep 17  2022 namespace.yml
-rw-r--r-- 1 admin admin  219 Sep 17  2022 nodeport.yml
```

`curl localhost`  
```console
404 page not found
```

`curl localhost:8888`  
```console
curl: (7) Failed to connect to localhost port 8888: Connection refused
```

`cat agent/check.sh`  
```bash
#!/usr/bin/bash
key=$(curl -s localhost:8888)
key=$(echo $key|tr -d '\r')
res=$(echo -n $key|md5sum|awk '{print $1}')
res=$(echo $res|tr -d '\r')
res2=$(sudo docker ps -a|grep 8888)

if [[ "$res" = "c6faeb6a2f39140cccf1f69cc3be84cd" && "$res2" = "" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`cat deployment.yml`  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  namespace: web
spec:
  selector:
    matchLabels:
      app: webapp
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: webapp
        imagePullPolicy: Always
        ports:
        - containerPort: 8880
```

`cat namespace.yml`  
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: web
```

`cat nodeport.yml`  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: web
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: webapp
  ports:
    - port: 80
      targetPort: 8888
      nodePort: 30007
```

`kubectl get all`  
```console
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   507d
```

`kubectl get ns`  
```console
NAME              STATUS   AGE
default           Active   507d
kube-system       Active   507d
kube-public       Active   507d
kube-node-lease   Active   507d
web               Active   507d
```

`kubectl get all -n web`  
```console
NAME                                     READY   STATUS         RESTARTS   AGE
pod/webapp-deployment-666b67994b-5sffz   0/1     Terminating    0          507d
pod/webapp-deployment-666b67994b-9rbqn   0/1     ErrImagePull   0          2m24s

NAME                     TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/webapp-service   NodePort   10.43.35.97   <none>        80:30007/TCP   507d

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-deployment   0/1     1            0           507d

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-deployment-666b67994b   1         1         0       507d
```

`kubectl apply -f .`  
```console
deployment.apps/webapp-deployment unchanged
namespace/web unchanged
service/webapp-service unchanged
```

`kubectl describe pod -n web`  
<details>

  <summary>output</summary>

```console
Name:                      webapp-deployment-666b67994b-5sffz
Namespace:                 web
Priority:                  0
Node:                      ip-10-0-0-17/10.0.0.17
Start Time:                Sun, 18 Sep 2022 03:08:04 +0000
Labels:                    app=webapp
                           pod-template-hash=666b67994b
Annotations:               <none>
Status:                    Terminating (lasts 2m16s)
Termination Grace Period:  30s
IP:                        10.42.0.38
IPs:
  IP:           10.42.0.38
Controlled By:  ReplicaSet/webapp-deployment-666b67994b
Containers:
  webapp:
    Container ID:   
    Image:          webapp
    Image ID:       
    Port:           8880/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gqqnt (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-gqqnt:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason          Age                  From               Message
  ----     ------          ----                 ----               -------
  Normal   Scheduled       507d                 default-scheduler  Successfully assigned web/webapp-deployment-666b67994b-5sffz to ip-10-0-0-17
  Normal   Pulling         507d (x4 over 507d)  kubelet            Pulling image "webapp"
  Warning  Failed          507d (x4 over 507d)  kubelet            Failed to pull image "webapp": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/library/webapp:latest": failed to resolve reference "docker.io/library/webapp:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed          507d (x4 over 507d)  kubelet            Error: ErrImagePull
  Warning  Failed          507d (x6 over 507d)  kubelet            Error: ImagePullBackOff
  Normal   BackOff         507d (x7 over 507d)  kubelet            Back-off pulling image "webapp"
  Normal   SandboxChanged  507d                 kubelet            Pod sandbox changed, it will be killed and re-created.
  Normal   BackOff         507d (x6 over 507d)  kubelet            Back-off pulling image "webapp"
  Warning  Failed          507d (x6 over 507d)  kubelet            Error: ImagePullBackOff
  Normal   Pulling         507d (x4 over 507d)  kubelet            Pulling image "webapp"
  Warning  Failed          507d (x4 over 507d)  kubelet            Failed to pull image "webapp": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/library/webapp:latest": failed to resolve reference "docker.io/library/webapp:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed          507d (x4 over 507d)  kubelet            Error: ErrImagePull


Name:         webapp-deployment-666b67994b-9rbqn
Namespace:    web
Priority:     0
Node:         ip-10-0-0-64/172.31.32.205
Start Time:   Wed, 07 Feb 2024 07:44:05 +0000
Labels:       app=webapp
              pod-template-hash=666b67994b
Annotations:  <none>
Status:       Pending
IP:           10.42.1.5
IPs:
  IP:           10.42.1.5
Controlled By:  ReplicaSet/webapp-deployment-666b67994b
Containers:
  webapp:
    Container ID:   
    Image:          webapp
    Image ID:       
    Port:           8880/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5gdf8 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-5gdf8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  2m46s                default-scheduler  Successfully assigned web/webapp-deployment-666b67994b-9rbqn to ip-10-0-0-64
  Normal   Pulling    70s (x3 over 2m45s)  kubelet            Pulling image "webapp"
  Warning  Failed     40s (x3 over 2m15s)  kubelet            Failed to pull image "webapp": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/library/webapp:latest": failed to resolve reference "docker.io/library/webapp:latest": failed to do request: Head "https://registry-1.docker.io/v2/library/webapp/manifests/latest": dial tcp 34.226.69.105:443: i/o timeout
  Warning  Failed     40s (x3 over 2m15s)  kubelet            Error: ErrImagePull
  Normal   BackOff    5s (x5 over 2m15s)   kubelet            Back-off pulling image "webapp"
  Warning  Failed     5s (x5 over 2m15s)   kubelet            Error: ImagePullBackOff
```

</details>

##### As we can see, there is an Error: ImagePullBackOff while pulling image "webapp". Let's try to pull it manually.


#### 2. Try to pull the image
`docker pull webapp`  
```console
Using default tag: latest
Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```
##### OK, no chance. Let's take a look at the previously pulled images.

#### 3. List pulled images
`docker images --digests`  
```console
REPOSITORY   TAG        DIGEST                                                                    IMAGE ID       CREATED         SIZE
webapp       latest     <none>                                                                    9c082e2983bc   16 months ago   135MB
python       3.7-slim   sha256:48876a8a7db7ce3af5836ac638dc202f4589d300dc666f1d2507ddcda6cb5ce4   c1d0bab51bbf   17 months ago   123MB
registry     2          sha256:83bb78d7b28f1ac99c68133af32c93e9a1c149bcd3cb6e683a3ee56e312f1c96   3a0f7b0a13ef   18 months ago   24.1MB
```
##### We have an image `webapp` without digest and a registry image. We can run the registry and push `webapp` image into it, then pull it from the deployment manifest.


#### 4. Run registry and push image
`docker run -d -p 5000:5000 registry:2`  
```console
3afde6159949d6fd205921e2e0ef369f5fbfeed766e5f5f5eeb1eb297b232d15
```

`docker tag webapp localhost:5000/webapp`  
`docker push localhost:5000/webapp`  
```console
Using default tag: latest
The push refers to repository [localhost:5000/webapp]
d4a3f4c0cc92: Pushed 
2d08ca8d8c31: Pushed 
be0120f1f0a9: Pushed 
f1ec60f70b75: Pushed 
55fd06cb471b: Pushed 
b249887652be: Pushed 
95a02847aa85: Pushed 
b45078e74ec9: Pushed 
latest: digest: sha256:529296e183723c5170e832cac2c1144e3f11c15d33bee5900d88842df0232e35 size: 1995
```


#### 5. Fix image and port in deployment.yml
`vi deployment.yml`  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  namespace: web
spec:
  selector:
    matchLabels:
      app: webapp
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: localhost:5000/webapp
        imagePullPolicy: Always
        ports:
        - containerPort: 8888
```

`kubectl apply -f deployment.yml`  
```console
deployment.apps/webapp-deployment configured
```


#### 6. Forward container port to localhost
`kubectl port-forward deployments/webapp-deployment 8888 -n web &`  
```console
[1] 1835
root@ip-10-0-0-64:/home/admin# Forwarding from 127.0.0.1:8888 -> 8888
Forwarding from [::1]:8888 -> 8888
```


#### 7. Validate the task
`curl localhost:8888`  
```console
Llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogoch
```

`cd ~/agent && ./check.sh`  
```console
OK
```
