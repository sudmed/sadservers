# "Paris": Where is my webserver?

**Description:** A developer put an important password on his webserver `localhost:5000`. However, he can't find a way to recover it. This scenario is easy to once you realize the one "trick".  

Find the password and save it in `/home/admin/mysolution`, for example: `echo "somepassword" > ~/mysolution`.  

**Test:** `md5sum ~/mysolution` returns `d8bee9d7f830d5fb59b89e1e120cce8e`.  


---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 44
drwxr-xr-x 6 admin admin 4096 Sep 24 23:20 .
drwxr-xr-x 3 root  root  4096 Sep 17 16:44 ..
drwx------ 3 admin admin 4096 Sep 20 15:52 .ansible
-rw------- 1 admin admin   57 Sep 20 15:58 .bash_history
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin 4096 Sep 20 15:56 .config
-rw-r--r-- 1 admin admin  807 Aug  4  2021 .profile
drwx------ 2 admin admin 4096 Sep 17 16:44 .ssh
drwxr-xr-x 2 admin root  4096 Sep 24 23:20 agent
-rwxrwx--- 1 root  root   360 Sep 24 23:20 webserver.py
```

`cat agent/check.sh`  
```console
#!/bin/bash

expected_checksum="d8bee9d7f830d5fb59b89e1e120cce8e"
actual_checksum=$(md5sum /home/admin/mysolution | awk '{print $1}')

if [[ "$actual_checksum" == "$expected_checksum" ]]; then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`./webserver.py`  
```console
bash: ./webserver.py: Permission denied
```

`python3 webserver.py`  
```console
python3: can't open file '/home/admin/webserver.py': [Errno 13] Permission denied
```

`strace ./webserver.py`  
```console
execve("./webserver.py", ["./webserver.py"], 0x7ffdddc35ef0 /* 17 vars */) = -1 EACCES (Permission denied)
strace: exec: Permission denied
+++ exited with 1 +++
```

`sudo su`  
```console
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:
    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.
[sudo] password for admin:
```

`history`  
```console
    1  2023-09-20T15:57:57 > /home/admin/.bash_history
    2  2023-09-20T15:58:02 exit
    3  2024-01-31T07:47:01 ls -la
```

`groups`  
```console
admin adm dialout cdrom floppy sudo audio dip video plugdev netdev
```

