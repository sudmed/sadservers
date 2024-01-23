# "Salta": Docker container won't start.

**Description:** There's a "dockerized" Node.js web application in the `/home/admin/app` directory. Create a Docker container so you get a web app on port `:8888` and can curl to it. For the solution to be valid, there should be only one running Docker container.  

**Test:** `curl localhost:8888` returns `Hello World!` from a running container.  


---

### Solution:
#### 1. Reconnaissance on the server
`cat /home/admin/agent/check.sh `  
```console
#!/usr/bin/bash
res=$(curl -s localhost:8888)
res2=$(sudo docker ps --format "{{.Ports}}" |grep -c '8888->8888/tcp')
res2=$(echo $res2|tr -d '\r')

if [[ "$res" = "Hello World!" && "$res2" = "1" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`docker ps`  
```console
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied
```

`sudo su`  
`docker ps`  
```console
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

`cd /home/admin/app && ls -la`  
```console
total 120
drwxr-xr-x 2 admin admin  4096 Sep 16  2022 .
drwxr-xr-x 6 admin admin  4096 Sep 16  2022 ..
-rw-r--r-- 1 admin admin    28 Sep 16  2022 .dockerignore
-rw-r--r-- 1 admin admin   523 Sep 16  2022 Dockerfile
-rw-r--r-- 1 admin admin 95602 Sep 16  2022 package-lock.json
-rw-r--r-- 1 admin admin   274 Sep 16  2022 package.json
-rw-r--r-- 1 admin admin   442 Sep 16  2022 server.js
```

`cat Dockerfile`  
```console
# documentation https://nodejs.org/en/docs/guides/nodejs-docker-webapp/

# most recent node (security patches) and alpine (minimal, adds to security, possible libc issues)
FROM node:15.7-alpine 

# Create app directory & copy app files
WORKDIR /usr/src/app

# we copy first package.json only, so we take advantage of cached Docker layers
COPY ./package*.json ./

# RUN npm ci --only=production
RUN npm install

# Copy app source
COPY ./* ./

# port used by this app
EXPOSE 8880

# command to run
CMD [ "node", "serve.js" ]
```

#### 2. Fix Dockerfile exposed port and command to run
```console
# port used by this app
EXPOSE 8888

# command to run
CMD [ "node", "server.js" ]
```


#### 3. Build image
`docker build -t myapp:v1 .`  
```console
Sending build context to Docker daemon  101.9kB
Step 1/7 : FROM node:15.7-alpine
 ---> 706d12284dd5
Step 2/7 : WORKDIR /usr/src/app
 ---> Using cache
 ---> 463b1571f18e
Step 3/7 : COPY ./package*.json ./
 ---> Using cache
 ---> acfb467c80ba
Step 4/7 : RUN npm install
 ---> Using cache
 ---> 5cad5aa08c7a
Step 5/7 : COPY ./* ./
 ---> 017410373a0b
Step 6/7 : EXPOSE 8880
 ---> Running in 3f5d151ee92b
Removing intermediate container 3f5d151ee92b
 ---> 1cc61a5074cd
Step 7/7 : CMD [ "node", "server.js" ]
 ---> Running in 1ec6099ee479
Removing intermediate container 1ec6099ee479
 ---> 1403eba144b1
Successfully built 1403eba144b1
Successfully tagged myapp:v1
```


#### 4. Run container from image
`docker run -d --rm -p 8888:8888 myapp:v1`  
```console
20b31769c3af7d2a1ac0a9fdd9a4b63619a75803b070689b7184d6e932e452f9
```

`docker ps`  
```console
CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS          PORTS                                       NAMES
20b31769c3af   myapp:v1   "docker-entrypoint.s…"   12 seconds ago   Up 11 seconds   0.0.0.0:8888->8888/tcp, :::8888->8888/tcp   thirsty_jang
```

`docker logs 20b31769c3af`  
```console
Server Started on: 8888
```


#### 5. Validate the task
`curl localhost:8888`  
```console
Hello World!
```

`cd /home/admin/agent/ && ./check.sh `  
```console
OK
```
