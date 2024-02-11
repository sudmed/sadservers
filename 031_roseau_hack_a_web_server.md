031_roseau_hack_a_web_server.md

# "Roseau": Hack a Web Server

**Description:** There is a secret stored in a file that the local Apache web server can provide. Find this secret and have it as a `/home/admin/secret.txt` file.  

Note that in this server the admin user is not a sudoer.  

Also note that the password crackers Hashcat and Hydra are installed from packages and John the Ripper binaries have been built from source in `/home/admin/john/run`.  

**Test:** `sha1sum /home/admin/secret.txt | awk '{print $1}'` returns `cc2c322fbcac56923048d083b465901aac0fe8f8`.  


---

### Solution:
#### 1. Reconnaissance on the server
`ls -la /home/admin/`  
```console
total 36
drwxr-xr-x  6 admin admin 4096 Feb 13  2023 .
drwxr-xr-x  3 root  root  4096 Aug 31  2022 ..
-rw-------  1 admin admin    0 Feb 20  2023 .bash_history
-rw-r--r--  1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r--  1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x  3 admin admin 4096 Feb 13  2023 .local
-rw-r--r--  1 admin admin  807 Aug  4  2021 .profile
drwx------  2 admin admin 4096 Aug 31  2022 .ssh
drwxr-xr-x  2 admin admin 4096 Feb 13  2023 agent
drwxr-xr-x 10 admin admin 4096 Feb 13  2023 john
```