`cat /var/log/user.log.1`  
```console
Sep 24 23:19:25 i-03be7122de01cf0bf ansible-ansible.legacy.setup: Invoked with gather_subset=['all'] gather_timeout=10 filter=[] fact_path=/etc/ansible/facts.d
Sep 24 23:19:26 i-03be7122de01cf0bf ansible-ansible.legacy.command: Invoked with _raw_params=echo 'paris' > /etc/sadscenario _uses_shell=True stdin_add_newline=True strip_empty_ends=True argv=None chdir=None executable=None creates=None removes=None stdin=None
Sep 24 23:19:28 i-03be7122de01cf0bf ansible-apt: Invoked with update_cache=True state=present update_cache_retries=5 update_cache_retry_max_delay=12 cache_valid_time=0 purge=False force=False upgrade=no dpkg_options=force-confdef,force-confold autoremove=False autoclean=False fail_on_autoremove=False only_upgrade=False force_apt_get=False clean=False allow_unauthenticated=False allow_downgrade=False allow_change_held_packages=False lock_timeout=60 package=None deb=None default_release=None install_recommends=None policy_rc_d=None
Sep 24 23:19:33 i-03be7122de01cf0bf ansible-apt: Invoked with name=['lsof', 'strace', 'netcat', 'telnet', 'wget', 'curl'] state=present package=['lsof', 'strace', 'netcat', 'telnet', 'wget', 'curl'] update_cache_retries=5 update_cache_retry_max_delay=12 cache_valid_time=0 purge=False force=False upgrade=no dpkg_options=force-confdef,force-confold autoremove=False autoclean=False fail_on_autoremove=False only_upgrade=False force_apt_get=False clean=False allow_unauthenticated=False allow_downgrade=False allow_change_held_packages=False lock_timeout=60 update_cache=None deb=None default_release=None install_recommends=None policy_rc_d=None
Sep 24 23:19:37 i-03be7122de01cf0bf ansible-apt: Invoked with name=python3-pip state=present package=['python3-pip'] update_cache_retries=5 update_cache_retry_max_delay=12 cache_valid_time=0 purge=False force=False upgrade=no dpkg_options=force-confdef,force-confold autoremove=False autoclean=False fail_on_autoremove=False only_upgrade=False force_apt_get=False clean=False allow_unauthenticated=False allow_downgrade=False allow_change_held_packages=False lock_timeout=60 update_cache=None deb=None default_release=None install_recommends=None policy_rc_d=None
Sep 24 23:20:21 i-03be7122de01cf0bf ansible-pip: Invoked with name=['flask'] executable=pip3 state=present virtualenv_site_packages=False virtualenv_command=virtualenv editable=False version=None requirements=None virtualenv=None virtualenv_python=None extra_args=None chdir=None umask=None
Sep 24 23:20:26 i-03be7122de01cf0bf ansible-ansible.legacy.stat: Invoked with path=/home/admin/webserver.py follow=False get_checksum=True checksum_algorithm=sha1 get_md5=False get_mime=True get_attributes=True
Sep 24 23:20:27 i-03be7122de01cf0bf ansible-ansible.legacy.copy: Invoked with src=/home/admin/.ansible/tmp/ansible-tmp-1695597623.734664-20901-107145741058694/source dest=/home/admin/webserver.py mode=0770 _original_basename=webserver.py follow=False checksum=af56df03320edfbf97c0cee77d3a3ba3bbeeab98 backup=False force=True unsafe_writes=False content=NOT_LOGGING_PARAMETER validate=None directory_mode=None remote_src=None local_follow=None owner=None group=None seuser=None serole=None selevel=None setype=None attributes=None
Sep 24 23:20:28 i-03be7122de01cf0bf ansible-ansible.legacy.stat: Invoked with path=/etc/systemd/system/flaskapp.service follow=False get_checksum=True checksum_algorithm=sha1 get_md5=False get_mime=True get_attributes=True
Sep 24 23:20:29 i-03be7122de01cf0bf ansible-ansible.legacy.copy: Invoked with src=/home/admin/.ansible/tmp/ansible-tmp-1695597626.543989-20918-170416001239157/source dest=/etc/systemd/system/flaskapp.service _original_basename=flaskapp.service follow=False checksum=7bcdb8d6f58aa69fce8c8e1356a0779436b238d6 backup=False force=True unsafe_writes=False content=NOT_LOGGING_PARAMETER validate=None directory_mode=None remote_src=None local_follow=None mode=None owner=None group=None seuser=None serole=None selevel=None setype=None attributes=None
Sep 24 23:20:31 i-03be7122de01cf0bf ansible-systemd: Invoked with name=flaskapp state=started enabled=True daemon_reload=False daemon_reexec=False scope=system no_block=False force=None masked=None
Sep 24 23:20:33 i-03be7122de01cf0bf ansible-ansible.legacy.stat: Invoked with path=/home/admin/agent/check.sh follow=False get_checksum=True checksum_algorithm=sha1 get_md5=False get_mime=True get_attributes=True
Sep 24 23:20:34 i-03be7122de01cf0bf ansible-ansible.legacy.copy: Invoked with src=/home/admin/.ansible/tmp/ansible-tmp-1695597630.983638-20944-112761724585294/source dest=/home/admin/agent/check.sh mode=+x owner=admin group=admin _original_basename=check.sh follow=False checksum=81e5ff46d75234826bc1ab2e1cdf34024cc798d8 backup=False force=True unsafe_writes=False content=NOT_LOGGING_PARAMETER validate=None directory_mode=None remote_src=None local_follow=None seuser=None serole=None selevel=None setype=None attributes=None
Sep 24 23:20:35 i-03be7122de01cf0bf ansible-ansible.builtin.file: Invoked with path=/var/log/cast state=absent recurse=False force=False follow=True modification_time_format=%Y%m%d%H%M.%S access_time_format=%Y%m%d%H%M.%S unsafe_writes=False _original_basename=None _diff_peek=None src=None modification_time=None access_time=None mode=None owner=None group=None seuser=None serole=None selevel=None setype=None attributes=None
Sep 24 23:20:36 i-03be7122de01cf0bf ansible-ansible.builtin.file: Invoked with path=/var/log/cast owner=admin mode=a+wx state=directory recurse=False force=False follow=True modification_time_format=%Y%m%d%H%M.%S access_time_format=%Y%m%d%H%M.%S unsafe_writes=False _original_basename=None _diff_peek=None src=None modification_time=None access_time=None group=None seuser=None serole=None selevel=None setype=None attributes=None
Sep 24 23:20:38 i-03be7122de01cf0bf ansible-replace: Invoked with path=/etc/cloud/cloud.cfg regexp=sudo: replace=#sudo: backup=False encoding=utf-8 unsafe_writes=False after=None before=None validate=None mode=None owner=None group=None seuser=None serole=None selevel=None setype=None attributes=None
Sep 24 23:20:39 i-03be7122de01cf0bf ansible-replace: Invoked with path=/etc/cloud/cloud.cfg.d/01_debian_cloud.cfg regexp=sudo: replace=#sudo: backup=False encoding=utf-8 unsafe_writes=False after=None before=None validate=None mode=None owner=None group=None seuser=None serole=None selevel=None setype=None attributes=None
Sep 24 23:20:40 i-03be7122de01cf0bf ansible-file: Invoked with path=/etc/sudoers.d/90-cloud-init-users state=absent recurse=False force=False follow=True modification_time_format=%Y%m%d%H%M.%S access_time_format=%Y%m%d%H%M.%S unsafe_writes=False _original_basename=None _diff_peek=None src=None modification_time=None access_time=None mode=None owner=None group=None seuser=None serole=None selevel=None setype=None attributes=None
```

