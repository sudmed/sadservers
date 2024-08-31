## "Gitega": Find the Bad Git Commit

**Description:** The directory at  `/home/admin/git` has a Git repository with a Golang program and a test for it. To execute the test, from this "git" directory run: `go test`. The last (current HEAD) commit fails the test. Suppose the first commit passed the test.  
Find the (long hash) commit that first broke the test and enter it in the `/home/admin/solution` file. For example: `echo 9e80a7eb1b09385e93ab4a76cb2c93beec48fd9f > /home/admin/solution`

**Test:** Doing `md5sum /home/admin/solution` returns `f7db1bb6b7bfcd66a4eb66782804b39d`. The "Check My Solution" button runs the script `/home/admin/agent/check.sh`, which you can see and execute.

---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`
```console
total 40
drwxr-xr-x 6 admin admin 4096 Jun  3 01:19 .
drwxr-xr-x 3 root  root  4096 Feb 17  2024 ..
drwx------ 3 admin admin 4096 Feb 17  2024 .ansible
-rw-r--r-- 1 admin admin  220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 admin admin 3526 Mar 27  2022 .bashrc
-rw-r--r-- 1 admin admin  807 Mar 27  2022 .profile
drwx------ 2 admin admin 4096 Feb 17  2024 .ssh
-rw-r--r-- 1 admin admin  374 Jun  3 01:19 README.txt
drwxr-xr-x 2 admin root  4096 Jun  3 01:19 agent
drwxr-xr-x 3 admin admin 4096 Jun  3 01:19 git
-rw-r--r-- 1 admin admin    0 Jun  3 01:19 solution
```

`cd ~/agent && cat check.sh`
```console
#!/usr/bin/bash
# DO NOT MODIFY THIS FILE ("Check My Solution" will fail)

res=$(md5sum /home/admin/solution |awk '{print $1}')
res=$(echo $res|tr -d '\r')

if [[ "$res" = "f7db1bb6b7bfcd66a4eb66782804b39d" ]]
then
  echo -n "OK"
else
  echo -n "NO"
