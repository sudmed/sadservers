# "Tarifa": Between Two Seas

**Description:** There are three Docker containers defined in the `docker-compose.yml` file: an HAProxy accepting connetions on port `:5000` of the host, and two nginx containers, not exposed to the host.  
The person who tried to set this up wanted to have HAProxy in front of the (backend or upstream) nginx containers load-balancing them but something is not working.  

**Test:** Running `curl localhost:5000` several times returns both `hello there from `nginx_0` and `hello there from nginx_1`.  
Check `/home/admin/agent/check.sh` for the test that "Check My Solution" runs.


---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 60
drwxr-xr-x 7 admin admin 4096 Nov 23 03:19 .
drwxr-xr-x 3 root  root  4096 Nov 22 01:04 ..
drwx------ 3 admin admin 4096 Nov 22 01:06 .ansible
-rw------- 1 admin admin    0 Nov 23 02:25 .bash_history
-rw-r--r-- 1 admin admin  220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 admin admin 3526 Mar 27  2022 .bashrc
drwxr-xr-x 3 admin admin 4096 Nov 23 02:21 .local
-rw-r--r-- 1 admin admin  807 Mar 27  2022 .profile
drwx------ 2 admin admin 4096 Nov 22 01:04 .ssh
-rw-r--r-- 1 admin admin  969 Nov 23 03:19 README.txt
drwxr-xr-x 2 admin root  4096 Nov 23 01:58 agent
-rw-r--r-- 1 admin admin  142 Nov 23 01:58 custom-nginx_0.conf
-rw-r--r-- 1 admin admin  142 Nov 23 01:58 custom-nginx_1.conf
drwxrwxrwx 4 admin admin 4096 Nov 23 01:58 custom_index
-rw-r--r-- 1 admin admin  892 Nov 23 02:22 docker-compose.yml
-rw-r--r-- 1 admin admin  371 Nov 23 01:58 haproxy.cfg
```

`cat ~/custom_index/nginx_0/index.html `  
```console
hello there from nginx_0
```

`cat ~/custom_index/nginx_1/index.html `  
```console
hello there from nginx_1
```

`cat agent/check.sh`  
```console
#!/bin/bash
# DO NOT MODIFY THIS FILE ("Check My Solution" will fail)

found=0

for i in {1..4}
do
  res=$(curl -s localhost:5000)
  ex=$?
  if test "$ex" != "0"; then
     echo -n "NO"
     exit
  fi

  res2=$(echo $res|grep -s nginx_0)
  ex=$?
  if test "$ex" != "0"; then
     continue
  else
    found=1
    break
  fi
done

if test "$found" != "1"; then
   echo -n "NO"
   exit
fi

for i in {1..4}
do
  res=$(curl -s localhost:5000)
  ex=$?
  if test "$ex" != "0"; then
     echo -n "NO"
     exit
  fi

  res2=$(echo $res|grep -s nginx_1)
  ex=$?
  if test "$ex" != "0"; then
     continue
  else
    echo -n "OK"
    exit
  fi
done

echo -n "NO"
```

`cat haproxy.cfg`  
```console
global
    daemon
    maxconn 256

defaults
    mode http
    default-server init-addr last,libc,none
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend http-in
    bind *:5000
    default_backend nginx_backends

backend nginx_backends
    balance roundrobin
    server nginx_0 nginx_0:80 check
    server nginx_1 nginx_1:80 check
```
#####  ↑ As we can see, roundrobin is made between two identical ports (80).


`cat docker-compose.yml`  
```console
version: '3'

services:
  nginx_0:
    image: nginx:1.25.3
    container_name: nginx_0
    restart: always
    volumes:
      - ./custom_index/nginx_0:/usr/share/nginx/html
      - ./custom-nginx_0.conf:/etc/nginx/conf.d/default.conf:ro
    networks:
      - frontend_network

  nginx_1:
    image: nginx:1.25.3
    container_name: nginx_1
    restart: always
    volumes:
      - ./custom_index/nginx_1:/usr/share/nginx/html
      - ./custom-nginx_1.conf:/etc/nginx/conf.d/default.conf:ro
    networks:
      - backend_network

  haproxy:
    image: haproxy:2.8.4
    container_name: haproxy
    restart: always
    ports:
      - "5000:5000"
    depends_on:
      - nginx_0
      - nginx_1
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
    networks:
      - frontend_network

networks:
  frontend_network:
    driver: bridge
  backend_network:
    driver: bridge
```
##### ↑ As we can see, 3 services use 2 different networks.


`cat custom-nginx_0.conf`  
```console
server {
    listen 80;

    server_name localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }
}
```

`cat custom-nginx_1.conf`  
```console
server {
    listen 81;

    server_name localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }
}
```
##### ↑ OK, one error has been detected, the listening port of one of the nginx servers is 81.


`docker ps`  
```console
CONTAINER ID   IMAGE           COMMAND                  CREATED        STATUS         PORTS                                       NAMES
c79c9eb25143   haproxy:2.8.4   "docker-entrypoint.s…"   2 months ago   Up 6 minutes   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   haproxy
4052b12402a3   nginx:1.25.3    "/docker-entrypoint.…"   2 months ago   Up 6 minutes   80/tcp                                      nginx_1
2624cd20f3c4   nginx:1.25.3    "/docker-entrypoint.…"   2 months ago   Up 6 minutes   80/tcp                                      nginx_0
```

`curl localhost:5000` `(several times)`  
```console
hello there from nginx_0
hello there from nginx_0
hello there from nginx_0
hello there from nginx_0
```


#### 2. Fix docker-compose.yml
##### Change the `nginx_1` service network from `backend_network` to `frontend_network` and remove an unused network `backend_network` altogether
`vi docker-compose.yml`  
```console
version: '3'

services:
  nginx_0:
    image: nginx:1.25.3
    container_name: nginx_0
    restart: always
    volumes:
      - ./custom_index/nginx_0:/usr/share/nginx/html
      - ./custom-nginx_0.conf:/etc/nginx/conf.d/default.conf:ro
    networks:
      - frontend_network

  nginx_1:
    image: nginx:1.25.3
    container_name: nginx_1
    restart: always
    volumes:
      - ./custom_index/nginx_1:/usr/share/nginx/html
      - ./custom-nginx_1.conf:/etc/nginx/conf.d/default.conf:ro
    networks:
      - frontend_network

  haproxy:
    image: haproxy:2.8.4
    container_name: haproxy
    restart: always
    ports:
      - "5000:5000"
    depends_on:
      - nginx_0
      - nginx_1
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
    networks:
      - frontend_network

networks:
  frontend_network:
    driver: bridge
```


#### 3. Fix HAproxy backend
##### Replace the wrong port `80` with the correct one `81` for nginx_1 server
`vi haproxy.cfg`  
```console
global
    daemon
    maxconn 256

defaults
    mode http
    default-server init-addr last,libc,none
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend http-in
    bind *:5000
    default_backend nginx_backends

backend nginx_backends
    balance roundrobin
    server nginx_0 nginx_0:80 check
    server nginx_1 nginx_1:81 check
```


#### 4. Restart containers
`docker compose up -d`  
```console
[+] Running 3/3
 ✔ Container nginx_1  Started                                                                                                                                                                                    0.3s 
 ✔ Container nginx_0  Running                                                                                                                                                                                    0.0s 
 ✔ Container haproxy  Started                                                                                                                                                                                    0.3s 
```


#### 5. Validate the task
`curl localhost:5000` `(several times)` 
```console
hello there from nginx_0
hello there from nginx_1
hello there from nginx_0
hello there from nginx_1
```

`./agent/check.sh `  
```console
OK
```