`tail /var/log/syslog`  
```console
Jan 31 07:51:07 i-08d9171e79c13ea09 systemd[1]: Started Hammer Time.
Jan 31 07:51:09 i-08d9171e79c13ea09 systemd[1]: mc.service: Succeeded.
Jan 31 07:51:09 i-08d9171e79c13ea09 systemd[1]: mc.service: Consumed 1.088s CPU time.
Jan 31 07:52:01 i-08d9171e79c13ea09 dhclient[470]: XMT: Solicit on ens5, interval 114790ms.
Jan 31 07:52:05 i-08d9171e79c13ea09 systemd[1]: Started Hammer Time.
Jan 31 07:52:07 i-08d9171e79c13ea09 systemd[1]: mc.service: Succeeded.
Jan 31 07:52:07 i-08d9171e79c13ea09 systemd[1]: mc.service: Consumed 1.141s CPU time.
Jan 31 07:52:25 i-08d9171e79c13ea09 gotty[565]: 2024/01/31 07:52:25 127.0.0.1:42888 200 GET /
Jan 31 07:53:05 i-08d9171e79c13ea09 systemd[1]: Started Hammer Time.
Jan 31 07:53:07 i-08d9171e79c13ea09 systemd[1]: mc.service: Succeeded.
```

`iptables -L -v -n`  
```console
iptables v1.8.7 (nf_tables): Could not fetch rule set generation id: Permission denied (you must be root)
```

`curl localhost:5000`  
```console
Unauthorized
```

`telnet localhost 5000`  
```console
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
password
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
        "http://www.w3.org/TR/html4/strict.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
        <title>Error response</title>
    </head>
    <body>
        <h1>Error response</h1>
        <p>Error code: 400</p>
        <p>Message: Bad request syntax ('password').</p>
        <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
    </body>
</html>
Connection closed by foreign host.
```


#### 2. Telnet port 5000 with GET request
`telnet localhost 5000`  
```console
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
GET /

Welcome! Password is FDZPmh5AX3oiJt
Connection closed by foreign host.
```

#### 3. Curl with another user-agent
`curl -A "Mozilla" localhost:5000`  
```console
Welcome! Password is FDZPmh5AX3oiJt
```


#### 4. Netcat port 5000
`nc localhost 5000`  
```console
GET /

Welcome! Password is FDZPmh5AX3oiJt
```


#### 5. Validate the task
`echo "FDZPmh5AX3oiJt" > ~/mysolution`  
`md5sum ~/mysolution`  
```console
d8bee9d7f830d5fb59b89e1e120cce8e  /home/admin/mysolution
```

`./agent/check.sh`  
```console
OK
```

---

### AUTHOR'S SOLUTION
Use telnet or netcat: `nc localhost 5000` then `GET /` and Enter twice or using curl you can change the user agent: `curl --user-agent "whatever" localhost:5000`  
