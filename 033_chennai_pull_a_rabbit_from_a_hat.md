# "Chennai": Pull a Rabbit from a Hat

**Description:** There is a RabbitMQ (RMQ) cluster defined in a `docker-compose.yml` file. Bring this system up and then run the `producer.py` script in such a way that is able to send messages to RMQ. In particular you have to send the message "hello-lwc".  

- RMQ is a queuing system: messages are put in the queue with a "producer" and they are taken out from the other side by a "consumer". The queue name has to be the same for both.  

- To send the message "hello-lwc": `python3 ~/producer.py hello-lwc`. Should return `Message sent to RabbitMQ`. "IncompatibleProtocolError" means RMQ is not working properly.  

- To test consuming it: `python3 ~/consumer.py`, this will retrieve the next message from the queue and print it. Once everything is working send more than one message so there's at least one in the queue when the validation runs.  

- Do not change the `consumer.py` and `producer.py` files; if you do the Check My Solution will fail.  

**Test:** `python3 ~/consumer.py` returns `hello-lwc`.  

See `/home/admin/agent/check.sh` for the exact test.  

---

### Solution:
#### 1. Reconnaissance on the server
`cd /home/admin && ls -la`  
```console
total 52
drwxr-xr-x 7 admin admin 4096 Sep 20 17:51 .
drwxr-xr-x 3 root  root  4096 Sep 17 16:44 ..
drwx------ 3 admin admin 4096 Sep 20 15:52 .ansible
-rw------- 1 admin admin   57 Sep 20 15:58 .bash_history
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin 4096 Sep 20 15:56 .config
-rw-r--r-- 1 admin admin  807 Aug  4  2021 .profile
drwx------ 2 admin admin 4096 Sep 17 16:44 .ssh
drwxr-xr-x 2 admin root  4096 Sep 20 17:51 agent
-rw-r--r-- 1 admin root   705 Sep 20 17:51 consumer.py
-rw-r--r-- 1 admin root   760 Sep 20 17:51 producer.py
drwxr-xr-x 2 root  root  4096 May 16  2022 rabbitmq-cluster-docker-master
```

`cat agent/check.sh`  
```bash
#!/bin/bash
res=$(nmap -p 5672 localhost|grep open|awk {'print $3'})
res=$(echo $res|tr -d '\r')

if [[ "$res" != "amqp" ]]
then
  echo -n "NO"
  exit -1
fi

res=$(md5sum /home/admin/consumer.py | awk '{print $1}')
res=$(echo $res|tr -d '\r')

if [[ "$res" != "2216a243695c9d7834bdc299e68d051c" ]]
then
  echo -n "NO"
  exit -1
fi

res=$(md5sum /home/admin/producer.py | awk '{print $1}')
res=$(echo $res|tr -d '\r')

if [[ "$res" != "86c470d5822fea6c31293f42f5e0aa34" ]]
then
  echo -n "NO"
  exit -1
fi

res=$(python3 /home/admin/consumer.py)
res=$(echo $res|tr -d '\r')

if [[ "$res" == "hello-lwc" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`cat consumer.py`  
```python
import pika

# variables
USER = 'username'
PASSWORD = 'password'
PORT = 5672
QUEUE = 'hello'

# Connection parameters
credentials = pika.PlainCredentials(USER, PASSWORD)
parameters = pika.ConnectionParameters('localhost', PORT, '/', credentials)

# Establish a connection to RabbitMQ
connection = pika.BlockingConnection(parameters)
channel = connection.channel()

# Declare a queue
channel.queue_declare(queue=QUEUE)

# Fetch a single message from the queue
method_frame, header_frame, body = channel.basic_get(queue='hello', auto_ack=True)

if method_frame:
    # Print the received message
    print(body.decode())
else:
    print("No messages in the queue")

# Close the connection
connection.close()
```

`cat producer.py`  
```python
import pika
import os
import sys

# Auth port and queue from env vars
USER = os.getenv('RMQ_USER', 'guest')
PASSWORD = os.environ.get('RMQ_PASSWORD', 'guest')
PORT = os.environ.get('RMQ_PORT', 5672)
QUEUE = os.getenv('RMQ_QUEUE')

# message is the argument
MESSAGE = sys.argv[1]

credentials = pika.PlainCredentials(USER, PASSWORD)
parameters = pika.ConnectionParameters('localhost', PORT, '/', credentials)

# Establish a connection to RabbitMQ
connection = pika.BlockingConnection(parameters)

channel = connection.channel()

# Declare a queue
channel.queue_declare(queue=QUEUE)

# Publish a message to the queue
channel.basic_publish(exchange='', routing_key=QUEUE, body=MESSAGE)
print("Message sent to RabbitMQ")

