# Scenario: "Venice": Am I in a container?

**Description:** Try and figure out if you are inside a container (like a Docker one for example) or inside a Virtual Machine (like in the other scenarios).  

**Test:** This scenario doesn't have a test (hence also no "Check My Solution" either).  


---

### Solution:
#### 1. Check the existence of the /proc/1/cgroup file
##### If it contains the string `docker`, then we are likely inside a container
`cat /proc/1/cgroup`  
```console
0::/init.scope
```


#### 2. Check `top` command for `dockerd` or `containerd` processes, witch indicate a container
##### If there are no kernel threads like [kthreadd] we are in a container too
`top`  
```console
top - 19:25:50 up 4 min,  0 users,  load average: 0.53, 0.28, 0.10
Tasks:   7 total,   1 running,   6 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :    455.4 total,     40.5 free,    112.1 used,    302.8 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.    323.8 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                           
      1 root      20   0  100784   9876   7780 S   0.0   2.1   0:02.73 systemd                           
     21 root      20   0   26724   7000   6264 S   0.0   1.5   0:00.09 systemd-journal                   
     47 message+  20   0    8268   4008   3532 S   0.0   0.9   0:00.04 dbus-daemon                       
     49 root      20   0   13404   5676   5024 S   0.0   1.2   0:00.07 systemd-logind                    
     71 root       0 -20 1303824  11520   8112 S   0.0   2.5   0:00.13 gotty                             
     87 root       0 -20    6056   3080   2588 S   0.0   0.7   0:00.00 bash                              
    100 root       0 -20    8896   3524   3036 R   0.0   0.8   0:00.00 top  
```


#### 3. Check `ps` command for `systemd` or `init` processes with PID=1, witch indicate a VM or host
##### Inside a container only the main application has PID=1
`ps aux`  
```console
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.8  2.1 100784  9876 ?        Ss   19:23   0:02 /sbin/init
root          21  0.0  1.5  26724  7000 ?        Ss   19:23   0:00 /lib/systemd/systemd-journald
message+      47  0.0  0.8   8268  4008 ?        Ss   19:23   0:00 /usr/bin/dbus-daemon --system --addres
root          49  0.0  1.2  13404  5676 ?        Ss   19:23   0:00 /lib/systemd/systemd-logind
root          71  0.0  2.4 1303824 11520 ?       S<sl 19:23   0:00 /usr/local/gotty --permit-write --reco
root          87  0.0  0.7   6056  3288 pts/0    S<s  19:24   0:00 bash
root         104  0.0  0.7   8652  3288 pts/0    R<+  19:28   0:00 ps aux
```


#### 4. Check `docker info` command
##### If it runs without errors and returns information about Docker, then we are in a Docker container
`docker info`  
```console
bash: docker: command not found
```


#### 5. Check `lscpu` command
##### If we see tags indicating virtualization, such as "vmx" for Intel VT-x or "svm" for AMD-V, then we are in a virtual machine
`lscpu`  
```console
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
<...>
Hypervisor vendor:               KVM
Virtualization type:             full
<...>
```


#### 6. Check `mount` command
##### In a container, the file system is usually mounted using "overlay" or "aufs". If we see the results, we are in a container
`mount | grep overlay`  
```console
overlay on / type overlay (rw,relatime,lowerdir=/var/lib/containers/storage/overlay/l/4UGO27476EXYY2UQSGBWL6EZC4:/var/lib/containers/storage/overlay/l/3JZQV4UY3FCO3W7SL3ERYENEZN:/var/lib/containers/storage/overlay/l/SCP4AZOKFN5HY4R5CQ5UVOYS7K:/var/lib/containers/storage/overlay/l/LPA46WOYFQ5ZHRJPEGNEEUCN36:/var/lib/containers/storage/overlay/l/WYTOBRCWJIZJALTLB3T5GBAQAB,upperdir=/var/lib/containers/storage/overlay/97d89e3f7cc7178dcfd77caeabf3350a092c5edad5ec553bf2ca84956755065f/diff,workdir=/var/lib/containers/storage/overlay/97d89e3f7cc7178dcfd77caeabf3350a092c5edad5ec553bf2ca84956755065f/work)
```

`mount | grep aufs`  
```console
<no output>
```


#### 7. Chech environment variables with `env` command
##### If we see Docker-related variables (such as DOCKER_HOST, DOCKER_CONTAINER, DOCKER_VERSION etc.) we are in a Docker container
`env | grep -E "(DOCKER_HOST|DOCKER_CONTAINER|DOCKER_VERSION)"`  
```console
<no output>
```


#### 8. Check network interfaces with `ip a` or `ifconfig` command
##### In a Docker container, some network interface is usually prefixed with `docker0`
`ifconfig`  
```console
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:c6:aa:e5:14  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens5: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 172.31.43.112  netmask 255.255.240.0  broadcast 172.31.47.255
        inet6 fe80::8e0:1aff:fe90:62a9  prefixlen 64  scopeid 0x20<link>
        ether 0a:e0:1a:90:62:a9  txqueuelen 1000  (Ethernet)
        RX packets 747  bytes 66231 (64.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 688  bytes 456131 (445.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 12  bytes 1020 (1020.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12  bytes 1020 (1020.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```


#### 9. Check `lsns` command to get a list of namespaces
##### If we see namespaces like `mht`, `pid`, `ipc`, `net`, then this is a Docker container. If we see other namespaces such as `dm`, `vz`, `lxc`, `veth`, this may indicate the presence of a virtual machine
`lsns`  
```console
        NS TYPE   NPROCS PID USER COMMAND
4026531834 time        7   1 root /sbin/init
4026531837 user        7   1 root /sbin/init
4026531992 net         7   1 root /sbin/init
4026532184 mnt         6   1 root /sbin/init
4026532185 uts         6   1 root /sbin/init
4026532186 ipc         7   1 root /sbin/init
4026532187 pid         7   1 root /sbin/init
4026532188 cgroup      7   1 root /sbin/init
4026532253 mnt         1  49 root /lib/systemd/systemd-logind
4026532254 uts         1  49 root /lib/systemd/systemd-logind
```


#### 10. AUTHOR'S SOLUTION
##### A way of checking is by looking at the environment of the PID=1 process and see if there's a container variable like `container=podman`. But its value is changed for fun
`cat /proc/1/environ|tr "\0" "\n"|grep container`  
```console
container=youlooked
```