`cat /home/admin/agent/check.sh`  
```bash
#!/usr/bin/bash
res=$(sha1sum /home/admin/secret.txt |awk '{print $1}')
res=$(echo $res|tr -d '\r')

if [[ "$res" = "cc2c322fbcac56923048d083b465901aac0fe8f8" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`ls -la /home/admin/john/`  
```console
total 140
drwxr-xr-x 10 admin admin  4096 Feb 13  2023 .
drwxr-xr-x  6 admin admin  4096 Feb 13  2023 ..
drwxr-xr-x  2 admin admin  4096 Feb 13  2023 .ci
drwxr-xr-x  2 admin admin  4096 Feb 13  2023 .circleci
-rw-r--r--  1 admin admin  2898 Feb 13  2023 .editorconfig
drwxr-xr-x  8 admin admin  4096 Feb 13  2023 .git
-rw-r--r--  1 admin admin   982 Feb 13  2023 .gitattributes
drwxr-xr-x  2 admin admin  4096 Feb 13  2023 .github
-rw-r--r--  1 admin admin  1473 Feb 13  2023 .gitignore
-rw-r--r--  1 admin admin  2975 Feb 13  2023 .mailmap
-rwxr-xr-x  1 admin admin  3838 Feb 13  2023 .pre-commit.sh
drwxr-xr-x  2 admin admin  4096 Feb 13  2023 .travis
-rw-r--r--  1 admin admin  1251 Feb 13  2023 .travis.yml
-rw-r--r--  1 admin admin  1654 Feb 13  2023 CONTRIBUTING.md
-rw-r--r--  1 admin admin  7864 Feb 13  2023 README.md
drwxr-xr-x  3 admin admin  4096 Feb 13  2023 doc
drwxr-xr-x  9 admin admin 12288 Feb 13  2023 run
drwxr-xr-x 13 admin admin 57344 Feb 13  2023 src
```

`ls -la /home/admin/john/run/`  
```console
total 88100
drwxr-xr-x  9 admin admin    12288 Feb 13  2023 .
drwxr-xr-x 10 admin admin     4096 Feb 13  2023 ..
-rwxr-xr-x  1 admin admin    11545 Feb 13  2023 1password2john.py
-rwxr-xr-x  1 admin admin    97726 Feb 13  2023 7z2john.pl
-rwxr-xr-x  1 admin admin    26130 Feb 13  2023 DPAPImk2john.py
-rwxr-xr-x  1 admin admin    91896 Feb 13  2023 SIPdump
-rwxr-xr-x  1 admin admin     3799 Feb 13  2023 adxcsouf2john.py
-rwxr-xr-x  1 admin admin     2729 Feb 13  2023 aem2john.py
-rwxr-xr-x  1 admin admin      894 Feb 13  2023 aix2john.pl
-rwxr-xr-x  1 admin admin     2300 Feb 13  2023 aix2john.py
-rw-r--r--  1 admin admin  4086722 Feb 13  2023 alnum.chr
-rw-r--r--  1 admin admin  4174257 Feb 13  2023 alnumspace.chr
-rw-r--r--  1 admin admin  1950539 Feb 13  2023 alpha.chr
-rwxr-xr-x  1 admin admin     1368 Feb 13  2023 andotp2john.py
-rwxr-xr-x  1 admin admin     3455 Feb 13  2023 androidbackup2john.py
-rwxr-xr-x  1 admin admin     7471 Feb 13  2023 androidfde2john.py
-rwxr-xr-x  1 admin admin     1758 Feb 13  2023 ansible2john.py
-rwxr-xr-x  1 admin admin      731 Feb 13  2023 apex2john.py
-rwxr-xr-x  1 admin admin     2141 Feb 13  2023 apop2john.py
-rwxr-xr-x  1 admin admin     2074 Feb 13  2023 applenotes2john.py
-rwxr-xr-x  1 admin admin     1447 Feb 13  2023 aruba2john.py
-rw-r--r--  1 admin admin  5720262 Feb 13  2023 ascii.chr
-rwxr-xr-x  1 admin admin      614 Feb 13  2023 atmail2john.pl
-rwxr-xr-x  1 admin admin     6299 Feb 13  2023 axcrypt2john.py
lrwxrwxrwx  1 admin admin        4 Feb 13  2023 base64conv -> john
-rwxr-xr-x  1 admin admin    10577 Feb 13  2023 benchmark-unify
-rwxr-xr-x  1 admin admin    10174 Feb 13  2023 bestcrypt2john.py
-rw-r--r--  1 admin admin     2314 Feb 13  2023 bestcryptve2john.py
drwxr-xr-x  2 admin admin     4096 Feb 13  2023 bip-0039
-rwxr-xr-x  1 admin admin     9421 Feb 13  2023 bitcoin2john.py
-rwxr-xr-x  1 admin admin   227128 Feb 13  2023 bitlocker2john
-rwxr-xr-x  1 admin admin     2719 Feb 13  2023 bitshares2john.py
-rwxr-xr-x  1 admin admin     4315 Feb 13  2023 bitwarden2john.py
-rwxr-xr-x  1 admin admin     7754 Feb 13  2023 bks2john.py
-rwxr-xr-x  1 admin admin     2940 Feb 13  2023 blockchain2john.py
-rwxr-xr-x  1 admin admin    25560 Feb 13  2023 calc_stat
-rwxr-xr-x  1 admin admin     2057 Feb 13  2023 cardano2john.py
-rwxr-xr-x  1 admin admin    26774 Feb 13  2023 ccache2john.py
-rwxr-xr-x  1 admin admin     7054 Feb 13  2023 cisco2john.pl
-rwxr-xr-x  1 admin admin      898 Feb 13  2023 codepage.pl
-rwxr-xr-x  1 admin admin   107488 Feb 13  2023 cprepair
-rwxr-xr-x  1 admin admin      880 Feb 13  2023 cracf2john.py
-rwxr-xr-x  1 admin admin     2570 Feb 13  2023 dashlane2john.py
-rwxr-xr-x  1 admin admin     3771 Feb 13  2023 deepsound2john.py
-rw-r--r--  1 admin admin     4099 Feb 13  2023 dictionary.rfc2865
-rw-r--r--  1 admin admin   465097 Feb 13  2023 digits.chr
-rwxr-xr-x  1 admin admin     7341 Feb 13  2023 diskcryptor2john.py
-rwxr-xr-x  1 admin admin    70856 Feb 13  2023 dmg2john
-rwxr-xr-x  1 admin admin     5249 Feb 13  2023 dmg2john.py
drwxr-xr-x  2 admin admin     4096 Feb 13  2023 dns
-rw-r--r--  1 admin admin    54925 Feb 13  2023 dumb16.conf
-rw-r--r--  1 admin admin   100781 Feb 13  2023 dumb32.conf
-rw-r--r--  1 admin admin    60346 Feb 13  2023 dynamic.conf
-rw-r--r--  1 admin admin     8539 Feb 13  2023 dynamic_disabled.conf
-rw-r--r--  1 admin admin     9542 Feb 13  2023 dynamic_flat_sse_formats.conf
-rwxr-xr-x  1 admin admin    40856 Feb 13  2023 eapmd5tojohn
-rwxr-xr-x  1 admin admin     2053 Feb 13  2023 ecryptfs2john.py
-rwxr-xr-x  1 admin admin     5083 Feb 13  2023 ejabberd2john.py
-rwxr-xr-x  1 admin admin     9558 Feb 13  2023 electrum2john.py
-rwxr-xr-x  1 admin admin     3633 Feb 13  2023 encdatavault2john.py
-rwxr-xr-x  1 admin admin     2717 Feb 13  2023 encfs2john.py
-rwxr-xr-x  1 admin admin     1107 Feb 13  2023 enpass2john.py
-rwxr-xr-x  1 admin admin     1113 Feb 13  2023 enpass5tojohn.py
-rwxr-xr-x  1 admin admin     3773 Feb 13  2023 ethereum2john.py
-rwxr-xr-x  1 admin admin     1725 Feb 13  2023 filezilla2john.py
-rw-r--r--  1 admin admin    99202 Feb 13  2023 fuzz.dic
-rwxr-xr-x  1 admin admin     3324 Feb 13  2023 fuzz_option.pl
-rwxr-xr-x  1 admin admin     3439 Feb 13  2023 geli2john.py
-rwxr-xr-x  1 admin admin     1868 Feb 13  2023 genincstats.rb
-rwxr-xr-x  1 admin admin   188088 Feb 13  2023 genmkvpwd
lrwxrwxrwx  1 admin admin        4 Feb 13  2023 gpg2john -> john
-rwxr-xr-x  1 admin admin    60864 Feb 13  2023 hccap2john
-rwxr-xr-x  1 admin admin     7096 Feb 13  2023 hccapx2john.py
-rwxr-xr-x  1 admin admin      428 Feb 13  2023 hextoraw.pl
-rwxr-xr-x  1 admin admin     1059 Feb 13  2023 htdigest2john.py
-rw-r--r--  1 admin admin     7536 Feb 13  2023 hybrid.conf
-rwxr-xr-x  1 admin admin     1458 Feb 13  2023 ibmiscanner2john.py
-rwxr-xr-x  1 admin admin      710 Feb 13  2023 ikescan2john.py
-rwxr-xr-x  1 admin admin     1626 Feb 13  2023 ios7tojohn.pl
-rwxr-xr-x  1 admin admin     5715 Feb 13  2023 itunes_backup2john.pl
-rwxr-xr-x  1 admin admin     4810 Feb 13  2023 iwork2john.py
-rwxr-xr-x  1 admin admin 23990808 Feb 13  2023 john
-rw-r--r--  1 admin admin    33024 Feb 13  2023 john.bash_completion
-rw-r--r--  1 admin admin   119670 Feb 13  2023 john.conf
-rw-r--r--  1 admin admin     8928 Feb 13  2023 john.zsh_completion
-rw-r--r--  1 admin admin    35525 Feb 13  2023 jtr_rulez.pm
-rw-r--r--  1 admin admin     3620 Feb 13  2023 jtrconf.pm
-rwxr-xr-x  1 admin admin     1110 Feb 13  2023 kdcdump2john.py
-rwxr-xr-x  1 admin admin   296472 Feb 13  2023 keepass2john
-rwxr-xr-x  1 admin admin     2955 Feb 13  2023 keychain2john.py
-rwxr-xr-x  1 admin admin     3534 Feb 13  2023 keyring2john.py
-rwxr-xr-x  1 admin admin     5416 Feb 13  2023 keystore2john.py
-rwxr-xr-x  1 admin admin     2644 Feb 13  2023 kirbi2john.py
-rwxr-xr-x  1 admin admin      849 Feb 13  2023 known_hosts2john.py
-rw-r--r--  1 admin admin    22635 Feb 13  2023 korelogic.conf
-rwxr-xr-x  1 admin admin     9677 Feb 13  2023 krb2john.py
-rwxr-xr-x  1 admin admin     4431 Feb 13  2023 kwallet2john.py
-rw-r--r--  1 admin admin  1466791 Feb 13  2023 lanman.chr
-rwxr-xr-x  1 admin admin     4297 Feb 13  2023 lastpass2john.py
-rw-r--r--  1 admin admin  7449800 Feb 13  2023 latin1.chr
-rwxr-xr-x  1 admin admin      472 Feb 13  2023 ldif2john.pl
-rwxr-xr-x  1 admin admin     2628 Feb 13  2023 leet.pl
drwxr-xr-x  2 admin admin     4096 Feb 13  2023 lib
-rwxr-xr-x  1 admin admin     6253 Feb 13  2023 libreoffice2john.py
-rwxr-xr-x  1 admin admin      878 Feb 13  2023 lion2john-alt.pl
-rwxr-xr-x  1 admin admin      994 Feb 13  2023 lion2john.pl
-rw-r--r--  1 admin admin  1184244 Feb 13  2023 lm_ascii.chr
-rwxr-xr-x  1 admin admin     1525 Feb 13  2023 lotus2john.py
-rw-r--r--  1 admin admin  1161863 Feb 13  2023 lower.chr
-rw-r--r--  1 admin admin  2464980 Feb 13  2023 lowernum.chr
-rw-r--r--  1 admin admin  1209621 Feb 13  2023 lowerspace.chr
-rwxr-xr-x  1 admin admin     4590 Feb 13  2023 luks2john.py
-rwxr-xr-x  1 admin admin     2602 Feb 13  2023 mac2john-alt.py
-rwxr-xr-x  1 admin admin    24611 Feb 13  2023 mac2john.py
-rwxr-xr-x  1 admin admin     1432 Feb 13  2023 mailer
-rwxr-xr-x  1 admin admin      842 Feb 13  2023 makechr
-rwxr-xr-x  1 admin admin     2297 Feb 13  2023 mcafee_epo2john.py
-rwxr-xr-x  1 admin admin    24848 Feb 13  2023 mkvcalcproba
-rwxr-xr-x  1 admin admin     1221 Feb 13  2023 monero2john.py
-rwxr-xr-x  1 admin admin     2495 Feb 13  2023 money2john.py
-rw-r--r--  1 admin admin     1055 Feb 13  2023 mongodb2john.js
-rwxr-xr-x  1 admin admin     6300 Feb 13  2023 mosquitto2john.py
-rwxr-xr-x  1 admin admin     2575 Feb 13  2023 mozilla2john.py
-rwxr-xr-x  1 admin admin     5440 Feb 13  2023 multibit2john.py
-rwxr-xr-x  1 admin admin      942 Feb 13  2023 neo2john.py
-rwxr-xr-x  1 admin admin     9676 Feb 13  2023 netntlm.pl
-rwxr-xr-x  1 admin admin     5186 Feb 13  2023 netscreen.py
-rwxr-xr-x  1 admin admin    11395 Feb 13  2023 network2john.lua
-rwxr-xr-x  1 admin admin   133905 Feb 13  2023 office2john.py
-rwxr-xr-x  1 admin admin     3072 Feb 13  2023 openbsd_softraid2john.py
drwxr-xr-x  4 admin admin     4096 Feb 13  2023 opencl
-rwxr-xr-x  1 admin admin     3911 Feb 13  2023 openssl2john.py
-rw-r--r--  1 admin admin  4051581 Feb 13  2023 oui.txt
-rwxr-xr-x  1 admin admin     2844 Feb 13  2023 padlock2john.py
-rwxr-xr-x  1 admin admin   203604 Feb 13  2023 pass_gen.pl
-rw-r--r--  1 admin admin 15327454 Feb 13  2023 password.lst
-rwxr-xr-x  1 admin admin    55349 Feb 13  2023 pcap2john.py
-rwxr-xr-x  1 admin admin    59808 Feb 13  2023 pdf2john.pl
-rwxr-xr-x  1 admin admin     4901 Feb 13  2023 pem2john.py
-rwxr-xr-x  1 admin admin     3329 Feb 13  2023 pfx2john.py
-rwxr-xr-x  1 admin admin     9861 Feb 13  2023 pgpdisk2john.py
-rwxr-xr-x  1 admin admin     2751 Feb 13  2023 pgpsda2john.py
-rwxr-xr-x  1 admin admin     9381 Feb 13  2023 pgpwde2john.py
-rw-r--r--  1 admin admin     6305 Feb 13  2023 pkcs12kdf.py
-rwxr-xr-x  1 admin admin     4198 Feb 13  2023 potcheck.pl
-rwxr-xr-x  1 admin admin     1462 Feb 13  2023 prosody2john.py
drwxr-xr-x  2 admin admin     4096 Feb 13  2023 protobuf
-rwxr-xr-x  1 admin admin     1438 Feb 13  2023 ps_token2john.py
-rwxr-xr-x  1 admin admin     2480 Feb 13  2023 pse2john.py
-rwxr-xr-x  1 admin admin    61128 Feb 13  2023 putty2john
-rwxr-xr-x  1 admin admin     1686 Feb 13  2023 pwsafe2john.py
-rwxr-xr-x  1 admin admin    43416 Feb 13  2023 racf2john
-rwxr-xr-x  1 admin admin     7165 Feb 13  2023 radius2john.pl
-rwxr-xr-x  1 admin admin     2274 Feb 13  2023 radius2john.py
lrwxrwxrwx  1 admin admin        4 Feb 13  2023 rar2john -> john
-rwxr-xr-x  1 admin admin    27776 Feb 13  2023 raw2dyna
-rw-r--r--  1 admin admin     5712 Feb 13  2023 regex_alphabets.conf
-rwxr-xr-x  1 admin admin     8143 Feb 13  2023 relbench
-rw-r--r--  1 admin admin    54525 Feb 13  2023 repeats16.conf
-rw-r--r--  1 admin admin   100475 Feb 13  2023 repeats32.conf
-rwxr-xr-x  1 admin admin     1606 Feb 13  2023 restic2john.py
-rwxr-xr-x  1 admin admin    16427 Feb 13  2023 rexgen2rules.pl
drwxr-xr-x  2 admin admin     4096 Feb 13  2023 rules
-rw-r--r--  1 admin admin   107926 Feb 13  2023 rules-by-rate.conf
-rw-r--r--  1 admin admin   107977 Feb 13  2023 rules-by-score.conf
-rwxr-xr-x  1 admin admin     2363 Feb 13  2023 rulestack.pl
-rwxr-xr-x  1 admin admin     9541 Feb 13  2023 sap2john.pl
-rwxr-xr-x  1 admin admin     1234 Feb 13  2023 sense2john.py
-rwxr-xr-x  1 admin admin      544 Feb 13  2023 sha-dump.pl
-rwxr-xr-x  1 admin admin      510 Feb 13  2023 sha-test.pl
-rwxr-xr-x  1 admin admin    20433 Feb 13  2023 signal2john.py
-rwxr-xr-x  1 admin admin      899 Feb 13  2023 sipdump2john.py
-rwxr-xr-x  1 admin admin     9677 Feb 13  2023 ssh2john.py
-rwxr-xr-x  1 admin admin    25114 Feb 13  2023 sspr2john.py
-rwxr-xr-x  1 admin admin     3814 Feb 13  2023 staroffice2john.py
-rw-r--r--  1 admin admin   107571 Feb 13  2023 stats
-rwxr-xr-x  1 admin admin      760 Feb 13  2023 strip2john.py
-rwxr-xr-x  1 admin admin    15572 Feb 13  2023 telegram2john.py
-rw-r--r--  1 admin admin    20245 Feb 13  2023 test_tezos2john.py
-rwxr-xr-x  1 admin admin    11581 Feb 13  2023 tezos2john.py
-rwxr-xr-x  1 admin admin    37384 Feb 13  2023 tgtsnarf
-rwxr-xr-x  1 admin admin     3238 Feb 13  2023 truecrypt2john.py
-rwxr-xr-x  1 admin admin    88496 Feb 13  2023 uaf2john
lrwxrwxrwx  1 admin admin        4 Feb 13  2023 unafs -> john
lrwxrwxrwx  1 admin admin        4 Feb 13  2023 undrop -> john
lrwxrwxrwx  1 admin admin        4 Feb 13  2023 unique -> john
-rw-r--r--  1 admin admin    21798 Feb 13  2023 unisubst.conf
-rwxr-xr-x  1 admin admin     1087 Feb 13  2023 unrule.pl
lrwxrwxrwx  1 admin admin        4 Feb 13  2023 unshadow -> john
-rw-r--r--  1 admin admin   668568 Feb 13  2023 upper.chr
-rw-r--r--  1 admin admin  1220961 Feb 13  2023 uppernum.chr
-rw-r--r--  1 admin admin  9286825 Feb 13  2023 utf8.chr
-rwxr-xr-x  1 admin admin     2250 Feb 13  2023 vdi2john.pl
-rwxr-xr-x  1 admin admin     2184 Feb 13  2023 vmx2john.py
-rwxr-xr-x  1 admin admin    45144 Feb 13  2023 vncpcap2john
-rwxr-xr-x  1 admin admin   220176 Feb 13  2023 wpapcap2john
-rwxr-xr-x  1 admin admin     4576 Feb 13  2023 zed2john.py
lrwxrwxrwx  1 admin admin        4 Feb 13  2023 zip2john -> john
drwxr-xr-x  2 admin admin     4096 Feb 13  2023 ztex
```

`curl localhost`  
```html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>401 Unauthorized</title>
</head><body>
<h1>Unauthorized</h1>
<p>This server could not verify that you
are authorized to access the document
requested.  Either you supplied the wrong
credentials (e.g., bad password), or your
browser doesn't understand how to supply
the credentials required.</p>
<hr>
<address>Apache/2.4.54 (Debian) Server at localhost Port 80</address>
</body></html>
```

`ls -la /var/www/html/`  
```console
total 12
drwxr-xr-x 2 root     root     4096 Feb 13  2023 .
drwxr-xr-x 3 root     root     4096 Feb 13  2023 ..
-rw-r----- 1 www-data www-data  215 Feb 13  2023 webfile
```

`cat /var/www/html/webfile`  
```console
cat: /var/www/html/webfile: Permission denied
```

`curl http://localhost/webfile`  
```html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>401 Unauthorized</title>
</head><body>
<h1>Unauthorized</h1>
<p>This server could not verify that you
are authorized to access the document
requested.  Either you supplied the wrong
credentials (e.g., bad password), or your
browser doesn't understand how to supply
the credentials required.</p>
<hr>
<address>Apache/2.4.54 (Debian) Server at localhost Port 80</address>
</body></html>
```
##### It's Basic Auth - the simplest type of auth, made by apache module `mod_auth_basic`. Username and password pairs are stored in the `.htpasswd` file. Lets search it

