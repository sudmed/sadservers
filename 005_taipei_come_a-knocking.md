# "Taipei": Come a-knocking

**Description:** There is a web server on port `:80` protected with [Port Knocking](https://en.wikipedia.org/wiki/Port_knocking). Find the one "knock" needed (sending a SYN to a single port, not a sequence) so you can curl localhost.  

**Test:** Executing curl localhost returns a message with `md5sum fe474f8e1c29e9f412ed3b726369ab65`. (Note: the resulting md5sum includes the new line terminator: `echo $(curl localhost)`)  

---

### Solution:
#### 1. Reconnaissance on the server
`curl localhost`  
```console
curl: (7) Failed to connect to localhost port 80: Connection refused
```

`echo $(curl localhost)`  
```console
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
curl: (7) Failed to connect to localhost port 80: Connection refused
```

`ping  localhost`  
```console
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.044 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.039 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.053 ms
64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.064 ms
^C
--- localhost ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3073ms
rtt min/avg/max/mdev = 0.039/0.050/0.064/0.009 ms
```


#### 2. Let's knock on each port
`touch script.sh && chmod +x script.sh && vi script.sh`  
```bash
#!/bin/bash
for port in {1..65535}; do
  knock localhost $port
  echo "$port"
done
```
`./script.sh`  


#### 3. Send request to web server and get a message
`echo $(curl localhost)`  
```console
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    13  100    13    0     0   3250      0 --:--:-- --:--:-- --:--:--  3250
Who is there?
```


#### 4. Validate the task
`echo "Who is there?" | md5sum | awk '{print $1}'`  
```console
fe474f8e1c29e9f412ed3b726369ab65
```
