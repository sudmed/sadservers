# Scenario: "Kihei": Surely Not Another Disk Space Scenario

**Description:** There is a `/home/admin/kihei` program. Make the changes necessary so it runs succesfully, without deleting the `/home/admin/datafile` file.  

**Test:** Running `/home/admin/kihei` returns `Done.`.  


---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 5245080
drwxr-xr-x 7 admin admin       4096 Jan 28 16:56 .
drwxr-xr-x 3 root  root        4096 Sep 17 16:44 ..
drwx------ 3 admin admin       4096 Sep 17 17:15 .ansible
-rw-r--r-- 1 admin admin        220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin       3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin       4096 Jan 28 16:56 .config
-rw-r--r-- 1 admin admin        807 Aug  4  2021 .profile
drwx------ 2 admin admin       4096 Sep 17 16:44 .ssh
drwxr-xr-x 2 admin root        4096 Sep 17 17:28 agent
drwxr-xr-x 2 admin root        4096 Sep 17 17:28 data
-rw-r--r-- 1 root  root  5368709120 Sep 17 17:28 datafile
-rwxr-xr-x 1 admin root     2207109 Sep 17 17:28 kihei
```

`./kihei`  
```console
panic: exit status 1

goroutine 1 [running]:
main.main()
        ./main.go:64 +0x47d
```

`# ./kihei `  
```console
Error: This program cannot be run as the 'root' superuser.
```

`./kihei -h`  
```console
Usage: ./kihei [options]
  -h    Display help
  -help
        Display help
  -v    Verbose mode (print extra info)
  -verbose
        Verbose mode (print extra info)
```

`./kihei -v`  
```console
Creating file /home/admin/data/newdatafile with size 1.5GB...
panic: exit status 1

goroutine 1 [running]:
main.main()
        ./main.go:64 +0x47d
```