`ls -la /etc/apache2/`  
```console
total 92
drwxr-xr-x  8 root root  4096 Feb 13  2023 .
drwxr-xr-x 83 root root  4096 Feb 11 15:09 ..
-rw-r--r--  1 root root    45 Feb 13  2023 .htpasswd
-rw-r--r--  1 root root  7224 Jun  9  2022 apache2.conf
drwxr-xr-x  2 root root  4096 Feb 13  2023 conf-available
drwxr-xr-x  2 root root  4096 Feb 13  2023 conf-enabled
-rw-r--r--  1 root root  1782 Jun  9  2022 envvars
-rw-r--r--  1 root root 31063 Jun  9  2022 magic
drwxr-xr-x  2 root root 12288 Feb 13  2023 mods-available
drwxr-xr-x  2 root root  4096 Feb 13  2023 mods-enabled
-rw-r--r--  1 root root   320 Jun  9  2022 ports.conf
drwxr-xr-x  2 root root  4096 Feb 13  2023 sites-available
drwxr-xr-x  2 root root  4096 Feb 13  2023 sites-enabled
```

`cat /etc/apache2/.htpasswd`  
```console
carlos:$apr1$b1kyfnHB$yRHwzbuKSMyW62QTnGYCb0
```
##### We have login `carlos` and some encrypted password. Let's brutforce it by john-the-ripper tool.
##### Fast googling by prompt `john the ripper how to crack .htpasswd` says that it is easy-peasy: `john htpasswd`

