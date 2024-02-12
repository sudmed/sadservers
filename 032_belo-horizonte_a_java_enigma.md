# "Belo-Horizonte": A Java Enigma

**Description:** There is a one-class Java application in your `/home/admin` directory. Running the program will print out a secret code, or you may be able to extract the secret from the class file without executing it but I'm not providing any special tools for that.  

Put the secret code in a `/home/admin/solution` file, eg `echo "code" > /home/admin/solution`.  

**Test:** `md5sum /home/admin/solution |awk '{print $1}'` returns `9d2bd7aabb26681eacd9444da6b6643c`.  


---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 36
drwxr-xr-x 5 admin admin 4096 Mar  5  2023 .
drwxr-xr-x 3 root  root  4096 Feb 21  2023 ..
drwx------ 3 admin admin 4096 Mar  5  2023 .ansible
-rw-r--r-- 1 admin admin  220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 admin admin 3526 Mar 27  2022 .bashrc
-rw-r--r-- 1 admin admin  807 Mar 27  2022 .profile
drwx------ 2 admin admin 4096 Feb 21  2023 .ssh
-rw-r--r-- 1 admin admin  907 Mar  5  2023 Sad.class
drwxr-xr-x 2 admin admin 4096 Mar  5  2023 agent
```

`cat agent/check.sh`  
```bash
#!/usr/bin/bash
res=$(md5sum /home/admin/solution |awk '{print $1}')
res=$(echo $res|tr -d '\r')

if [[ "$res" = "9d2bd7aabb26681eacd9444da6b6643c" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`cat Sad.class`  
```console
=>

java/lang/Object<init>()V       VerySadjava/lang/Object



java/lang/String        substring(II)Ljava/lang/String;


replace(CC)Ljava/lang/String;



toUpperCase()Ljava/lang/String;
                                ava/lang/SystemoutLjava/io/PrintStream;
 !
  "#java/io/PrintStreamprintln(Ljava/lang/String;)V
'()
   *+java/lang/Threadsleep(J)V-java/lang/InterruptedException
,/
StackMapTable9[Ljava/lang/String;;[BmberTablemain([Ljava/lang/String;)V
SourceFile                        N-
:Sx::$&:.-36,42ad.java!13*4     56@<

%       -
         368=7 -8:



                  H,<=
```
##### If the class file is named Sad.class, the command to run would be: `java Sad`

`java Sad`  
```console
Error: LinkageError occurred while loading main class Sad
        java.lang.UnsupportedClassVersionError: Sad has been compiled by a more recent version of the Java Runtime (class file version 61.0), this version of the Java Runtime only recognizes class file versions up to 55.0
```

`java -version`  
```console
openjdk version "11.0.18" 2023-01-17
OpenJDK Runtime Environment (build 11.0.18+10-post-Debian-1deb11u1)
OpenJDK 64-Bit Server VM (build 11.0.18+10-post-Debian-1deb11u1, mixed mode, sharing)
```
##### Wrong error of Java Runtime, current version recognizes class file versions up to 55.0, but we need 61.0 or above. Try to download new version, first check which Linux is here

`uname -a`  
```console
Linux i-06f035d3f9f17bb2c 5.10.0-21-cloud-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64 GNU/Linux
```

`wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.deb`  
```console
--2024-02-12 06:52:50--  https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.deb
Resolving download.oracle.com (download.oracle.com)... 23.221.244.113
Connecting to download.oracle.com (download.oracle.com)|23.221.244.113|:443... ^C
```
##### No internet connection on the host. 
##### OK, maybe new version is just pre-installed

`which java`  
```console
/usr/bin/java
```

`whereis java`  
```console
java: /usr/bin/java /usr/share/java /usr/share/man/man1/java.1.gz
```

`whereis jvm`  
```console
jvm: /usr/lib/jvm
```

`ls -la /usr/lib/jvm`  
```console
total 32
drwxr-xr-x  6 root root 4096 Mar  5  2023 .
drwxr-xr-x 48 root root 4096 Mar  5  2023 ..
-rw-r--r--  1 root root 2047 Jan 27  2023 .java-1.11.0-openjdk-amd64.jinfo
-rw-r--r--  1 root root 1773 Jan 28  2023 .java-1.17.0-openjdk-amd64.jinfo
lrwxrwxrwx  1 root root   21 Jan 27  2023 java-1.11.0-openjdk-amd64 -> java-11-openjdk-amd64
lrwxrwxrwx  1 root root   21 Jan 28  2023 java-1.17.0-openjdk-amd64 -> java-17-openjdk-amd64
drwxr-xr-x  9 root root 4096 Mar  5  2023 java-11-openjdk-amd64
drwxr-xr-x  9 root root 4096 Mar  5  2023 java-17-openjdk-amd64
drwxr-xr-x  2 root root 4096 Mar  5  2023 openjdk-11
drwxr-xr-x  2 root root 4096 Mar  5  2023 openjdk-17
```
##### Here you are! Needed openjdk-17 was also installed. Let's run it

`/usr/lib/jvm/java-17-openjdk-amd64/bin/java Sad`  
```console
Error: Could not find or load main class Sad
Caused by: java.lang.NoClassDefFoundError: VerySad (wrong name: Sad)
```
##### We have an error: VerySad (wrong name: Sad). Let's rename the file and run it again

`mv Sad.class VerySad.class`  
`/usr/lib/jvm/java-17-openjdk-amd64/bin/java VerySad`  
```console
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
        at VerySad.main(VerySad.java:4)
```
##### We have new error now: OutOfMemoryError: Java heap space. Let's check RAM

`free -m`  
```console
               total        used        free      shared  buff/cache   available
Mem:             455          59         168           0         227         383
Swap:              0           0           0
```
##### We have 455 Mb total memory only and no swap at all. We cannot add RAM, but we can add swap file

`sudo fallocate -l 1G /swapfile`  
`sudo mkswap /swapfile`  
```console
mkswap: /swapfile: insecure permissions 0644, 0600 suggested.
Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
no label, UUID=bac734d2-42df-4964-be19-b61f86669cb4
```

`swapon --show`  
```console
NAME      TYPE  SIZE USED PRIO
/swapfile file 1024M   0B   -2
```

`free -m`  
```console
               total        used        free      shared  buff/cache   available
Mem:             455          60         161           0         234         383
Swap:           1023           0        1023
```
##### Now we can run java Let's run now

`/usr/lib/jvm/java-17-openjdk-amd64/bin/java -Xms1g -Xmx1g VerySad`  
```console
YXADJA
```


#### 2. Validate the task
`md5sum /home/admin/solution |awk '{print $1}'`  
```console
9d2bd7aabb26681eacd9444da6b6643c
```

`./agent/check.sh`  
```console
OK
```