# Close the connection
connection.close()
```

`cd ~/rabbitmq-cluster-docker-master && ls -la`  
```console
total 36
drwxr-xr-x 2 root  root  4096 May 16  2022 .
drwxr-xr-x 7 admin admin 4096 Sep 20 17:51 ..
-rw-r--r-- 1 root  root    80 May 16  2022 .env
-rw-r--r-- 1 root  root     5 May 16  2022 .erlang.cookie
-rw-r--r-- 1 root  root  1062 May 16  2022 LICENSE
-rw-r--r-- 1 root  root  2039 May 16  2022 README.md
-rwxrwxrwx 1 admin root   723 May 16  2022 cluster-entrypoint.sh
-rw-r--r-- 1 root  root  1399 May 16  2022 docker-compose.yml
-rw-r--r-- 1 root  root  1298 May 16  2022 haproxy.cfg
```

`cat .env`  
```console
RABBITMQ_DEFAULT_USER=guest
RABBITMQ_DEFAULT_PASS=guest
RABBITMQ_DEFAULT_VHOST=/
```

`cat README.md`  
```console
# RabbitMQ Cluster Docker
Setup a RabbitMQ Cluster environment on your device using the pure [RabbitMQ](https://hub.docker.com/_/rabbitmq/) official docker image with Docker Compose.

## Features
- Super easy setup, config and expand
- Use a purely official RabbitMQ image
- Support latest version, optimized for Erlang cookie config
- Build-in HAProxy load balancing

## Quick start
`docker compose up`

Open http://localhost:15672 to login RabbitMQ dashboard.

> Username: `guest`  
> Password: `guest`

## Configuration
### `docker-compose.yml`

Docker [compose](https://docs.docker.com/compose/compose-file/) config file, including 3 RabbitMQ service cluster and a HAProxy.

| Service     | Description               |
| ----------- | ------------------------- |
| `rabbitmq1` | RabbitMQ (cluster)        |
| `rabbitmq2` | RabbitMQ (cluster member) |
| `rabbitmq3` | RabbitMQ (cluster member) |
| `haproxy`   | Load Balancer             |

#### Default expose ports

| Host              | Description                                         |
| ----------------- | --------------------------------------------------- |
| `localhost:5672`  | AMQP 0-9-1 and AMQP 1.0 clients                     |
| `localhost:15672` | HTTP API clients, management UI and `rabbitmqadmin` |

### `.env`

| Name                     | Default |
| ------------------------ | ------- |
| `RABBITMQ_DEFAULT_USER`  | guest   |
| `RABBITMQ_DEFAULT_PASS`  | guest   |
| `RABBITMQ_DEFAULT_VHOST` | /       |

### `.erlang.cookie`

Put your custom [Erlang Cookie](https://www.rabbitmq.com/clustering.html#erlang-cookie) inside this file (default: `12345`) for the nodes in cluster communicate with each other.

### `haproxy.cfg`

Load balancer [HA Proxy](http://www.haproxy.org/) config. Including the load balancing config and the hostnames of the nodes in cluster.

## References

- [docker-rabbitmq-cluster](https://github.com/pardahlman/docker-rabbitmq-cluster)
- [rabbitmq-cluster](https://github.com/JohnnyVicious/rabbitmq-cluster)

## LICENSE
MIT
```

`cat cluster-entrypoint.sh`  
```bash
#!/bin/bash

set -e

# Change .erlang.cookie permission
chmod 400 /var/lib/rabbitmq/.erlang.cookie

# Get hostname from enviromant variable
HOSTNAME=`env hostname`
echo "Starting RabbitMQ Server For host: " $HOSTNAME

if [ -z "$JOIN_CLUSTER_HOST" ]; then
    /usr/local/bin/docker-entrypoint.sh rabbitmq-server &
    sleep 5
    rabbitmqctl wait /var/lib/rabbitmq/mnesia/rabbit\@$HOSTNAME.pid
else
    /usr/local/bin/docker-entrypoint.sh rabbitmq-server -detached
    sleep 5
    rabbitmqctl wait /var/lib/rabbitmq/mnesia/rabbit\@$HOSTNAME.pid
    rabbitmqctl stop_app
    rabbitmqctl join_cluster rabbit@$JOIN_CLUSTER_HOST
    rabbitmqctl start_app
fi

# Keep foreground process active ...
tail -f /var/log/rabbitmq/*.log
```

`cat docker-compose.yml`  
```yaml
version: '3'
services:
  rabbitmq1:
    image: rabbitmq:3-management
    hostname: rabbitmq1
    environment:
      - RABBITMQ_DEFAULT_USER=${RABBITMQ_DEFAULT_USER}
      - RABBITMQ_DEFAULT_PASS=${RABBITMQ_DEFAULT_PASS}
      - RABBITMQ_DEFAULT_VHOST=${RABBITMQ_DEFAULT_VHOST}
    volumes:
      - ./.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie
      - ./cluster-entrypoint.sh:/usr/local/bin/cluster-entrypoint.sh
    entrypoint: /usr/local/bin/cluster-entrypoint.sh

  rabbitmq2:
    image: rabbitmq:3-management
    hostname: rabbitmq2
    depends_on:
      - rabbitmq1
    environment:
      - JOIN_CLUSTER_HOST=rabbitmq1
    volumes:
      - ./.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie
      - ./cluster-entrypoint.sh:/usr/local/bin/cluster-entrypoint.sh
    entrypoint: /usr/local/bin/cluster-entrypoint.sh

  rabbitmq3:
    image: rabbitmq:3-management
    hostname: rabbitmq3
    depends_on:
      - rabbitmq1
    environment:
      - JOIN_CLUSTER_HOST=rabbitmq1
    volumes:
      - ./.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie
      - ./cluster-entrypoint.sh:/usr/local/bin/cluster-entrypoint.sh
    entrypoint: /usr/local/bin/cluster-entrypoint.sh

  haproxy:
    image: haproxy:1.7
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
    depends_on:
      - rabbitmq1
      - rabbitmq2
      - rabbitmq3
    ports:
      - 15672:15672
      - 5672:5672
```

`cat haproxy.cfg`  
```console
global
        log 127.0.0.1   local1
        maxconn 4096
 
defaults
        log     global
        mode    tcp
        option  tcplog
        retries 3
        option redispatch
        maxconn 2000
        timeout connect 5000
        timeout client 50000
        timeout server 50000
 
listen  stats
        bind *:1936
        mode http
        stats enable
        stats hide-version
        stats realm Haproxy\ Statistics
        stats uri /
 
listen rabbitmq
        bind *:5672
        mode            tcp
        balance         roundrobin
        timeout client  3h
        timeout server  3h
        option          clitcpka
        server          rabbitmq1 rabbitmq1:5672  check inter 5s rise 2 fall 3
        server          rabbitmq2 rabbitmq2:5672  check inter 5s rise 2 fall 3
        server          rabbitmq3 rabbitmq3:5672  check inter 5s rise 2 fall 3

listen mgmt
        bind *:15672
        mode            tcp
        balance         roundrobin
        timeout client  3h
        timeout server  3h
        option          clitcpka
        server          rabbitmq1 rabbitmq1:15672  check inter 5s rise 2 fall 3
        server          rabbitmq2 rabbitmq2:15672  check inter 5s rise 2 fall 3
        server          rabbitmq3 rabbitmq3:15672  check inter 5s rise 2 fall 3
```

`docker ps`  
```console
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

`docker compose up`  

<details>

  <summary>output</summary>

```console
[+] Running 5/5
 ✔ Network rabbitmq-cluster-docker-master_default        Created                                                                                                                                                0.1s 
 ✔ Container rabbitmq-cluster-docker-master-rabbitmq1-1  Created                                                                                                                                                0.1s 
 ✔ Container rabbitmq-cluster-docker-master-rabbitmq2-1  Created                                                                                                                                                0.2s 
 ✔ Container rabbitmq-cluster-docker-master-rabbitmq3-1  Created                                                                                                                                                0.2s 
 ✔ Container rabbitmq-cluster-docker-master-haproxy-1    Created                                                                                                                                                0.0s 
Attaching to rabbitmq-cluster-docker-master-haproxy-1, rabbitmq-cluster-docker-master-rabbitmq1-1, rabbitmq-cluster-docker-master-rabbitmq2-1, rabbitmq-cluster-docker-master-rabbitmq3-1
rabbitmq-cluster-docker-master-rabbitmq1-1  | Starting RabbitMQ Server For host:  rabbitmq1
rabbitmq-cluster-docker-master-rabbitmq2-1  | Starting RabbitMQ Server For host:  rabbitmq2
rabbitmq-cluster-docker-master-rabbitmq3-1  | Starting RabbitMQ Server For host:  rabbitmq3
rabbitmq-cluster-docker-master-haproxy-1    | <7>haproxy-systemd-wrapper: executing /usr/local/sbin/haproxy -p /run/haproxy.pid -db -f /usr/local/etc/haproxy/haproxy.cfg -Ds 
rabbitmq-cluster-docker-master-rabbitmq1-1  | Waiting for pid file '/var/lib/rabbitmq/mnesia/rabbit@rabbitmq1.pid' to appear
rabbitmq-cluster-docker-master-rabbitmq3-1  | Waiting for pid file '/var/lib/rabbitmq/mnesia/rabbit@rabbitmq3.pid' to appear
rabbitmq-cluster-docker-master-rabbitmq2-1  | Waiting for pid file '/var/lib/rabbitmq/mnesia/rabbit@rabbitmq2.pid' to appear
rabbitmq-cluster-docker-master-rabbitmq1-1  | pid is 26
rabbitmq-cluster-docker-master-rabbitmq1-1  | Waiting for erlang distribution on node 'rabbit@rabbitmq1' while OS process '26' is running
rabbitmq-cluster-docker-master-rabbitmq1-1  | Waiting for applications 'rabbit_and_plugins' to start on node 'rabbit@rabbitmq1'
rabbitmq-cluster-docker-master-rabbitmq2-1  | pid is 28
rabbitmq-cluster-docker-master-rabbitmq2-1  | Waiting for erlang distribution on node 'rabbit@rabbitmq2' while OS process '28' is running
rabbitmq-cluster-docker-master-rabbitmq3-1  | pid is 28
rabbitmq-cluster-docker-master-rabbitmq3-1  | Waiting for erlang distribution on node 'rabbit@rabbitmq3' while OS process '28' is running
rabbitmq-cluster-docker-master-rabbitmq2-1  | Waiting for applications 'rabbit_and_plugins' to start on node 'rabbit@rabbitmq2'
rabbitmq-cluster-docker-master-rabbitmq3-1  | Waiting for applications 'rabbit_and_plugins' to start on node 'rabbit@rabbitmq3'
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.056429+00:00 [notice] <0.44.0> Application syslog exited with reason: stopped
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.072414+00:00 [notice] <0.230.0> Logging: switching to configured handler(s); following messages may not be visible in this log output
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.076616+00:00 [notice] <0.230.0> Logging: configured log handlers are now ACTIVE
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.640201+00:00 [info] <0.230.0> ra: starting system quorum_queues
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.640310+00:00 [info] <0.230.0> starting Ra system: quorum_queues in directory: /var/lib/rabbitmq/mnesia/rabbit@rabbitmq1/quorum/rabbit@rabbitmq1
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.817225+00:00 [info] <0.309.0> ra system 'quorum_queues' running pre init for 0 registered servers
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.850127+00:00 [info] <0.310.0> ra: meta data store initialised for system quorum_queues. 0 record(s) recovered
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.885589+00:00 [notice] <0.316.0> WAL: ra_log_wal init, open tbls: ra_log_open_mem_tables, closed tbls: ra_log_closed_mem_tables
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.902787+00:00 [info] <0.230.0> ra: starting system coordination
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.902866+00:00 [info] <0.230.0> starting Ra system: coordination in directory: /var/lib/rabbitmq/mnesia/rabbit@rabbitmq1/coordination/rabbit@rabbitmq1
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.907466+00:00 [info] <0.323.0> ra system 'coordination' running pre init for 0 registered servers
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.912359+00:00 [info] <0.324.0> ra: meta data store initialised for system coordination. 0 record(s) recovered
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.912704+00:00 [notice] <0.329.0> WAL: ra_coordination_log_wal init, open tbls: ra_coordination_log_open_mem_tables, closed tbls: ra_coordination_log_closed_mem_tables
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.970987+00:00 [info] <0.230.0> 
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.970987+00:00 [info] <0.230.0>  Starting RabbitMQ 3.12.4 on Erlang 25.3.2.6 [jit]
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.970987+00:00 [info] <0.230.0>  Copyright (c) 2007-2023 VMware, Inc. or its affiliates.
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.970987+00:00 [info] <0.230.0>  Licensed under the MPL 2.0. Website: https://rabbitmq.com
rabbitmq-cluster-docker-master-rabbitmq1-1  | 
rabbitmq-cluster-docker-master-rabbitmq1-1  |   ##  ##      RabbitMQ 3.12.4
rabbitmq-cluster-docker-master-rabbitmq1-1  |   ##  ##
rabbitmq-cluster-docker-master-rabbitmq1-1  |   ##########  Copyright (c) 2007-2023 VMware, Inc. or its affiliates.
rabbitmq-cluster-docker-master-rabbitmq1-1  |   ######  ##
rabbitmq-cluster-docker-master-rabbitmq1-1  |   ##########  Licensed under the MPL 2.0. Website: https://rabbitmq.com
rabbitmq-cluster-docker-master-rabbitmq1-1  | 
rabbitmq-cluster-docker-master-rabbitmq1-1  |   Erlang:      25.3.2.6 [jit]
rabbitmq-cluster-docker-master-rabbitmq1-1  |   TLS Library: OpenSSL - OpenSSL 3.1.3 19 Sep 2023
rabbitmq-cluster-docker-master-rabbitmq1-1  |   Release series support status: supported
rabbitmq-cluster-docker-master-rabbitmq1-1  | 
rabbitmq-cluster-docker-master-rabbitmq1-1  |   Doc guides:  https://rabbitmq.com/documentation.html
rabbitmq-cluster-docker-master-rabbitmq1-1  |   Support:     https://rabbitmq.com/contact.html
rabbitmq-cluster-docker-master-rabbitmq1-1  |   Tutorials:   https://rabbitmq.com/getstarted.html
rabbitmq-cluster-docker-master-rabbitmq1-1  |   Monitoring:  https://rabbitmq.com/monitoring.html
rabbitmq-cluster-docker-master-rabbitmq1-1  | 
rabbitmq-cluster-docker-master-rabbitmq1-1  |   Logs: <stdout>
rabbitmq-cluster-docker-master-rabbitmq1-1  | 
rabbitmq-cluster-docker-master-rabbitmq1-1  |   Config file(s): /etc/rabbitmq/conf.d/10-defaults.conf
rabbitmq-cluster-docker-master-rabbitmq1-1  | 
rabbitmq-cluster-docker-master-rabbitmq1-1  |   Starting broker...2024-02-08 06:09:00.996540+00:00 [info] <0.230.0> 
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.996540+00:00 [info] <0.230.0>  node           : rabbit@rabbitmq1
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.996540+00:00 [info] <0.230.0>  home dir       : /var/lib/rabbitmq
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.996540+00:00 [info] <0.230.0>  config file(s) : /etc/rabbitmq/conf.d/10-defaults.conf
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.996540+00:00 [info] <0.230.0>  cookie hash    : gnzLDuqKcGxMNKFokfhOew==
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.996540+00:00 [info] <0.230.0>  log(s)         : <stdout>
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:00.996540+00:00 [info] <0.230.0>  data dir       : /var/lib/rabbitmq/mnesia/rabbit@rabbitmq1
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.874289+00:00 [info] <0.230.0> Running boot step pre_boot defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.874379+00:00 [info] <0.230.0> Running boot step rabbit_global_counters defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.879290+00:00 [info] <0.230.0> Running boot step rabbit_osiris_metrics defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.879486+00:00 [info] <0.230.0> Running boot step rabbit_core_metrics defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.880335+00:00 [info] <0.230.0> Running boot step rabbit_alarm defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.895200+00:00 [info] <0.445.0> Memory high watermark set to 381 MiB (400479027 bytes) of 954 MiB (1001197568 bytes) total
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.909693+00:00 [info] <0.447.0> Enabling free disk space monitoring (disk free space: 5080203264, total memory: 1001197568)
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.909913+00:00 [info] <0.447.0> Disk free limit set to 50MB
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.916397+00:00 [info] <0.230.0> Running boot step code_server_cache defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.916597+00:00 [info] <0.230.0> Running boot step file_handle_cache defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.916899+00:00 [info] <0.450.0> Limiting to approx 1048479 file handles (943629 sockets)
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.917054+00:00 [info] <0.451.0> FHC read buffering: OFF
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.917133+00:00 [info] <0.451.0> FHC write buffering: ON
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.919952+00:00 [info] <0.230.0> Running boot step worker_pool defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.920130+00:00 [info] <0.332.0> Will use 2 processes for default worker pool
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.920211+00:00 [info] <0.332.0> Starting worker pool 'worker_pool' with 2 processes in it
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.920666+00:00 [info] <0.230.0> Running boot step database defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.922783+00:00 [info] <0.230.0> Node database directory at /var/lib/rabbitmq/mnesia/rabbit@rabbitmq1 is empty. Assuming we need to join an existing cluster or initialise from scratch...
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.922884+00:00 [info] <0.230.0> Configured peer discovery backend: rabbit_peer_discovery_classic_config
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.923557+00:00 [info] <0.230.0> Will try to lock with peer discovery backend rabbit_peer_discovery_classic_config
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.926294+00:00 [info] <0.230.0> All discovered existing cluster peers:
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.926467+00:00 [info] <0.230.0> Discovered no peer nodes to cluster with. Some discovery backends can filter nodes out based on a readiness criteria. Enabling debug logging might help troubleshoot.
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:10.932280+00:00 [notice] <0.44.0> Application mnesia exited with reason: stopped
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.318758+00:00 [info] <0.230.0> Waiting for Mnesia tables for 30000 ms, 9 retries left
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.319005+00:00 [info] <0.230.0> Successfully synced tables from a peer
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.320737+00:00 [notice] <0.333.0> Feature flags: attempt to enable `stream_sac_coordinator_unblock_group`...
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.410847+00:00 [notice] <0.333.0> Feature flags: `stream_sac_coordinator_unblock_group` enabled
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.411052+00:00 [notice] <0.333.0> Feature flags: attempt to enable `restart_streams`...
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.499424+00:00 [notice] <0.333.0> Feature flags: `restart_streams` enabled
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.499874+00:00 [info] <0.230.0> Waiting for Mnesia tables for 30000 ms, 9 retries left
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.500317+00:00 [info] <0.230.0> Successfully synced tables from a peer
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.521041+00:00 [info] <0.230.0> Waiting for Mnesia tables for 30000 ms, 9 retries left
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.522260+00:00 [info] <0.230.0> Successfully synced tables from a peer
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.522365+00:00 [info] <0.230.0> Peer discovery backend rabbit_peer_discovery_classic_config does not support registration, skipping registration.
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.522546+00:00 [info] <0.230.0> Will try to unlock with peer discovery backend rabbit_peer_discovery_classic_config
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.523549+00:00 [info] <0.230.0> Running boot step tracking_metadata_store defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.523696+00:00 [info] <0.632.0> Setting up a table for connection tracking on this node: tracked_connection
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.523812+00:00 [info] <0.632.0> Setting up a table for per-vhost connection counting on this node: tracked_connection_per_vhost
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.523957+00:00 [info] <0.632.0> Setting up a table for per-user connection counting on this node: tracked_connection_per_user
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.524064+00:00 [info] <0.632.0> Setting up a table for channel tracking on this node: tracked_channel
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.524174+00:00 [info] <0.632.0> Setting up a table for channel tracking on this node: tracked_channel_per_user
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.524305+00:00 [info] <0.230.0> Running boot step networking_metadata_store defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.524452+00:00 [info] <0.230.0> Running boot step feature_flags defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.525341+00:00 [info] <0.230.0> Running boot step codec_correctness_check defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.525433+00:00 [info] <0.230.0> Running boot step external_infrastructure defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.525567+00:00 [info] <0.230.0> Running boot step rabbit_event defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.525766+00:00 [info] <0.230.0> Running boot step rabbit_registry defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.525877+00:00 [info] <0.230.0> Running boot step rabbit_auth_mechanism_amqplain defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.526003+00:00 [info] <0.230.0> Running boot step rabbit_auth_mechanism_cr_demo defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.529370+00:00 [info] <0.230.0> Running boot step rabbit_auth_mechanism_plain defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.529543+00:00 [info] <0.230.0> Running boot step rabbit_exchange_type_direct defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.529634+00:00 [info] <0.230.0> Running boot step rabbit_exchange_type_fanout defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.529733+00:00 [info] <0.230.0> Running boot step rabbit_exchange_type_headers defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.537803+00:00 [info] <0.230.0> Running boot step rabbit_exchange_type_topic defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.537989+00:00 [info] <0.230.0> Running boot step rabbit_mirror_queue_mode_all defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.538082+00:00 [info] <0.230.0> Running boot step rabbit_mirror_queue_mode_exactly defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.538200+00:00 [info] <0.230.0> Running boot step rabbit_mirror_queue_mode_nodes defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.538297+00:00 [info] <0.230.0> Running boot step rabbit_priority_queue defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.538371+00:00 [info] <0.230.0> Priority queues enabled, real BQ is rabbit_variable_queue
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.538509+00:00 [info] <0.230.0> Running boot step rabbit_queue_location_client_local defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.542715+00:00 [info] <0.230.0> Running boot step rabbit_queue_location_min_masters defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.542923+00:00 [info] <0.230.0> Running boot step rabbit_queue_location_random defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.543038+00:00 [info] <0.230.0> Running boot step kernel_ready defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.543097+00:00 [info] <0.230.0> Running boot step rabbit_sysmon_minder defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.543310+00:00 [info] <0.230.0> Running boot step rabbit_epmd_monitor defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.552599+00:00 [info] <0.640.0> epmd monitor knows us, inter-node communication (distribution) port: 25672
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.553152+00:00 [info] <0.230.0> Running boot step guid_generator defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.619284+00:00 [info] <0.230.0> Running boot step rabbit_node_monitor defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.620797+00:00 [info] <0.645.0> Starting rabbit_node_monitor
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.621883+00:00 [info] <0.230.0> Running boot step delegate_sup defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.624976+00:00 [info] <0.230.0> Running boot step rabbit_memory_monitor defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.625360+00:00 [info] <0.230.0> Running boot step rabbit_fifo_dlx_sup defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.625481+00:00 [info] <0.230.0> Running boot step core_initialized defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.625549+00:00 [info] <0.230.0> Running boot step rabbit_channel_tracking_handler defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.625620+00:00 [info] <0.230.0> Running boot step rabbit_connection_tracking_handler defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.625704+00:00 [info] <0.230.0> Running boot step rabbit_definitions_hashing defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.625812+00:00 [info] <0.230.0> Running boot step rabbit_exchange_parameters defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.626815+00:00 [info] <0.230.0> Running boot step rabbit_mirror_queue_misc defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.627900+00:00 [info] <0.230.0> Running boot step rabbit_policies defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.628401+00:00 [info] <0.230.0> Running boot step rabbit_policy defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.628563+00:00 [info] <0.230.0> Running boot step rabbit_queue_location_validator defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.628658+00:00 [info] <0.230.0> Running boot step rabbit_quorum_memory_manager defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.629038+00:00 [info] <0.230.0> Running boot step rabbit_stream_coordinator defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.633668+00:00 [info] <0.230.0> Running boot step rabbit_vhost_limit defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.633863+00:00 [info] <0.230.0> Running boot step rabbit_mgmt_reset_handler defined by app rabbitmq_management
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.633954+00:00 [info] <0.230.0> Running boot step rabbit_mgmt_db_handler defined by app rabbitmq_management_agent
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.634203+00:00 [info] <0.230.0> Management plugin: using rates mode 'basic'
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.636328+00:00 [info] <0.230.0> Running boot step recovery defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.637560+00:00 [info] <0.230.0> Running boot step empty_db_check defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.637636+00:00 [info] <0.230.0> Will seed default virtual host and user...
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.638743+00:00 [info] <0.230.0> Adding vhost '/' (description: 'Default virtual host', tags: [])
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.648738+00:00 [info] <0.230.0> Inserted a virtual host record {vhost,<<"/">>,[],
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.648738+00:00 [info] <0.230.0>                                       #{description =>
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.648738+00:00 [info] <0.230.0>                                             <<"Default virtual host">>,
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.648738+00:00 [info] <0.230.0>                                         tags => []}}
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.670505+00:00 [info] <0.691.0> Making sure data directory '/var/lib/rabbitmq/mnesia/rabbit@rabbitmq1/msg_stores/vhosts/628WB79CIFDYO9LJI6DKMI09L' for vhost '/' exists
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.675669+00:00 [info] <0.691.0> Setting segment_entry_count for vhost '/' with 0 queues to '2048'
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.738778+00:00 [info] <0.691.0> Starting message stores for vhost '/'
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.739841+00:00 [info] <0.701.0> Message store "628WB79CIFDYO9LJI6DKMI09L/msg_store_transient": using rabbit_msg_store_ets_index to provide index
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.742554+00:00 [info] <0.691.0> Started message store of type transient for vhost '/'
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.743439+00:00 [info] <0.705.0> Message store "628WB79CIFDYO9LJI6DKMI09L/msg_store_persistent": using rabbit_msg_store_ets_index to provide index
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.747007+00:00 [warning] <0.705.0> Message store "628WB79CIFDYO9LJI6DKMI09L/msg_store_persistent": rebuilding indices from scratch
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.750257+00:00 [info] <0.691.0> Started message store of type persistent for vhost '/'
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.750575+00:00 [info] <0.691.0> Recovering 0 queues of type rabbit_classic_queue took 17ms
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.750751+00:00 [info] <0.691.0> Recovering 0 queues of type rabbit_quorum_queue took 0ms
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.751594+00:00 [info] <0.691.0> Recovering 0 queues of type rabbit_stream_queue took 0ms
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.758950+00:00 [info] <0.230.0> Created user 'guest'
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.762465+00:00 [info] <0.230.0> Successfully set user tags for user 'guest' to [administrator]
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.765816+00:00 [info] <0.230.0> Successfully set permissions for user 'guest' in virtual host '/' to '.*', '.*', '.*'
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.766012+00:00 [info] <0.230.0> Running boot step rabbit_observer_cli defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.766490+00:00 [info] <0.230.0> Running boot step rabbit_looking_glass defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.766697+00:00 [info] <0.230.0> Running boot step rabbit_core_metrics_gc defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.766954+00:00 [info] <0.230.0> Running boot step background_gc defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.767868+00:00 [info] <0.230.0> Running boot step routing_ready defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.767982+00:00 [info] <0.230.0> Running boot step pre_flight defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.768275+00:00 [info] <0.230.0> Running boot step notify_cluster defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.768700+00:00 [info] <0.230.0> Running boot step networking defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.772929+00:00 [info] <0.230.0> Running boot step definition_import_worker_pool defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.773637+00:00 [info] <0.332.0> Starting worker pool 'definition_import_pool' with 2 processes in it
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.776104+00:00 [info] <0.230.0> Running boot step cluster_name defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.776235+00:00 [info] <0.230.0> Initialising internal cluster ID to 'rabbitmq-cluster-id-hd0PAAb6Gr4G0T00Ei-TAA'
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.779536+00:00 [info] <0.230.0> Running boot step direct_client defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.779732+00:00 [info] <0.230.0> Running boot step rabbit_maintenance_mode_state defined by app rabbit
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.779913+00:00 [info] <0.230.0> Creating table rabbit_node_maintenance_states for maintenance mode status
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.795329+00:00 [info] <0.230.0> Running boot step rabbit_management_load_definitions defined by app rabbitmq_management
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.798725+00:00 [info] <0.744.0> Resetting node maintenance status
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.843934+00:00 [info] <0.803.0> Management plugin: HTTP (non-TLS) listener started on port 15672
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.844142+00:00 [info] <0.831.0> Statistics database started.
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.844311+00:00 [info] <0.830.0> Starting worker pool 'management_worker_pool' with 3 processes in it
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.870875+00:00 [info] <0.846.0> Prometheus metrics: HTTP (non-TLS) listener started on port 15692
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.871448+00:00 [info] <0.744.0> Ready to start client connection listeners
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:11.876175+00:00 [info] <0.890.0> started TCP listener on [::]:5672
rabbitmq-cluster-docker-master-rabbitmq1-1  |  completed with 4 plugins.
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:12.157654+00:00 [info] <0.744.0> Server startup complete; 4 plugins started.
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:12.157654+00:00 [info] <0.744.0>  * rabbitmq_prometheus
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:12.157654+00:00 [info] <0.744.0>  * rabbitmq_management
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:12.157654+00:00 [info] <0.744.0>  * rabbitmq_management_agent
rabbitmq-cluster-docker-master-rabbitmq1-1  | 2024-02-08 06:09:12.157654+00:00 [info] <0.744.0>  * rabbitmq_web_dispatch
rabbitmq-cluster-docker-master-rabbitmq2-1  | Applications 'rabbit_and_plugins' are running on node 'rabbit@rabbitmq2'
rabbitmq-cluster-docker-master-rabbitmq1-1  | Applications 'rabbit_and_plugins' are running on node 'rabbit@rabbitmq1'
rabbitmq-cluster-docker-master-rabbitmq3-1  | Applications 'rabbit_and_plugins' are running on node 'rabbit@rabbitmq3'
rabbitmq-cluster-docker-master-rabbitmq1-1  | tail: cannot open '/var/log/rabbitmq/*.log' for reading: No such file or directory
rabbitmq-cluster-docker-master-rabbitmq1-1  | tail: no files remaining
rabbitmq-cluster-docker-master-rabbitmq1-1 exited with code 1
rabbitmq-cluster-docker-master-rabbitmq3-1  | Stopping rabbit application on node rabbit@rabbitmq3 ...
rabbitmq-cluster-docker-master-rabbitmq2-1  | Stopping rabbit application on node rabbit@rabbitmq2 ...
rabbitmq-cluster-docker-master-rabbitmq3-1  | Clustering node rabbit@rabbitmq3 with rabbit@rabbitmq1
rabbitmq-cluster-docker-master-rabbitmq2-1  | Clustering node rabbit@rabbitmq2 with rabbit@rabbitmq1
rabbitmq-cluster-docker-master-rabbitmq3-1  | Error: unable to perform an operation on node 'rabbit@rabbitmq1'. Please see diagnostics information and suggestions below.
rabbitmq-cluster-docker-master-rabbitmq3-1  | 
rabbitmq-cluster-docker-master-rabbitmq3-1  | Most common reasons for this are:
rabbitmq-cluster-docker-master-rabbitmq3-1  | 
rabbitmq-cluster-docker-master-rabbitmq3-1  |  * Target node is unreachable (e.g. due to hostname resolution, TCP connection or firewall issues)
rabbitmq-cluster-docker-master-rabbitmq3-1  |  * CLI tool fails to authenticate with the server (e.g. due to CLI tool's Erlang cookie not matching that of the server)
rabbitmq-cluster-docker-master-rabbitmq3-1  |  * Target node is not running
rabbitmq-cluster-docker-master-rabbitmq3-1  | 
rabbitmq-cluster-docker-master-rabbitmq3-1  | In addition to the diagnostics info below:
rabbitmq-cluster-docker-master-rabbitmq3-1  | 
rabbitmq-cluster-docker-master-rabbitmq3-1  |  * See the CLI, clustering and networking guides on https://rabbitmq.com/documentation.html to learn more
rabbitmq-cluster-docker-master-rabbitmq3-1  |  * Consult server logs on node rabbit@rabbitmq1
rabbitmq-cluster-docker-master-rabbitmq3-1  |  * If target node is configured to use long node names, don't forget to use --longnames with CLI tools
rabbitmq-cluster-docker-master-rabbitmq3-1  | 
rabbitmq-cluster-docker-master-rabbitmq3-1  | DIAGNOSTICS
rabbitmq-cluster-docker-master-rabbitmq3-1  | ===========
rabbitmq-cluster-docker-master-rabbitmq3-1  | 
rabbitmq-cluster-docker-master-rabbitmq3-1  | attempted to contact: [rabbit@rabbitmq1]
rabbitmq-cluster-docker-master-rabbitmq3-1  | 
rabbitmq-cluster-docker-master-rabbitmq3-1  | rabbit@rabbitmq1:
rabbitmq-cluster-docker-master-rabbitmq3-1  |   * unable to connect to epmd (port 4369) on rabbitmq1: nxdomain (non-existing domain)
rabbitmq-cluster-docker-master-rabbitmq3-1  | 
rabbitmq-cluster-docker-master-rabbitmq3-1  | 
rabbitmq-cluster-docker-master-rabbitmq3-1  | Current node details:
rabbitmq-cluster-docker-master-rabbitmq3-1  |  * node name: 'rabbitmqcli-818-rabbit@rabbitmq3'
rabbitmq-cluster-docker-master-rabbitmq3-1  |  * effective user's home directory: /var/lib/rabbitmq
rabbitmq-cluster-docker-master-rabbitmq3-1  |  * Erlang cookie hash: gnzLDuqKcGxMNKFokfhOew==
rabbitmq-cluster-docker-master-rabbitmq3-1  | 
rabbitmq-cluster-docker-master-rabbitmq2-1  | Error: unable to perform an operation on node 'rabbit@rabbitmq1'. Please see diagnostics information and suggestions below.
rabbitmq-cluster-docker-master-rabbitmq2-1  | 
rabbitmq-cluster-docker-master-rabbitmq2-1  | Most common reasons for this are:
rabbitmq-cluster-docker-master-rabbitmq2-1  | 
rabbitmq-cluster-docker-master-rabbitmq2-1  |  * Target node is unreachable (e.g. due to hostname resolution, TCP connection or firewall issues)
rabbitmq-cluster-docker-master-rabbitmq2-1  |  * CLI tool fails to authenticate with the server (e.g. due to CLI tool's Erlang cookie not matching that of the server)
rabbitmq-cluster-docker-master-rabbitmq2-1  |  * Target node is not running
rabbitmq-cluster-docker-master-rabbitmq2-1  | 
rabbitmq-cluster-docker-master-rabbitmq2-1  | In addition to the diagnostics info below:
rabbitmq-cluster-docker-master-rabbitmq2-1  | 
rabbitmq-cluster-docker-master-rabbitmq2-1  |  * See the CLI, clustering and networking guides on https://rabbitmq.com/documentation.html to learn more
rabbitmq-cluster-docker-master-rabbitmq2-1  |  * Consult server logs on node rabbit@rabbitmq1
rabbitmq-cluster-docker-master-rabbitmq2-1  |  * If target node is configured to use long node names, don't forget to use --longnames with CLI tools
rabbitmq-cluster-docker-master-rabbitmq2-1  | 
rabbitmq-cluster-docker-master-rabbitmq2-1  | DIAGNOSTICS
rabbitmq-cluster-docker-master-rabbitmq2-1  | ===========
rabbitmq-cluster-docker-master-rabbitmq2-1  | 
rabbitmq-cluster-docker-master-rabbitmq2-1  | attempted to contact: [rabbit@rabbitmq1]
rabbitmq-cluster-docker-master-rabbitmq2-1  | 
rabbitmq-cluster-docker-master-rabbitmq2-1  | rabbit@rabbitmq1:
rabbitmq-cluster-docker-master-rabbitmq2-1  |   * unable to connect to epmd (port 4369) on rabbitmq1: nxdomain (non-existing domain)
rabbitmq-cluster-docker-master-rabbitmq2-1  | 
rabbitmq-cluster-docker-master-rabbitmq2-1  | 
rabbitmq-cluster-docker-master-rabbitmq2-1  | Current node details:
rabbitmq-cluster-docker-master-rabbitmq2-1  |  * node name: 'rabbitmqcli-913-rabbit@rabbitmq2'
rabbitmq-cluster-docker-master-rabbitmq2-1  |  * effective user's home directory: /var/lib/rabbitmq
rabbitmq-cluster-docker-master-rabbitmq2-1  |  * Erlang cookie hash: gnzLDuqKcGxMNKFokfhOew==
rabbitmq-cluster-docker-master-rabbitmq2-1  | 
rabbitmq-cluster-docker-master-rabbitmq3-1 exited with code 69
rabbitmq-cluster-docker-master-rabbitmq2-1 exited with code 69
```

</details>


`docker compose ps`  
```console
NAME                                       IMAGE         COMMAND                                                                SERVICE   CREATED          STATUS          PORTS
rabbitmq-cluster-docker-master-haproxy-1   haproxy:1.7   "docker-entrypoint.sh haproxy -f /usr/local/etc/haproxy/haproxy.cfg"   haproxy   16 minutes ago   Up 16 minutes   0.0.0.0:5672->5672/tcp, :::5672->5672/tcp, 0.0.0.0:15672->15672/tcp, :::15672->15672/tcp
```

##### We see that the cluster has not started and we see the error when RMQ cluster was starting
```console
tail: cannot open '/var/log/rabbitmq/*.log' for reading: No such file or directory
tail: no files remaining
```

##### We could fix the error by creating a missing file, but that doesn't make sense. The comment in `cluster-entrypoint.sh` clearly says that this code `tail -f /var/log/rabbitmq/*.log` is only needed to "Keep foreground process active".  
##### Let's add any additional code to keep the process running, so the container continues to exist and doesn't exit. For example, we will use an infinite `while` loop with `sleep` command.


#### 2. Add infinite loop into the cluster-entrypoint.sh
`vi cluster-entrypoint.sh`  
```bash
#!/bin/bash

set -e

# Change .erlang.cookie permission
chmod 400 /var/lib/rabbitmq/.erlang.cookie

# Get hostname from enviromant variable
HOSTNAME=`env hostname`
echo "Starting RabbitMQ Server For host: " $HOSTNAME

if [ -z "$JOIN_CLUSTER_HOST" ]; then
    /usr/local/bin/docker-entrypoint.sh rabbitmq-server &
    sleep 5
    rabbitmqctl wait /var/lib/rabbitmq/mnesia/rabbit\@$HOSTNAME.pid
else
    /usr/local/bin/docker-entrypoint.sh rabbitmq-server -detached
    sleep 5
    rabbitmqctl wait /var/lib/rabbitmq/mnesia/rabbit\@$HOSTNAME.pid
    rabbitmqctl stop_app
    rabbitmqctl join_cluster rabbit@$JOIN_CLUSTER_HOST
    rabbitmqctl start_app
fi

# Keep foreground process active ...
while true; do
  sleep 60
done
```