/home/admin/john/run/john /etc/apache2/.htpasswd``  
```console
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 2 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
Warning: Only 34 candidates buffered for the current salt, minimum 48 needed for performance.
Almost done: Processing the remaining buffered candidate passwords, if any.
0g 0:00:00:00 DONE 1/3 (2024-02-11 15:15) 0g/s 8733p/s 8733c/s 8733C/s Carlos1921..Carlos1900
Proceeding with wordlist:/home/admin/john/run/password.lst
Enabling duplicate candidate password suppressor
chalet           (carlos)     
1g 0:00:00:01 DONE 2/3 (2024-02-11 15:15) 0.5376g/s 35763p/s 35763c/s 35763C/s 050381..song
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
##### Fine, it seems that the password is cracked, and it's `chalet`. Let's check web server with those credentials

`cd /home/admin/`  
`curl http://carlos:chalet@localhost/webfile`  
```console
Warning: Binary output can mess up your terminal. Use "--output -" to tell 
Warning: curl to output it to your terminal anyway, or consider "--output 
Warning: <FILE>" to save to a file.
```
##### Wweb server want to transfer a file with `--output -` option

`curl http://carlos:chalet@localhost/webfile --output -`  
```console
PK
        MVƪ
secret.txtUT    âcâcux
                      sƪPK
        MVƪ
secret.txtUTâcux
                PKPq
```
##### Add pipe to file