```

`cat /home/admin/README.txt`
```text
# Git Bisect
## What and Why
Trivial example to learn and practice [git bisect](https://git-scm.com/docs/git-bisect)
## Problem Description
This repository contains a Golang program and a test for it. 
To execute the test, run `go test`. The last commit fails the test. Suppose the first commit passed the test.
Problem: find the commit that first broke the test.
```

`cat /var/log/messages`
```console
Aug 31 14:45:37 i-02c27891642c72a9d kernel: [   60.549251] device-mapper: uevent: version 1.0.3
Aug 31 14:45:37 i-02c27891642c72a9d kernel: [   60.553737] device-mapper: ioctl: 4.43.0-ioctl (2020-10-01) initialised: dm-devel@redhat.com
Aug 31 14:45:37 i-02c27891642c72a9d kernel: [   60.978510] audit: type=1400 audit(1725115537.902:10): apparmor="STATUS" operation="profile_load" profile="unconfined" name="docker-default" pid=691 comm="apparmor_parser"
Aug 31 14:45:38 i-02c27891642c72a9d kernel: [   61.114495] bridge: filtering via arp/ip/ip6tables is no longer available by default. Update your scripts to load br_netfilter if you need this.
Aug 31 14:45:38 i-02c27891642c72a9d kernel: [   61.130884] Bridge firewalling registered
Aug 31 14:45:38 i-02c27891642c72a9d kernel: [   61.301854] Initializing XFRM netlink socket
Aug 31 14:45:39 i-02c27891642c72a9d ec2: 
Aug 31 14:45:39 i-02c27891642c72a9d ec2: #############################################################
Aug 31 14:45:39 i-02c27891642c72a9d ec2: -----BEGIN SSH HOST KEY FINGERPRINTS-----
Aug 31 14:45:39 i-02c27891642c72a9d ec2: 1024 SHA256:+HK+hUgFIMweWQJ9FCCCOd3nz45OywVYoJX1rQziIrg root@i-02c27891642c72a9d (DSA)
Aug 31 14:45:39 i-02c27891642c72a9d ec2: 256 SHA256:0wN7RmW9oWaiGzR5mLc3M4LTTzE/0dO59AOD5pz+BIQ root@i-02c27891642c72a9d (ECDSA)
Aug 31 14:45:39 i-02c27891642c72a9d ec2: 256 SHA256:O5VXf5GSEPtm0F5cuRojJet+vfLt8HzrEWoSVPmM12s root@i-02c27891642c72a9d (ED25519)
Aug 31 14:45:39 i-02c27891642c72a9d ec2: 3072 SHA256:ytIE7XvODdgmkG+XPv6nrrYd+xeCqypcYy93bKOHom8 root@i-02c27891642c72a9d (RSA)
Aug 31 14:45:39 i-02c27891642c72a9d ec2: -----END SSH HOST KEY FINGERPRINTS-----
Aug 31 14:45:39 i-02c27891642c72a9d ec2: #############################################################
Aug 31 14:55:37 i-02c27891642c72a9d rsyslogd: [origin software="rsyslogd" swVersion="8.2102.0" x-pid="611" x-info="https://www.rsyslog.com"] rsyslogd was HUPed
```

`cat /var/log/user.log`
```console
Aug 31 14:45:39 i-02c27891642c72a9d ec2: 
Aug 31 14:45:39 i-02c27891642c72a9d ec2: #############################################################
Aug 31 14:45:39 i-02c27891642c72a9d ec2: -----BEGIN SSH HOST KEY FINGERPRINTS-----
Aug 31 14:45:39 i-02c27891642c72a9d ec2: 1024 SHA256:+HK+hUgFIMweWQJ9FCCCOd3nz45OywVYoJX1rQziIrg root@i-02c27891642c72a9d (DSA)
Aug 31 14:45:39 i-02c27891642c72a9d ec2: 256 SHA256:0wN7RmW9oWaiGzR5mLc3M4LTTzE/0dO59AOD5pz+BIQ root@i-02c27891642c72a9d (ECDSA)
Aug 31 14:45:39 i-02c27891642c72a9d ec2: 256 SHA256:O5VXf5GSEPtm0F5cuRojJet+vfLt8HzrEWoSVPmM12s root@i-02c27891642c72a9d (ED25519)
Aug 31 14:45:39 i-02c27891642c72a9d ec2: 3072 SHA256:ytIE7XvODdgmkG+XPv6nrrYd+xeCqypcYy93bKOHom8 root@i-02c27891642c72a9d (RSA)
Aug 31 14:45:39 i-02c27891642c72a9d ec2: -----END SSH HOST KEY FINGERPRINTS-----
Aug 31 14:45:39 i-02c27891642c72a9d ec2: #############################################################
```

#### 2. Check git
`cd /home/admin/git && git status`
```console
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

#### 3. Run test
`go test`
```console
--- FAIL: TestHandler (0.00s)
    main_test.go:22: handler returned unexpected body: got Hey! /
         want Hey: /
FAIL
exit status 1
FAIL    github.com/fduran/git_bisect    0.005s
```

#### 4. Check git log
`git log`
```console
commit f2e018e090afa14f0961bb221d6a6e1b506afb3e (HEAD -> main, origin/main, origin/HEAD)
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 21:15:29 2024 -0400

    README.md

commit 47995fcd96cd2c2ac6466d329110fbcf24d5f8e2
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 20:41:42 2024 -0400

    README

commit c21bcc402cf8e1344b81892ba539602964b4ad56
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 20:15:15 2024 -0400

    module name

commit 3657dadcc4a7278cb46e45fa549d4873555ad589
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 19:56:31 2024 -0400

    6th

commit 96086c854e4284bfd7d02b46accee1e21f7c93ca
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 19:46:40 2024 -0400

    5th

commit 2e44089778e44dcd9b97aa3baacdcff10311841b
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 19:46:07 2024 -0400

    4th

commit 51793999aa179ce76927e6461be31cfd1a89d8c9
:...skipping...
commit f2e018e090afa14f0961bb221d6a6e1b506afb3e (HEAD -> main, origin/main, origin/HEAD)
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 21:15:29 2024 -0400

    README.md

commit 47995fcd96cd2c2ac6466d329110fbcf24d5f8e2
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 20:41:42 2024 -0400

    README

commit c21bcc402cf8e1344b81892ba539602964b4ad56
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 20:15:15 2024 -0400

    module name

commit 3657dadcc4a7278cb46e45fa549d4873555ad589
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 19:56:31 2024 -0400

    6th

commit 96086c854e4284bfd7d02b46accee1e21f7c93ca
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 19:46:40 2024 -0400

    5th

commit 2e44089778e44dcd9b97aa3baacdcff10311841b
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 19:46:07 2024 -0400

    4th

commit 51793999aa179ce76927e6461be31cfd1a89d8c9
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 19:45:23 2024 -0400

    third

commit 641eadeeefcd0896ea30a3565aa55dc8523426b6
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 19:43:28 2024 -0400

    second

commit 9e80a7eb1b09385e93ab4a76cb2c93beec48fd9f
Author: fduran <duran.fernando@gmail.com>
Date:   Sun Jun 2 19:42:00 2024 -0400

    first
~
~
```

#### 5. Get only long hashes of the commits
`git log --format=%H`
```console
f2e018e090afa14f0961bb221d6a6e1b506afb3e
47995fcd96cd2c2ac6466d329110fbcf24d5f8e2
c21bcc402cf8e1344b81892ba539602964b4ad56
3657dadcc4a7278cb46e45fa549d4873555ad589
96086c854e4284bfd7d02b46accee1e21f7c93ca
2e44089778e44dcd9b97aa3baacdcff10311841b
51793999aa179ce76927e6461be31cfd1a89d8c9
641eadeeefcd0896ea30a3565aa55dc8523426b6
9e80a7eb1b09385e93ab4a76cb2c93beec48fd9f
```

#### 6. Reversed order of hashes needed
`git log --format=%H --reverse`
```console
9e80a7eb1b09385e93ab4a76cb2c93beec48fd9f
641eadeeefcd0896ea30a3565aa55dc8523426b6
51793999aa179ce76927e6461be31cfd1a89d8c9
2e44089778e44dcd9b97aa3baacdcff10311841b
96086c854e4284bfd7d02b46accee1e21f7c93ca
3657dadcc4a7278cb46e45fa549d4873555ad589
c21bcc402cf8e1344b81892ba539602964b4ad56
47995fcd96cd2c2ac6466d329110fbcf24d5f8e2
f2e018e090afa14f0961bb221d6a6e1b506afb3e
```

#### 7. Create simple script for testing all commits by `go test` with for-loop statement
```bash
for i in $(git log --format=%H --reverse); do
  git checkout $i &>/dev/null && go test || break
done
echo $i
```
```console
PASS
ok      myproject       0.003s
PASS
ok      myproject       0.003s
Request: /
PASS
ok      myproject       0.003s
Request: /
--- FAIL: TestHandler (0.00s)
    main_test.go:22: handler returned unexpected body: got Hey! /
         want Hey: /
FAIL
exit status 1
FAIL    myproject       0.003s
2e44089778e44dcd9b97aa3baacdcff10311841b
```

#### 8. Save finded commit's hashe to the solution file
`echo 2e44089778e44dcd9b97aa3baacdcff10311841b > /home/admin/solution`


#### 9. Validate the task
`md5sum /home/admin/solution`
```console
f7db1bb6b7bfcd66a4eb66782804b39d  /home/admin/solution
```

`bash /home/admin/agent/check.sh`
```console
OK
```


---

#### Clues
```text
1. An approach would be to do a binary search of the commit that broke the test: pick the commit close to the middle of the commit history from git log, run go test and if it passes, repeat with a commit in the middle of the latest (most recent) half of the commit history and if it fails, test with a commit in the middle of the earliest half fo the commit history. Repeat the process until finding the culprit commit. There's an automated way to do this in git
2. Use "git bisect" (next clue will give the full solution)
3. See the solution at git_bisect
```