`cat /home/admin/agent/check.sh`  
```console
#!/usr/bin/bash
# 6GB datafile exists
res=$(ls -l /home/admin/datafile |cut -d' ' -f 5)
res=$(echo $res|tr -d '\r')

if [[ "$res" != "5368709120" ]]
then
  echo -n "NO"
  exit
fi

# kihei binary didn't change
res=$(md5sum /home/admin/kihei |cut -d' ' -f 1)
res=$(echo $res|tr -d '\r')

if [[ "$res" != "79387f23f56e732aa789ee22761f8b84" ]]
then
  echo -n "NO"
  exit
fi

# kihei runs succesfully
res=$(/home/admin/kihei)
res=$(echo $res|tr -d '\r')

if [[ "$res" = "Done." ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

<details>

  <summary>strace /home/admin/kihei</summary>

```console
execve("/home/admin/kihei", ["/home/admin/kihei"], 0x7fffba8a3550 /* 13 vars */) = 0
arch_prctl(ARCH_SET_FS, 0x56b030)       = 0
sched_getaffinity(0, 8192, [0, 1])      = 8
openat(AT_FDCWD, "/sys/kernel/mm/transparent_hugepage/hpage_pmd_size", O_RDONLY) = 3
read(3, "2097152\n", 20)                = 8
close(3)                                = 0
mmap(NULL, 262144, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f76128ce000
mmap(NULL, 131072, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f76128ae000
mmap(NULL, 1048576, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f76127ae000
mmap(NULL, 8388608, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f7611fae000
mmap(NULL, 67108864, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f760dfae000
mmap(NULL, 536870912, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f75edfae000
mmap(0xc000000000, 67108864, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xc000000000
mmap(NULL, 33554432, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f75ebfae000
mmap(NULL, 2165776, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f75ebd9d000
mmap(0xc000000000, 4194304, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0xc000000000
mmap(0x7f76128ae000, 131072, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f76128ae000
mmap(0x7f761282e000, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f761282e000
mmap(0x7f76123b4000, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f76123b4000
mmap(0x7f760ffde000, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f760ffde000
mmap(0x7f75fe12e000, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f75fe12e000
mmap(NULL, 1048576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f75ebc9d000
mmap(NULL, 65536, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f75ebc8d000
mmap(NULL, 65536, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f75ebc7d000
rt_sigprocmask(SIG_SETMASK, NULL, [], 8) = 0
sigaltstack(NULL, {ss_sp=NULL, ss_flags=SS_DISABLE, ss_size=0}) = 0
sigaltstack({ss_sp=0xc000004000, ss_flags=0, ss_size=32768}, NULL) = 0
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
gettid()                                = 728
rt_sigaction(SIGHUP, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGHUP, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGINT, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGINT, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGQUIT, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGQUIT, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGILL, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGILL, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGTRAP, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGTRAP, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGABRT, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGABRT, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGBUS, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGBUS, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGFPE, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGFPE, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGUSR1, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGUSR1, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGSEGV, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGSEGV, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGUSR2, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGUSR2, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGPIPE, NULL, {sa_handler=SIG_IGN, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGPIPE, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGALRM, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGALRM, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGTERM, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGTERM, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGSTKFLT, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGSTKFLT, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGCHLD, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGCHLD, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGURG, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGURG, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGXCPU, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGXCPU, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGXFSZ, NULL, {sa_handler=SIG_IGN, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGXFSZ, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGVTALRM, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGVTALRM, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGPROF, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGPROF, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGWINCH, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGWINCH, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGIO, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGIO, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGPWR, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGPWR, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGSYS, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGSYS, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRTMIN, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_1, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_1, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_2, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_3, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_3, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_4, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_4, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_5, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_5, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_6, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_6, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_7, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_7, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_8, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_8, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_9, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_9, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_10, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_10, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_11, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_11, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_12, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_12, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_13, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_13, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_14, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_14, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_15, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_15, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_16, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_16, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_17, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_17, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_18, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_18, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_19, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_19, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_20, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_20, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_21, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_21, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_22, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_22, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_23, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_23, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_24, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_24, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_25, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_25, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_26, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_26, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_27, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_27, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_28, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_28, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_29, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_29, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_30, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_30, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_31, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_31, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigaction(SIGRT_32, NULL, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
rt_sigaction(SIGRT_32, {sa_handler=0x45faa0, sa_mask=~[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_RESTART|SA_SIGINFO, sa_restorer=0x45fbe0}, NULL, 8) = 0
rt_sigprocmask(SIG_SETMASK, ~[], [], 8) = 0
clone(child_stack=0xc000044000, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS, tls=0xc000034090) = 729
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
rt_sigprocmask(SIG_SETMASK, ~[], [], 8) = 0
clone(child_stack=0xc000046000, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS, tls=0xc000034490) = 730
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
rt_sigprocmask(SIG_SETMASK, ~[], [], 8) = 0
clone(child_stack=0xc000040000, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS, tls=0xc000034890) = 731
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
futex(0x56b0e8, FUTEX_WAIT_PRIVATE, 0, NULL) = 0
mmap(NULL, 262144, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f75ebc3d000
rt_sigprocmask(SIG_SETMASK, ~[], [], 8) = 0
clone(child_stack=0xc000092000, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS, tls=0xc000080090) = 732
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
futex(0x56b0e8, FUTEX_WAIT_PRIVATE, 0, NULL) = 0
fcntl(0, F_GETFL)                       = 0x2 (flags O_RDWR)
futex(0xc000034948, FUTEX_WAKE_PRIVATE, 1) = 1
fcntl(1, F_GETFL)                       = 0x2 (flags O_RDWR)
fcntl(2, F_GETFL)                       = 0x2 (flags O_RDWR)
getuid()                                = 1000
openat(AT_FDCWD, "/etc/passwd", O_RDONLY|O_CLOEXEC) = 3
epoll_create1(EPOLL_CLOEXEC)            = 4
pipe2([5, 6], O_NONBLOCK|O_CLOEXEC)     = 0
epoll_ctl(4, EPOLL_CTL_ADD, 5, {EPOLLIN, {u32=5871088, u64=5871088}}) = 0
epoll_ctl(4, EPOLL_CTL_ADD, 3, {EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, {u32=3955515064, u64=140144443416248}}) = -1 EPERM (Operation not permitted)
read(3, "root:x:0:0:root:/root:/bin/bash\n"..., 4096) = 1540
close(3)                                = 0
newfstatat(AT_FDCWD, "/home/admin/data/newdatafile", 0xc0000a4ac8, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/local/sbin/fallocate", 0xc0000a4d38, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/local/bin/fallocate", 0xc0000a4e08, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/sbin/fallocate", 0xc0000a4ed8, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/usr/bin/fallocate", {st_mode=S_IFREG|0755, st_size=35048, ...}, 0) = 0
openat(AT_FDCWD, "/dev/null", O_RDONLY|O_CLOEXEC) = 3
epoll_ctl(4, EPOLL_CTL_ADD, 3, {EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, {u32=3955515064, u64=140144443416248}}) = -1 EPERM (Operation not permitted)
openat(AT_FDCWD, "/dev/null", O_WRONLY|O_CLOEXEC) = 7
epoll_ctl(4, EPOLL_CTL_ADD, 7, {EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, {u32=3955515064, u64=140144443416248}}) = -1 EPERM (Operation not permitted)
openat(AT_FDCWD, "/dev/null", O_WRONLY|O_CLOEXEC) = 8
epoll_ctl(4, EPOLL_CTL_ADD, 8, {EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, {u32=3955515064, u64=140144443416248}}) = -1 EPERM (Operation not permitted)
pipe2([9, 10], O_CLOEXEC)               = 0
getpid()                                = 728
rt_sigprocmask(SIG_SETMASK, NULL, [], 8) = 0
rt_sigprocmask(SIG_SETMASK, ~[], NULL, 8) = 0
clone(child_stack=NULL, flags=CLONE_VM|CLONE_VFORK|SIGCHLD) = 733
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
close(10)                               = 0
read(9, "", 8)                          = 0
close(9)                                = 0
close(3)                                = 0
close(7)                                = 0
close(8)                                = 0
waitid(P_PID, 733, {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=733, si_uid=1000, si_status=1, si_utime=0, si_stime=0}, WEXITED|WNOWAIT, NULL) = 0
--- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=733, si_uid=1000, si_status=1, si_utime=0, si_stime=0} ---
rt_sigreturn({mask=[]})                 = 0
wait4(733, [{WIFEXITED(s) && WEXITSTATUS(s) == 1}], 0, {ru_utime={tv_sec=0, tv_usec=1946}, ru_stime={tv_sec=0, tv_usec=0}, ...}) = 733
unlinkat(AT_FDCWD, "/home/admin/data/newdatafile", 0) = 0
nanosleep({tv_sec=0, tv_nsec=1000000}, NULL) = 0
nanosleep({tv_sec=0, tv_nsec=1000000}, NULL) = 0
write(2, "panic: ", 7panic: )                  = 7
write(2, "exit status 1", 13exit status 1)           = 13
write(2, "\n", 1
)                       = 1
write(2, "\n", 1
)                       = 1
write(2, "goroutine ", 10goroutine )              = 10
write(2, "1", 11)                        = 1
write(2, " [", 2 [)                       = 2
write(2, "running", 7running)                  = 7
write(2, "]:\n", 3]:
)                     = 3
write(2, "main.main", 9main.main)                = 9
write(2, "(", 1()                        = 1
write(2, ")\n", 2)
)                      = 2
write(2, "\t", 1        )                       = 1
write(2, "./main.go", 9./main.go)                = 9
write(2, ":", 1:)                        = 1
write(2, "64", 264)                       = 2
write(2, " +", 2 +)                       = 2
write(2, "0x47d", 50x47d)                    = 5
write(2, "\n", 1
)                       = 1
exit_group(2)                           = ?
+++ exited with 2 +++
```

</details>


`cat /etc/fstab`  
```console
# /etc/fstab: static file system information
UUID=811e12d8-f542-4650-9330-8d96633bd90c / ext4 rw,discard,errors=remount-ro,x-systemd.growfs 0 1
UUID=8690-F844 /boot/efi vfat defaults 0 0
```

`df -h`  
```console
Filesystem       Size  Used Avail Use% Mounted on
udev             217M     0  217M   0% /dev
tmpfs             46M  368K   46M   1% /run
/dev/nvme0n1p1   7.7G  6.1G  1.2G  84% /
tmpfs            228M   12K  228M   1% /dev/shm
tmpfs            5.0M     0  5.0M   0% /run/lock
/dev/nvme0n1p15  124M  5.9M  118M   5% /boot/efi
```


`df -hi`  
```console
Filesystem      Inodes IUsed IFree IUse% Mounted on
udev               55K   307   54K    1% /dev
tmpfs              57K   441   57K    1% /run
/dev/nvme0n1p1    504K   33K  472K    7% /
tmpfs              57K     4   57K    1% /dev/shm
tmpfs              57K     3   57K    1% /run/lock
/dev/nvme0n1p15      0     0     0     - /boot/efi
```

`sudo lsblk`  
```console
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
nvme1n1      259:0    0    1G  0 disk 
nvme0n1      259:1    0    8G  0 disk 
├─nvme0n1p1  259:2    0  7.9G  0 part /
├─nvme0n1p14 259:3    0    3M  0 part 
└─nvme0n1p15 259:4    0  124M  0 part /boot/efi
nvme2n1      259:5    0    1G  0 disk
```

`sudo fdisk -l`  
```console
Disk /dev/nvme1n1: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: Amazon Elastic Block Store              
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes

Disk /dev/nvme0n1: 8 GiB, 8589934592 bytes, 16777216 sectors
Disk model: Amazon Elastic Block Store              
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 411E8940-1FEF-5347-B8D7-BE9578B62DC7

Device           Start      End  Sectors  Size Type
/dev/nvme0n1p1  262144 16777182 16515039  7.9G Linux filesystem
/dev/nvme0n1p14   2048     8191     6144    3M BIOS boot
/dev/nvme0n1p15   8192   262143   253952  124M EFI System
Partition table entries are not in disk order.

Disk /dev/nvme2n1: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: Amazon Elastic Block Store              
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```



#### 2. No free disk space
###### Creating a file `/home/admin/data/newdatafile` with size 1.5GB fails when there is only 1.2GB of free disk space. But there are two disks of 1 GB each (/dev/nvme1n1 and /dev/nvme2n1) and we can create a new logical disk from them.
`sudo pvcreate /dev/nvme1n1 /dev/nvme2n1`  
```console
  Physical volume "/dev/nvme1n1" successfully created.
  Physical volume "/dev/nvme2n1" successfully created.
```

`sudo vgcreate vg01 /dev/nvme1n1 /dev/nvme2n1`  
```console
  Volume group "vg01" successfully created
```

`sudo lvcreate -n lv01 -l 100%FREE vg01`  
```console
  Logical volume "lv01" created.
```

`sudo fdisk -l`  
```console
Disk /dev/nvme0n1: 8 GiB, 8589934592 bytes, 16777216 sectors
Disk model: Amazon Elastic Block Store              
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 411E8940-1FEF-5347-B8D7-BE9578B62DC7

Device           Start      End  Sectors  Size Type
/dev/nvme0n1p1  262144 16777182 16515039  7.9G Linux filesystem
/dev/nvme0n1p14   2048     8191     6144    3M BIOS boot
/dev/nvme0n1p15   8192   262143   253952  124M EFI System
Partition table entries are not in disk order.

Disk /dev/nvme2n1: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: Amazon Elastic Block Store              
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes

Disk /dev/nvme1n1: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: Amazon Elastic Block Store              
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes

Disk /dev/mapper/vg01-lv01: 1.99 GiB, 2139095040 bytes, 4177920 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

`sudo mkfs.ext4 /dev/vg01/lv01`  
```console
mke2fs 1.46.2 (28-Feb-2021)
Creating filesystem with 522240 4k blocks and 130560 inodes
Filesystem UUID: 4ed8fe6a-5daa-4155-ac54-5ea8c2ccc00e
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912
Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```

`sudo lsblk`  
```console
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
nvme0n1      259:0    0    8G  0 disk 
├─nvme0n1p1  259:2    0  7.9G  0 part /
├─nvme0n1p14 259:3    0    3M  0 part 
└─nvme0n1p15 259:4    0  124M  0 part /boot/efi
nvme2n1      259:1    0    1G  0 disk 
└─vg01-lv01  254:0    0    2G  0 lvm  
nvme1n1      259:5    0    1G  0 disk 
└─vg01-lv01  254:0    0    2G  0 lvm
```

`sudo pvscan`  
```console
  PV /dev/sdb   VG vg01            lvm2 [1020.00 MiB / 0    free]
  PV /dev/sdc   VG vg01            lvm2 [1020.00 MiB / 0    free]
  Total: 2 [1.99 GiB] / in use: 2 [1.99 GiB] / in no VG: 0 [0   ]
```

`sudo lvmdiskscan`  
```console
  /dev/xvda   [       8.00 GiB] 
  /dev/sdc    [       1.00 GiB] LVM physical volume
  /dev/xvda1  [       7.87 GiB] 
  /dev/xvda14 [       3.00 MiB] 
  /dev/xvda15 [     124.00 MiB] 
  /dev/sdb    [       1.00 GiB] LVM physical volume
  1 disk
  3 partitions
  2 LVM physical volume whole disks
  0 LVM physical volumes
```

`sudo pvdisplay`  
```console
  --- Physical volume ---
  PV Name               /dev/sdb
  VG Name               vg01
  PV Size               1.00 GiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              255
  Free PE               0
  Allocated PE          255
  PV UUID               ev54lO-76BS-BFMS-5zrd-3pkq-qWfa-jiBqYS
   
  --- Physical volume ---
  PV Name               /dev/sdc
  VG Name               vg01
  PV Size               1.00 GiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              255
  Free PE               0
  Allocated PE          255
  PV UUID               9Yx81A-EcLK-8cgu-Y9Gi-Mtss-ehAK-K0e5go
```

`sudo vgdisplay`  
```console
  --- Volume group ---
  VG Name               vg01
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               1.99 GiB
  PE Size               4.00 MiB
  Total PE              510
  Alloc PE / Size       510 / 1.99 GiB
  Free  PE / Size       0 / 0   
  VG UUID               PEsk7D-a6Q0-BcpR-k4Mf-RjUI-nJZE-j1x9mD
```


#### 3. Mount new logical disk and make owned by admin
`sudo mount /dev/vg01/lv01 /home/admin/data`  
`sudo chown -R admin: /home/admin/data`  



#### 4. Validate the task
`./kihei`  
```console
Done.
```

`cd /home/admin/agent/ && ./check.sh`  
```console
OK
```