`curl http://carlos:chalet@localhost/webfile -o - | file -`  
```console
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   215  100   215    0     0   104k      0 --:--:-- --:--:-- --:--:--  104k
/dev/stdin: Zip archive data, at least v1.0 to extract
```
##### The curl says that this is `/dev/stdin: Zip archive data`. OK, let's download as zip file

`curl http://carlos:chalet@localhost/webfile -o - > file.zip`  
```console
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   215  100   215    0     0  53750      0 --:--:-- --:--:-- --:--:-- 71666
```

`ls -la`  
```console
total 40
drwxr-xr-x  6 admin admin 4096 Feb 11 15:19 .
drwxr-xr-x  3 root  root  4096 Aug 31  2022 ..
-rw-------  1 admin admin    0 Feb 20  2023 .bash_history
-rw-r--r--  1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r--  1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x  3 admin admin 4096 Feb 13  2023 .local
-rw-r--r--  1 admin admin  807 Aug  4  2021 .profile
drwx------  2 admin admin 4096 Aug 31  2022 .ssh
drwxr-xr-x  2 admin admin 4096 Feb 13  2023 agent
drwxr-xr-x 10 admin admin 4096 Feb 13  2023 john
-rw-r--r--  1 admin admin  215 Feb 11 15:19 file.zip
```
##### Let's unzip this archive

`unzip file.zip `  
```console
Archive:  file.zip
[file.zip] secret.txt password: 
```
##### It is an encrupted zip archive. John-the-ripper will help us again

`john/run/zip2john file.zip > zip.hashes`  
```console
ver 1.0 efh 5455 efh 7875 file.zip/secret.txt PKZIP Encr: 2b chk, TS_chk, cmplen=29, decmplen=17, crc=AAC6E9AF ts=14E0 cs=14e0 type=0
```

`john/run/john zip.hashes`  
```console
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
0g 0:00:00:00 DONE 1/3 (2024-02-11 15:20) 0g/s 411318p/s 411318c/s 411318C/s Txtout1900..Tsecret1900
Proceeding with wordlist:john/run/password.lst
Enabling duplicate candidate password suppressor
andes            (out.zip/secret.txt)     
1g 0:00:00:00 DONE 2/3 (2024-02-11 15:20) 1.098g/s 639363p/s 639363c/s 639363C/s poussinet..nisa1234
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
##### The password is `andes`

`unzip file.zip`  
```console
Archive:  out.zip
[out.zip] secret.txt password: 
 extracting: secret.txt
```
##### OK

`ls -la`  
```console
total 48
drwxr-xr-x  6 admin admin 4096 Feb 11 15:20 .
drwxr-xr-x  3 root  root  4096 Aug 31  2022 ..
-rw-------  1 admin admin    0 Feb 20  2023 .bash_history
-rw-r--r--  1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r--  1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x  3 admin admin 4096 Feb 13  2023 .local
-rw-r--r--  1 admin admin  807 Aug  4  2021 .profile
drwx------  2 admin admin 4096 Aug 31  2022 .ssh
drwxr-xr-x  2 admin admin 4096 Feb 13  2023 agent
drwxr-xr-x 10 admin admin 4096 Feb 13  2023 john
-rw-r--r--  1 admin admin  215 Feb 11 15:19 file.zip
-rw-r--r--  1 admin admin   17 Feb 13  2023 secret.txt
-rw-r--r--  1 admin admin  160 Feb 11 15:19 zip.hashes
```

`cat secret.txt`  
```console
Roseau, Dominica
```


#### 2. Validate the task
`sha1sum /home/admin/secret.txt |awk '{print $1}'`  
```console
cc2c322fbcac56923048d083b465901aac0fe8f8
```

`./agent/check.sh`  
```console
OK
```
