## Saint Paul: Merge Many CSVs files

**Description**: Join (merge) all the 338 files in `/home/admin/polldayregistrations_enregistjourduscrutin?????.csv` into one single `/home/admin/all.csv` file with the contents of all the CSV files in any order. There should be only one line with the names of the columns as a header.

**Test**: The "Check My Solution" button runs the script `/home/admin/agent/check.sh`, which you can see and execute.  

---

### Solution:
#### 1. Reconnaissance on the server
`pwd`  
```console
/home/admin
```

`ls -la`  
<details>
  <summary>Spoiler text output</summary>

```console
total 7476
drwxr-xr-x 5 admin admin  36864 Jul 28 02:42 .
drwxr-xr-x 3 root  root    4096 Feb 17  2024 ..
drwx------ 3 admin admin   4096 Feb 17  2024 .ansible
-rw-r--r-- 1 admin admin    220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 admin admin   3526 Mar 27  2022 .bashrc
-rw-r--r-- 1 admin admin    807 Mar 27  2022 .profile
drwx------ 2 admin admin   4096 Feb 17  2024 .ssh
-rw-r--r-- 1 admin admin    319 Jul 28 02:42 README.txt
drwxr-xr-x 2 admin root    4096 Jul 28 02:42 agent
-rw-r--r-- 1 admin admin 847708 Jul 28 02:42 polldayregistrations.zip
-rw-r--r-- 1 admin admin  12318 Feb 15  2022 polldayregistrations_enregistjourduscrutin10001.csv
-rw-r--r-- 1 admin admin  25121 Feb 15  2022 polldayregistrations_enregistjourduscrutin10002.csv
-rw-r--r-- 1 admin admin  25876 Feb 15  2022 polldayregistrations_enregistjourduscrutin10003.csv
-rw-r--r-- 1 admin admin   4252 Feb 15  2022 polldayregistrations_enregistjourduscrutin10004.csv
-rw-r--r-- 1 admin admin  20448 Feb 15  2022 polldayregistrations_enregistjourduscrutin10005.csv
-rw-r--r-- 1 admin admin  12592 Feb 15  2022 polldayregistrations_enregistjourduscrutin10006.csv
-rw-r--r-- 1 admin admin  17595 Feb 15  2022 polldayregistrations_enregistjourduscrutin10007.csv
-rw-r--r-- 1 admin admin   5165 Feb 15  2022 polldayregistrations_enregistjourduscrutin11001.csv
-rw-r--r-- 1 admin admin   7117 Feb 15  2022 polldayregistrations_enregistjourduscrutin11002.csv
-rw-r--r-- 1 admin admin   5083 Feb 15  2022 polldayregistrations_enregistjourduscrutin11003.csv
-rw-r--r-- 1 admin admin   5441 Feb 15  2022 polldayregistrations_enregistjourduscrutin11004.csv
-rw-r--r-- 1 admin admin  15734 Feb 15  2022 polldayregistrations_enregistjourduscrutin12001.csv
-rw-r--r-- 1 admin admin  15104 Feb 15  2022 polldayregistrations_enregistjourduscrutin12002.csv
-rw-r--r-- 1 admin admin  18921 Feb 15  2022 polldayregistrations_enregistjourduscrutin12003.csv
-rw-r--r-- 1 admin admin  17115 Feb 15  2022 polldayregistrations_enregistjourduscrutin12004.csv
-rw-r--r-- 1 admin admin  11821 Feb 15  2022 polldayregistrations_enregistjourduscrutin12005.csv
-rw-r--r-- 1 admin admin  15923 Feb 15  2022 polldayregistrations_enregistjourduscrutin12006.csv
-rw-r--r-- 1 admin admin  13706 Feb 15  2022 polldayregistrations_enregistjourduscrutin12007.csv
-rw-r--r-- 1 admin admin  18525 Feb 15  2022 polldayregistrations_enregistjourduscrutin12008.csv
-rw-r--r-- 1 admin admin  23788 Feb 15  2022 polldayregistrations_enregistjourduscrutin12009.csv
-rw-r--r-- 1 admin admin  13571 Feb 15  2022 polldayregistrations_enregistjourduscrutin12010.csv
-rw-r--r-- 1 admin admin  13906 Feb 15  2022 polldayregistrations_enregistjourduscrutin12011.csv
-rw-r--r-- 1 admin admin  23919 Feb 15  2022 polldayregistrations_enregistjourduscrutin13001.csv
-rw-r--r-- 1 admin admin  16415 Feb 15  2022 polldayregistrations_enregistjourduscrutin13002.csv
-rw-r--r-- 1 admin admin  11707 Feb 15  2022 polldayregistrations_enregistjourduscrutin13003.csv
-rw-r--r-- 1 admin admin  13311 Feb 15  2022 polldayregistrations_enregistjourduscrutin13004.csv
-rw-r--r-- 1 admin admin  17132 Feb 15  2022 polldayregistrations_enregistjourduscrutin13005.csv
-rw-r--r-- 1 admin admin  14681 Feb 15  2022 polldayregistrations_enregistjourduscrutin13006.csv
-rw-r--r-- 1 admin admin  23940 Feb 15  2022 polldayregistrations_enregistjourduscrutin13007.csv
-rw-r--r-- 1 admin admin  18054 Feb 15  2022 polldayregistrations_enregistjourduscrutin13008.csv
-rw-r--r-- 1 admin admin  16074 Feb 15  2022 polldayregistrations_enregistjourduscrutin13009.csv
-rw-r--r-- 1 admin admin  15481 Feb 15  2022 polldayregistrations_enregistjourduscrutin13010.csv
-rw-r--r-- 1 admin admin  21854 Mar  2  2022 polldayregistrations_enregistjourduscrutin24001.csv
-rw-r--r-- 1 admin admin  25972 Mar  2  2022 polldayregistrations_enregistjourduscrutin24002.csv
-rw-r--r-- 1 admin admin  18981 Mar  2  2022 polldayregistrations_enregistjourduscrutin24003.csv
-rw-r--r-- 1 admin admin  13360 Mar  2  2022 polldayregistrations_enregistjourduscrutin24004.csv
-rw-r--r-- 1 admin admin  24409 Mar  2  2022 polldayregistrations_enregistjourduscrutin24005.csv
-rw-r--r-- 1 admin admin  30063 Mar  2  2022 polldayregistrations_enregistjourduscrutin24006.csv
-rw-r--r-- 1 admin admin  15680 Mar  2  2022 polldayregistrations_enregistjourduscrutin24007.csv
-rw-r--r-- 1 admin admin  16146 Mar  2  2022 polldayregistrations_enregistjourduscrutin24008.csv
-rw-r--r-- 1 admin admin  24068 Mar  2  2022 polldayregistrations_enregistjourduscrutin24009.csv
-rw-r--r-- 1 admin admin  35979 Mar  2  2022 polldayregistrations_enregistjourduscrutin24010.csv
-rw-r--r-- 1 admin admin  20308 Mar  2  2022 polldayregistrations_enregistjourduscrutin24011.csv
-rw-r--r-- 1 admin admin  24476 Mar  2  2022 polldayregistrations_enregistjourduscrutin24012.csv
-rw-r--r-- 1 admin admin  20561 Mar  2  2022 polldayregistrations_enregistjourduscrutin24013.csv
-rw-r--r-- 1 admin admin  28150 Mar  2  2022 polldayregistrations_enregistjourduscrutin24014.csv
-rw-r--r-- 1 admin admin  12234 Mar  2  2022 polldayregistrations_enregistjourduscrutin24015.csv
-rw-r--r-- 1 admin admin  21414 Mar  2  2022 polldayregistrations_enregistjourduscrutin24016.csv
-rw-r--r-- 1 admin admin  18696 Mar  2  2022 polldayregistrations_enregistjourduscrutin24017.csv
-rw-r--r-- 1 admin admin  32369 Mar  2  2022 polldayregistrations_enregistjourduscrutin24018.csv
-rw-r--r-- 1 admin admin  23458 Mar  2  2022 polldayregistrations_enregistjourduscrutin24019.csv
-rw-r--r-- 1 admin admin  39493 Mar  2  2022 polldayregistrations_enregistjourduscrutin24020.csv
-rw-r--r-- 1 admin admin  19519 Mar  2  2022 polldayregistrations_enregistjourduscrutin24021.csv
-rw-r--r-- 1 admin admin  15028 Mar  2  2022 polldayregistrations_enregistjourduscrutin24022.csv
-rw-r--r-- 1 admin admin  23011 Mar  2  2022 polldayregistrations_enregistjourduscrutin24023.csv
-rw-r--r-- 1 admin admin  21052 Mar  2  2022 polldayregistrations_enregistjourduscrutin24024.csv
-rw-r--r-- 1 admin admin  15533 Mar  2  2022 polldayregistrations_enregistjourduscrutin24025.csv
-rw-r--r-- 1 admin admin  22991 Mar  2  2022 polldayregistrations_enregistjourduscrutin24026.csv
-rw-r--r-- 1 admin admin  12842 Mar  2  2022 polldayregistrations_enregistjourduscrutin24027.csv
-rw-r--r-- 1 admin admin  12298 Mar  2  2022 polldayregistrations_enregistjourduscrutin24028.csv
-rw-r--r-- 1 admin admin  14043 Mar  2  2022 polldayregistrations_enregistjourduscrutin24029.csv
-rw-r--r-- 1 admin admin  14105 Mar  2  2022 polldayregistrations_enregistjourduscrutin24030.csv
-rw-r--r-- 1 admin admin  17552 Mar  2  2022 polldayregistrations_enregistjourduscrutin24031.csv
-rw-r--r-- 1 admin admin  15933 Mar  2  2022 polldayregistrations_enregistjourduscrutin24032.csv
-rw-r--r-- 1 admin admin  20449 Mar  2  2022 polldayregistrations_enregistjourduscrutin24033.csv
-rw-r--r-- 1 admin admin  14568 Mar  2  2022 polldayregistrations_enregistjourduscrutin24034.csv
-rw-r--r-- 1 admin admin  21743 Mar  2  2022 polldayregistrations_enregistjourduscrutin24035.csv
-rw-r--r-- 1 admin admin  18370 Mar  2  2022 polldayregistrations_enregistjourduscrutin24036.csv
-rw-r--r-- 1 admin admin  16759 Mar  2  2022 polldayregistrations_enregistjourduscrutin24037.csv
-rw-r--r-- 1 admin admin  24305 Mar  2  2022 polldayregistrations_enregistjourduscrutin24038.csv
-rw-r--r-- 1 admin admin  13362 Mar  2  2022 polldayregistrations_enregistjourduscrutin24039.csv
-rw-r--r-- 1 admin admin  14538 Mar  2  2022 polldayregistrations_enregistjourduscrutin24040.csv
-rw-r--r-- 1 admin admin  19796 Mar  2  2022 polldayregistrations_enregistjourduscrutin24041.csv
-rw-r--r-- 1 admin admin  22408 Mar  2  2022 polldayregistrations_enregistjourduscrutin24042.csv
-rw-r--r-- 1 admin admin  18588 Mar  2  2022 polldayregistrations_enregistjourduscrutin24043.csv
-rw-r--r-- 1 admin admin  15346 Mar  2  2022 polldayregistrations_enregistjourduscrutin24044.csv
-rw-r--r-- 1 admin admin  19309 Mar  2  2022 polldayregistrations_enregistjourduscrutin24045.csv
-rw-r--r-- 1 admin admin  16093 Mar  2  2022 polldayregistrations_enregistjourduscrutin24046.csv
-rw-r--r-- 1 admin admin  22486 Mar  2  2022 polldayregistrations_enregistjourduscrutin24047.csv
-rw-r--r-- 1 admin admin  14965 Mar  2  2022 polldayregistrations_enregistjourduscrutin24048.csv
-rw-r--r-- 1 admin admin  13963 Mar  2  2022 polldayregistrations_enregistjourduscrutin24049.csv
-rw-r--r-- 1 admin admin  13369 Mar  2  2022 polldayregistrations_enregistjourduscrutin24050.csv
-rw-r--r-- 1 admin admin  46788 Mar  2  2022 polldayregistrations_enregistjourduscrutin24051.csv
-rw-r--r-- 1 admin admin  13938 Mar  2  2022 polldayregistrations_enregistjourduscrutin24052.csv
-rw-r--r-- 1 admin admin  21180 Mar  2  2022 polldayregistrations_enregistjourduscrutin24053.csv
-rw-r--r-- 1 admin admin  10411 Mar  2  2022 polldayregistrations_enregistjourduscrutin24054.csv
-rw-r--r-- 1 admin admin  10639 Mar  2  2022 polldayregistrations_enregistjourduscrutin24055.csv
-rw-r--r-- 1 admin admin  18231 Mar  2  2022 polldayregistrations_enregistjourduscrutin24056.csv
-rw-r--r-- 1 admin admin  16843 Mar  2  2022 polldayregistrations_enregistjourduscrutin24057.csv
-rw-r--r-- 1 admin admin  27552 Mar  2  2022 polldayregistrations_enregistjourduscrutin24058.csv
-rw-r--r-- 1 admin admin  12426 Mar  2  2022 polldayregistrations_enregistjourduscrutin24059.csv
-rw-r--r-- 1 admin admin  15526 Mar  2  2022 polldayregistrations_enregistjourduscrutin24060.csv
-rw-r--r-- 1 admin admin  23349 Mar  2  2022 polldayregistrations_enregistjourduscrutin24061.csv
-rw-r--r-- 1 admin admin  20037 Mar  2  2022 polldayregistrations_enregistjourduscrutin24062.csv
-rw-r--r-- 1 admin admin  19543 Feb 15  2022 polldayregistrations_enregistjourduscrutin24063.csv
-rw-r--r-- 1 admin admin  21070 Mar  2  2022 polldayregistrations_enregistjourduscrutin24064.csv
-rw-r--r-- 1 admin admin  12662 Mar  2  2022 polldayregistrations_enregistjourduscrutin24065.csv
-rw-r--r-- 1 admin admin  21584 Mar  2  2022 polldayregistrations_enregistjourduscrutin24066.csv
-rw-r--r-- 1 admin admin  17518 Mar  2  2022 polldayregistrations_enregistjourduscrutin24067.csv
-rw-r--r-- 1 admin admin  10803 Mar  2  2022 polldayregistrations_enregistjourduscrutin24068.csv
-rw-r--r-- 1 admin admin  17878 Mar  2  2022 polldayregistrations_enregistjourduscrutin24069.csv
-rw-r--r-- 1 admin admin  26011 Mar  2  2022 polldayregistrations_enregistjourduscrutin24070.csv
-rw-r--r-- 1 admin admin  23685 Mar  2  2022 polldayregistrations_enregistjourduscrutin24071.csv
-rw-r--r-- 1 admin admin  16086 Mar  2  2022 polldayregistrations_enregistjourduscrutin24072.csv
-rw-r--r-- 1 admin admin  17716 Mar  2  2022 polldayregistrations_enregistjourduscrutin24073.csv
-rw-r--r-- 1 admin admin  21764 Mar  2  2022 polldayregistrations_enregistjourduscrutin24074.csv
-rw-r--r-- 1 admin admin  11740 Mar  2  2022 polldayregistrations_enregistjourduscrutin24075.csv
-rw-r--r-- 1 admin admin  23311 Mar  2  2022 polldayregistrations_enregistjourduscrutin24076.csv
-rw-r--r-- 1 admin admin  26995 Mar  2  2022 polldayregistrations_enregistjourduscrutin24077.csv
-rw-r--r-- 1 admin admin  11290 Mar  2  2022 polldayregistrations_enregistjourduscrutin24078.csv
-rw-r--r-- 1 admin admin  11822 Feb 15  2022 polldayregistrations_enregistjourduscrutin35001.csv
-rw-r--r-- 1 admin admin  20449 Feb 15  2022 polldayregistrations_enregistjourduscrutin35002.csv
-rw-r--r-- 1 admin admin  46083 Feb 15  2022 polldayregistrations_enregistjourduscrutin35003.csv
-rw-r--r-- 1 admin admin  11913 Feb 15  2022 polldayregistrations_enregistjourduscrutin35004.csv
-rw-r--r-- 1 admin admin  19484 Feb 15  2022 polldayregistrations_enregistjourduscrutin35005.csv
-rw-r--r-- 1 admin admin  17815 Feb 15  2022 polldayregistrations_enregistjourduscrutin35006.csv
-rw-r--r-- 1 admin admin  14799 Feb 15  2022 polldayregistrations_enregistjourduscrutin35007.csv
-rw-r--r-- 1 admin admin  12034 Feb 15  2022 polldayregistrations_enregistjourduscrutin35008.csv
-rw-r--r-- 1 admin admin  11529 Feb 15  2022 polldayregistrations_enregistjourduscrutin35009.csv
-rw-r--r-- 1 admin admin  12270 Feb 15  2022 polldayregistrations_enregistjourduscrutin35010.csv
-rw-r--r-- 1 admin admin  12378 Feb 15  2022 polldayregistrations_enregistjourduscrutin35011.csv
-rw-r--r-- 1 admin admin  12308 Feb 15  2022 polldayregistrations_enregistjourduscrutin35012.csv
-rw-r--r-- 1 admin admin  16457 Feb 15  2022 polldayregistrations_enregistjourduscrutin35013.csv
-rw-r--r-- 1 admin admin  21977 Feb 15  2022 polldayregistrations_enregistjourduscrutin35014.csv
-rw-r--r-- 1 admin admin  18514 Feb 15  2022 polldayregistrations_enregistjourduscrutin35015.csv
-rw-r--r-- 1 admin admin  12400 Feb 15  2022 polldayregistrations_enregistjourduscrutin35016.csv
-rw-r--r-- 1 admin admin  22440 Feb 15  2022 polldayregistrations_enregistjourduscrutin35017.csv
-rw-r--r-- 1 admin admin  10927 Feb 15  2022 polldayregistrations_enregistjourduscrutin35018.csv
-rw-r--r-- 1 admin admin  37381 Feb 15  2022 polldayregistrations_enregistjourduscrutin35019.csv
-rw-r--r-- 1 admin admin  16711 Feb 15  2022 polldayregistrations_enregistjourduscrutin35020.csv
-rw-r--r-- 1 admin admin  13437 Feb 15  2022 polldayregistrations_enregistjourduscrutin35021.csv
-rw-r--r-- 1 admin admin  16138 Feb 15  2022 polldayregistrations_enregistjourduscrutin35022.csv
-rw-r--r-- 1 admin admin  17537 Feb 15  2022 polldayregistrations_enregistjourduscrutin35023.csv
-rw-r--r-- 1 admin admin  18987 Feb 15  2022 polldayregistrations_enregistjourduscrutin35024.csv
-rw-r--r-- 1 admin admin  17199 Feb 15  2022 polldayregistrations_enregistjourduscrutin35025.csv
-rw-r--r-- 1 admin admin  11580 Feb 15  2022 polldayregistrations_enregistjourduscrutin35026.csv
-rw-r--r-- 1 admin admin  16151 Feb 15  2022 polldayregistrations_enregistjourduscrutin35027.csv
-rw-r--r-- 1 admin admin  19115 Feb 15  2022 polldayregistrations_enregistjourduscrutin35028.csv
-rw-r--r-- 1 admin admin  11604 Feb 15  2022 polldayregistrations_enregistjourduscrutin35029.csv
-rw-r--r-- 1 admin admin  20071 Feb 15  2022 polldayregistrations_enregistjourduscrutin35030.csv
-rw-r--r-- 1 admin admin  28718 Feb 15  2022 polldayregistrations_enregistjourduscrutin35031.csv
-rw-r--r-- 1 admin admin  12834 Feb 15  2022 polldayregistrations_enregistjourduscrutin35032.csv
-rw-r--r-- 1 admin admin  19159 Feb 15  2022 polldayregistrations_enregistjourduscrutin35033.csv
-rw-r--r-- 1 admin admin  28747 Feb 15  2022 polldayregistrations_enregistjourduscrutin35034.csv
-rw-r--r-- 1 admin admin  15188 Feb 15  2022 polldayregistrations_enregistjourduscrutin35035.csv
-rw-r--r-- 1 admin admin  19123 Feb 15  2022 polldayregistrations_enregistjourduscrutin35036.csv
-rw-r--r-- 1 admin admin  16192 Feb 15  2022 polldayregistrations_enregistjourduscrutin35037.csv
-rw-r--r-- 1 admin admin  24695 Feb 15  2022 polldayregistrations_enregistjourduscrutin35038.csv
-rw-r--r-- 1 admin admin  22623 Feb 15  2022 polldayregistrations_enregistjourduscrutin35039.csv
-rw-r--r-- 1 admin admin  16728 Feb 15  2022 polldayregistrations_enregistjourduscrutin35040.csv
-rw-r--r-- 1 admin admin  15910 Feb 15  2022 polldayregistrations_enregistjourduscrutin35041.csv
-rw-r--r-- 1 admin admin   7691 Feb 15  2022 polldayregistrations_enregistjourduscrutin35042.csv
-rw-r--r-- 1 admin admin  18425 Feb 15  2022 polldayregistrations_enregistjourduscrutin35043.csv
-rw-r--r-- 1 admin admin  22267 Feb 15  2022 polldayregistrations_enregistjourduscrutin35044.csv
-rw-r--r-- 1 admin admin  15220 Feb 15  2022 polldayregistrations_enregistjourduscrutin35045.csv
-rw-r--r-- 1 admin admin  20982 Feb 15  2022 polldayregistrations_enregistjourduscrutin35046.csv
-rw-r--r-- 1 admin admin  17246 Feb 15  2022 polldayregistrations_enregistjourduscrutin35047.csv
-rw-r--r-- 1 admin admin  22203 Feb 15  2022 polldayregistrations_enregistjourduscrutin35048.csv
-rw-r--r-- 1 admin admin  23808 Feb 15  2022 polldayregistrations_enregistjourduscrutin35049.csv
-rw-r--r-- 1 admin admin  34425 Mar  1  2022 polldayregistrations_enregistjourduscrutin35050.csv
-rw-r--r-- 1 admin admin  15510 Feb 15  2022 polldayregistrations_enregistjourduscrutin35051.csv
-rw-r--r-- 1 admin admin  34453 Feb 15  2022 polldayregistrations_enregistjourduscrutin35052.csv
-rw-r--r-- 1 admin admin  14698 Feb 15  2022 polldayregistrations_enregistjourduscrutin35053.csv
-rw-r--r-- 1 admin admin  21015 Feb 15  2022 polldayregistrations_enregistjourduscrutin35054.csv
-rw-r--r-- 1 admin admin  12647 Feb 15  2022 polldayregistrations_enregistjourduscrutin35055.csv
-rw-r--r-- 1 admin admin  16915 Feb 15  2022 polldayregistrations_enregistjourduscrutin35056.csv
-rw-r--r-- 1 admin admin   9300 Feb 15  2022 polldayregistrations_enregistjourduscrutin35057.csv
-rw-r--r-- 1 admin admin  17508 Feb 15  2022 polldayregistrations_enregistjourduscrutin35058.csv
-rw-r--r-- 1 admin admin  20536 Feb 15  2022 polldayregistrations_enregistjourduscrutin35059.csv
-rw-r--r-- 1 admin admin  19320 Feb 15  2022 polldayregistrations_enregistjourduscrutin35060.csv
-rw-r--r-- 1 admin admin  20772 Feb 15  2022 polldayregistrations_enregistjourduscrutin35061.csv
-rw-r--r-- 1 admin admin  14026 Feb 15  2022 polldayregistrations_enregistjourduscrutin35062.csv
-rw-r--r-- 1 admin admin  18122 Feb 15  2022 polldayregistrations_enregistjourduscrutin35063.csv
-rw-r--r-- 1 admin admin  10742 Feb 15  2022 polldayregistrations_enregistjourduscrutin35064.csv
-rw-r--r-- 1 admin admin  17372 Feb 15  2022 polldayregistrations_enregistjourduscrutin35065.csv
-rw-r--r-- 1 admin admin  14411 Feb 15  2022 polldayregistrations_enregistjourduscrutin35066.csv
-rw-r--r-- 1 admin admin  16723 Feb 15  2022 polldayregistrations_enregistjourduscrutin35067.csv
-rw-r--r-- 1 admin admin  12737 Feb 15  2022 polldayregistrations_enregistjourduscrutin35068.csv
-rw-r--r-- 1 admin admin  12236 Feb 15  2022 polldayregistrations_enregistjourduscrutin35069.csv
-rw-r--r-- 1 admin admin  18942 Feb 15  2022 polldayregistrations_enregistjourduscrutin35070.csv
-rw-r--r-- 1 admin admin  30265 Feb 15  2022 polldayregistrations_enregistjourduscrutin35071.csv
-rw-r--r-- 1 admin admin  13405 Feb 15  2022 polldayregistrations_enregistjourduscrutin35072.csv
-rw-r--r-- 1 admin admin  21009 Feb 15  2022 polldayregistrations_enregistjourduscrutin35073.csv
-rw-r--r-- 1 admin admin  15112 Feb 15  2022 polldayregistrations_enregistjourduscrutin35074.csv
-rw-r--r-- 1 admin admin  16500 Feb 15  2022 polldayregistrations_enregistjourduscrutin35075.csv
-rw-r--r-- 1 admin admin  14578 Feb 15  2022 polldayregistrations_enregistjourduscrutin35076.csv
-rw-r--r-- 1 admin admin  14801 Feb 15  2022 polldayregistrations_enregistjourduscrutin35077.csv
-rw-r--r-- 1 admin admin  17110 Feb 15  2022 polldayregistrations_enregistjourduscrutin35078.csv
-rw-r--r-- 1 admin admin  19132 Feb 15  2022 polldayregistrations_enregistjourduscrutin35079.csv
-rw-r--r-- 1 admin admin  13663 Feb 15  2022 polldayregistrations_enregistjourduscrutin35080.csv
-rw-r--r-- 1 admin admin  15932 Feb 15  2022 polldayregistrations_enregistjourduscrutin35081.csv
-rw-r--r-- 1 admin admin  19275 Feb 15  2022 polldayregistrations_enregistjourduscrutin35082.csv
-rw-r--r-- 1 admin admin  14591 Feb 15  2022 polldayregistrations_enregistjourduscrutin35083.csv
-rw-r--r-- 1 admin admin  23937 Feb 15  2022 polldayregistrations_enregistjourduscrutin35084.csv
-rw-r--r-- 1 admin admin  22086 Feb 15  2022 polldayregistrations_enregistjourduscrutin35085.csv
-rw-r--r-- 1 admin admin  23502 Feb 15  2022 polldayregistrations_enregistjourduscrutin35086.csv
-rw-r--r-- 1 admin admin  15490 Feb 15  2022 polldayregistrations_enregistjourduscrutin35087.csv
-rw-r--r-- 1 admin admin  14716 Feb 15  2022 polldayregistrations_enregistjourduscrutin35088.csv
-rw-r--r-- 1 admin admin  16323 Feb 15  2022 polldayregistrations_enregistjourduscrutin35089.csv
-rw-r--r-- 1 admin admin  17805 Feb 15  2022 polldayregistrations_enregistjourduscrutin35090.csv
-rw-r--r-- 1 admin admin  18518 Feb 15  2022 polldayregistrations_enregistjourduscrutin35091.csv
-rw-r--r-- 1 admin admin  14175 Feb 15  2022 polldayregistrations_enregistjourduscrutin35092.csv
-rw-r--r-- 1 admin admin  17010 Feb 15  2022 polldayregistrations_enregistjourduscrutin35093.csv
-rw-r--r-- 1 admin admin  15509 Feb 15  2022 polldayregistrations_enregistjourduscrutin35094.csv
-rw-r--r-- 1 admin admin  14623 Feb 15  2022 polldayregistrations_enregistjourduscrutin35095.csv
-rw-r--r-- 1 admin admin  13978 Feb 15  2022 polldayregistrations_enregistjourduscrutin35096.csv
-rw-r--r-- 1 admin admin  15394 Feb 15  2022 polldayregistrations_enregistjourduscrutin35097.csv
-rw-r--r-- 1 admin admin  18909 Feb 15  2022 polldayregistrations_enregistjourduscrutin35098.csv
-rw-r--r-- 1 admin admin  22103 Feb 15  2022 polldayregistrations_enregistjourduscrutin35099.csv
-rw-r--r-- 1 admin admin  16277 Feb 15  2022 polldayregistrations_enregistjourduscrutin35100.csv
-rw-r--r-- 1 admin admin  21256 Feb 15  2022 polldayregistrations_enregistjourduscrutin35101.csv
-rw-r--r-- 1 admin admin  26833 Feb 15  2022 polldayregistrations_enregistjourduscrutin35102.csv
-rw-r--r-- 1 admin admin  15913 Feb 15  2022 polldayregistrations_enregistjourduscrutin35103.csv
-rw-r--r-- 1 admin admin  14477 Feb 15  2022 polldayregistrations_enregistjourduscrutin35104.csv
-rw-r--r-- 1 admin admin  16673 Feb 15  2022 polldayregistrations_enregistjourduscrutin35105.csv
-rw-r--r-- 1 admin admin  19477 Feb 15  2022 polldayregistrations_enregistjourduscrutin35106.csv
-rw-r--r-- 1 admin admin  12588 Feb 15  2022 polldayregistrations_enregistjourduscrutin35107.csv
-rw-r--r-- 1 admin admin  18970 Feb 15  2022 polldayregistrations_enregistjourduscrutin35108.csv
-rw-r--r-- 1 admin admin  16769 Feb 15  2022 polldayregistrations_enregistjourduscrutin35109.csv
-rw-r--r-- 1 admin admin  23090 Feb 15  2022 polldayregistrations_enregistjourduscrutin35110.csv
-rw-r--r-- 1 admin admin  14600 Feb 15  2022 polldayregistrations_enregistjourduscrutin35111.csv
-rw-r--r-- 1 admin admin  12536 Feb 15  2022 polldayregistrations_enregistjourduscrutin35112.csv
-rw-r--r-- 1 admin admin  24201 Feb 15  2022 polldayregistrations_enregistjourduscrutin35113.csv
-rw-r--r-- 1 admin admin  15891 Feb 15  2022 polldayregistrations_enregistjourduscrutin35114.csv
-rw-r--r-- 1 admin admin  12167 Feb 15  2022 polldayregistrations_enregistjourduscrutin35115.csv
-rw-r--r-- 1 admin admin  17974 Feb 15  2022 polldayregistrations_enregistjourduscrutin35116.csv
-rw-r--r-- 1 admin admin  14466 Feb 15  2022 polldayregistrations_enregistjourduscrutin35117.csv
-rw-r--r-- 1 admin admin  13287 Feb 15  2022 polldayregistrations_enregistjourduscrutin35118.csv
-rw-r--r-- 1 admin admin  15957 Feb 15  2022 polldayregistrations_enregistjourduscrutin35119.csv
-rw-r--r-- 1 admin admin  14333 Feb 15  2022 polldayregistrations_enregistjourduscrutin35120.csv
-rw-r--r-- 1 admin admin  13947 Feb 15  2022 polldayregistrations_enregistjourduscrutin35121.csv
-rw-r--r-- 1 admin admin  12367 Feb 15  2022 polldayregistrations_enregistjourduscrutin46001.csv
-rw-r--r-- 1 admin admin  23315 Feb 15  2022 polldayregistrations_enregistjourduscrutin46002.csv
-rw-r--r-- 1 admin admin  14519 Feb 15  2022 polldayregistrations_enregistjourduscrutin46003.csv
-rw-r--r-- 1 admin admin  27170 Feb 15  2022 polldayregistrations_enregistjourduscrutin46004.csv
-rw-r--r-- 1 admin admin  15391 Feb 15  2022 polldayregistrations_enregistjourduscrutin46005.csv
-rw-r--r-- 1 admin admin  14606 Feb 15  2022 polldayregistrations_enregistjourduscrutin46006.csv
-rw-r--r-- 1 admin admin  20019 Feb 15  2022 polldayregistrations_enregistjourduscrutin46007.csv
-rw-r--r-- 1 admin admin  13190 Feb 15  2022 polldayregistrations_enregistjourduscrutin46008.csv
-rw-r--r-- 1 admin admin  17685 Feb 15  2022 polldayregistrations_enregistjourduscrutin46009.csv
-rw-r--r-- 1 admin admin  27200 Feb 15  2022 polldayregistrations_enregistjourduscrutin46010.csv
-rw-r--r-- 1 admin admin  12130 Feb 15  2022 polldayregistrations_enregistjourduscrutin46011.csv
-rw-r--r-- 1 admin admin  11661 Feb 15  2022 polldayregistrations_enregistjourduscrutin46012.csv
-rw-r--r-- 1 admin admin  11281 Feb 15  2022 polldayregistrations_enregistjourduscrutin46013.csv
-rw-r--r-- 1 admin admin  15319 Feb 15  2022 polldayregistrations_enregistjourduscrutin46014.csv
-rw-r--r-- 1 admin admin  14142 Feb 15  2022 polldayregistrations_enregistjourduscrutin47001.csv
-rw-r--r-- 1 admin admin  16752 Feb 15  2022 polldayregistrations_enregistjourduscrutin47002.csv
-rw-r--r-- 1 admin admin  26268 Feb 15  2022 polldayregistrations_enregistjourduscrutin47003.csv
-rw-r--r-- 1 admin admin  17011 Feb 15  2022 polldayregistrations_enregistjourduscrutin47004.csv
-rw-r--r-- 1 admin admin  19066 Feb 15  2022 polldayregistrations_enregistjourduscrutin47005.csv
-rw-r--r-- 1 admin admin  12929 Feb 15  2022 polldayregistrations_enregistjourduscrutin47006.csv
-rw-r--r-- 1 admin admin  13963 Feb 15  2022 polldayregistrations_enregistjourduscrutin47007.csv
-rw-r--r-- 1 admin admin  13794 Feb 15  2022 polldayregistrations_enregistjourduscrutin47008.csv
-rw-r--r-- 1 admin admin  13445 Feb 15  2022 polldayregistrations_enregistjourduscrutin47009.csv
-rw-r--r-- 1 admin admin  15568 Feb 15  2022 polldayregistrations_enregistjourduscrutin47010.csv
-rw-r--r-- 1 admin admin  13324 Feb 15  2022 polldayregistrations_enregistjourduscrutin47011.csv
-rw-r--r-- 1 admin admin  11260 Feb 15  2022 polldayregistrations_enregistjourduscrutin47012.csv
-rw-r--r-- 1 admin admin  16900 Feb 15  2022 polldayregistrations_enregistjourduscrutin47013.csv
-rw-r--r-- 1 admin admin  14985 Feb 15  2022 polldayregistrations_enregistjourduscrutin47014.csv
-rw-r--r-- 1 admin admin  20019 Feb 15  2022 polldayregistrations_enregistjourduscrutin48001.csv
-rw-r--r-- 1 admin admin  22818 Feb 15  2022 polldayregistrations_enregistjourduscrutin48002.csv
-rw-r--r-- 1 admin admin  12147 Feb 15  2022 polldayregistrations_enregistjourduscrutin48003.csv
-rw-r--r-- 1 admin admin  16876 Feb 15  2022 polldayregistrations_enregistjourduscrutin48004.csv
-rw-r--r-- 1 admin admin  21505 Feb 15  2022 polldayregistrations_enregistjourduscrutin48005.csv
-rw-r--r-- 1 admin admin  14403 Feb 15  2022 polldayregistrations_enregistjourduscrutin48006.csv
-rw-r--r-- 1 admin admin  13183 Feb 15  2022 polldayregistrations_enregistjourduscrutin48007.csv
-rw-r--r-- 1 admin admin  16804 Feb 15  2022 polldayregistrations_enregistjourduscrutin48008.csv
-rw-r--r-- 1 admin admin  14417 Feb 15  2022 polldayregistrations_enregistjourduscrutin48009.csv
-rw-r--r-- 1 admin admin  17681 Feb 15  2022 polldayregistrations_enregistjourduscrutin48010.csv
-rw-r--r-- 1 admin admin  18658 Feb 15  2022 polldayregistrations_enregistjourduscrutin48011.csv
-rw-r--r-- 1 admin admin  17181 Feb 15  2022 polldayregistrations_enregistjourduscrutin48012.csv
-rw-r--r-- 1 admin admin  13843 Feb 15  2022 polldayregistrations_enregistjourduscrutin48013.csv
-rw-r--r-- 1 admin admin  14268 Feb 15  2022 polldayregistrations_enregistjourduscrutin48014.csv
-rw-r--r-- 1 admin admin  18770 Feb 15  2022 polldayregistrations_enregistjourduscrutin48015.csv
-rw-r--r-- 1 admin admin  17561 Feb 15  2022 polldayregistrations_enregistjourduscrutin48016.csv
-rw-r--r-- 1 admin admin  14944 Feb 15  2022 polldayregistrations_enregistjourduscrutin48017.csv
-rw-r--r-- 1 admin admin  17308 Feb 15  2022 polldayregistrations_enregistjourduscrutin48018.csv
-rw-r--r-- 1 admin admin  16696 Feb 15  2022 polldayregistrations_enregistjourduscrutin48019.csv
-rw-r--r-- 1 admin admin  15706 Feb 15  2022 polldayregistrations_enregistjourduscrutin48020.csv
-rw-r--r-- 1 admin admin  24799 Feb 15  2022 polldayregistrations_enregistjourduscrutin48021.csv
-rw-r--r-- 1 admin admin  15977 Feb 15  2022 polldayregistrations_enregistjourduscrutin48022.csv
-rw-r--r-- 1 admin admin  21154 Feb 15  2022 polldayregistrations_enregistjourduscrutin48023.csv
-rw-r--r-- 1 admin admin  21912 Feb 15  2022 polldayregistrations_enregistjourduscrutin48024.csv
-rw-r--r-- 1 admin admin  16690 Feb 15  2022 polldayregistrations_enregistjourduscrutin48025.csv
-rw-r--r-- 1 admin admin  15055 Feb 15  2022 polldayregistrations_enregistjourduscrutin48026.csv
-rw-r--r-- 1 admin admin  23488 Feb 15  2022 polldayregistrations_enregistjourduscrutin48027.csv
-rw-r--r-- 1 admin admin  19553 Feb 15  2022 polldayregistrations_enregistjourduscrutin48028.csv
-rw-r--r-- 1 admin admin  21359 Feb 15  2022 polldayregistrations_enregistjourduscrutin48029.csv
-rw-r--r-- 1 admin admin  19647 Feb 15  2022 polldayregistrations_enregistjourduscrutin48030.csv
-rw-r--r-- 1 admin admin  18960 Feb 15  2022 polldayregistrations_enregistjourduscrutin48031.csv
-rw-r--r-- 1 admin admin  27423 Feb 15  2022 polldayregistrations_enregistjourduscrutin48032.csv
-rw-r--r-- 1 admin admin  25299 Feb 15  2022 polldayregistrations_enregistjourduscrutin48033.csv
-rw-r--r-- 1 admin admin  16189 Feb 15  2022 polldayregistrations_enregistjourduscrutin48034.csv
-rw-r--r-- 1 admin admin  10552 Feb 15  2022 polldayregistrations_enregistjourduscrutin59001.csv
-rw-r--r-- 1 admin admin  15290 Feb 15  2022 polldayregistrations_enregistjourduscrutin59002.csv
-rw-r--r-- 1 admin admin  11930 Feb 15  2022 polldayregistrations_enregistjourduscrutin59003.csv
-rw-r--r-- 1 admin admin  19411 Feb 15  2022 polldayregistrations_enregistjourduscrutin59004.csv
-rw-r--r-- 1 admin admin  29192 Feb 15  2022 polldayregistrations_enregistjourduscrutin59005.csv
-rw-r--r-- 1 admin admin  14610 Feb 15  2022 polldayregistrations_enregistjourduscrutin59006.csv
-rw-r--r-- 1 admin admin  16956 Feb 15  2022 polldayregistrations_enregistjourduscrutin59007.csv
-rw-r--r-- 1 admin admin  19113 Feb 15  2022 polldayregistrations_enregistjourduscrutin59008.csv
-rw-r--r-- 1 admin admin  19304 Feb 15  2022 polldayregistrations_enregistjourduscrutin59009.csv
-rw-r--r-- 1 admin admin  22571 Feb 15  2022 polldayregistrations_enregistjourduscrutin59010.csv
-rw-r--r-- 1 admin admin   9113 Feb 15  2022 polldayregistrations_enregistjourduscrutin59011.csv
-rw-r--r-- 1 admin admin  14459 Feb 15  2022 polldayregistrations_enregistjourduscrutin59012.csv
-rw-r--r-- 1 admin admin  26554 Feb 15  2022 polldayregistrations_enregistjourduscrutin59013.csv
-rw-r--r-- 1 admin admin  18529 Feb 15  2022 polldayregistrations_enregistjourduscrutin59014.csv
-rw-r--r-- 1 admin admin  20729 Feb 15  2022 polldayregistrations_enregistjourduscrutin59015.csv
-rw-r--r-- 1 admin admin  21527 Feb 15  2022 polldayregistrations_enregistjourduscrutin59016.csv
-rw-r--r-- 1 admin admin  17276 Feb 15  2022 polldayregistrations_enregistjourduscrutin59017.csv
-rw-r--r-- 1 admin admin  19900 Feb 15  2022 polldayregistrations_enregistjourduscrutin59018.csv
-rw-r--r-- 1 admin admin  19051 Feb 15  2022 polldayregistrations_enregistjourduscrutin59019.csv
-rw-r--r-- 1 admin admin  25428 Feb 15  2022 polldayregistrations_enregistjourduscrutin59020.csv
-rw-r--r-- 1 admin admin  15344 Feb 15  2022 polldayregistrations_enregistjourduscrutin59021.csv
-rw-r--r-- 1 admin admin  16675 Feb 15  2022 polldayregistrations_enregistjourduscrutin59022.csv
-rw-r--r-- 1 admin admin  16837 Feb 15  2022 polldayregistrations_enregistjourduscrutin59023.csv
-rw-r--r-- 1 admin admin  30877 Feb 15  2022 polldayregistrations_enregistjourduscrutin59024.csv
-rw-r--r-- 1 admin admin  13275 Feb 15  2022 polldayregistrations_enregistjourduscrutin59025.csv
-rw-r--r-- 1 admin admin  19342 Feb 15  2022 polldayregistrations_enregistjourduscrutin59026.csv
-rw-r--r-- 1 admin admin  19699 Feb 15  2022 polldayregistrations_enregistjourduscrutin59027.csv
-rw-r--r-- 1 admin admin  18071 Feb 15  2022 polldayregistrations_enregistjourduscrutin59028.csv
-rw-r--r-- 1 admin admin  28008 Feb 15  2022 polldayregistrations_enregistjourduscrutin59029.csv
-rw-r--r-- 1 admin admin  17856 Feb 15  2022 polldayregistrations_enregistjourduscrutin59030.csv
-rw-r--r-- 1 admin admin  15129 Feb 15  2022 polldayregistrations_enregistjourduscrutin59031.csv
-rw-r--r-- 1 admin admin  12302 Feb 15  2022 polldayregistrations_enregistjourduscrutin59032.csv
-rw-r--r-- 1 admin admin  10473 Feb 15  2022 polldayregistrations_enregistjourduscrutin59033.csv
-rw-r--r-- 1 admin admin  17400 Feb 15  2022 polldayregistrations_enregistjourduscrutin59034.csv
-rw-r--r-- 1 admin admin  16415 Feb 15  2022 polldayregistrations_enregistjourduscrutin59035.csv
-rw-r--r-- 1 admin admin  16552 Feb 15  2022 polldayregistrations_enregistjourduscrutin59036.csv
-rw-r--r-- 1 admin admin  23795 Feb 15  2022 polldayregistrations_enregistjourduscrutin59037.csv
-rw-r--r-- 1 admin admin  14375 Feb 15  2022 polldayregistrations_enregistjourduscrutin59038.csv
-rw-r--r-- 1 admin admin  12093 Feb 15  2022 polldayregistrations_enregistjourduscrutin59039.csv
-rw-r--r-- 1 admin admin  13119 Feb 15  2022 polldayregistrations_enregistjourduscrutin59040.csv
-rw-r--r-- 1 admin admin  14949 Feb 15  2022 polldayregistrations_enregistjourduscrutin59041.csv
-rw-r--r-- 1 admin admin  35796 Feb 15  2022 polldayregistrations_enregistjourduscrutin59042.csv
-rw-r--r-- 1 admin admin  24676 Feb 15  2022 polldayregistrations_enregistjourduscrutin60001.csv
-rw-r--r-- 1 admin admin   8265 Feb 15  2022 polldayregistrations_enregistjourduscrutin61001.csv
-rw-r--r-- 1 admin admin   2888 Feb 15  2022 polldayregistrations_enregistjourduscrutin62001.csv
```

</details>


`ls -la | wc -l`  
```console
349
```

`cat /home/admin/agent/check.sh`  
```bash
#!/usr/bin/bash
# DO NOT MODIFY THIS FILE ("Check My Solution" will fail)
cd /home/admin
expected_header=$(head -n 1 polldayregistrations_enregistjourduscrutin62001.csv)
expected_lines=72461

# count number of lines in the all.csv file
lines=$(wc -l all.csv | cut -d ' ' -f 1)

# if lines is not exaclty expected_lines, print "NO"
if [ $lines -ne $expected_lines ]; then
  echo -n "NO"
  exit
fi

# if expected_header is more than once in all.csv, print "NO"
if [ $(grep -c "$expected_header" all.csv) -gt 1 ]; then
  echo -n "NO"
  exit
fi

# get a random existing file from the current directory
random_file=$(ls polldayregistrations_enregistjourduscrutin?????.csv| shuf -n 1)

# get a random line from any of the pollbypoll_bureauparbureau?????.csv
random_line=$(grep -v "$expected_header" $random_file | shuf -n 1)

# check if random_line is in all.csv file
if grep -q "$random_line" all.csv; then
  echo -n "OK"
  exit
else
  echo -n "NO"
fi
```

#### 2. Print some files to check headers
`cat polldayregistrations_enregistjourduscrutin10001.csv`  
<details>
  <summary>Spoiler text output</summary>

```text
Electoral District Number/Numéro de circonscription,Electoral District Name/Nom de circonscription,Polling Station Number/Numéro du bureau de scrutin,Polling Station Name/Nom du bureau de scrutin,Additions/Ajouts,Corrections/Corrections,Deletions/Suppressions,Electors/Électeurs
10001,"Avalon/Avalon","1","Freshwater",10,8,1,106
10001,"Avalon/Avalon","2","Victoria",9,0,4,330
10001,"Avalon/Avalon","3","Victoria",13,1,4,454
10001,"Avalon/Avalon","4","Victoria",17,0,0,342
10001,"Avalon/Avalon","5","Victoria",13,3,5,358
10001,"Avalon/Avalon","6","Carbonear",6,0,8,440
10001,"Avalon/Avalon","7","Carbonear",4,1,6,302
10001,"Avalon/Avalon","8","Carbonear",9,0,1,295
10001,"Avalon/Avalon","9","Carbonear",2,2,4,364
10001,"Avalon/Avalon","10","Carbonear",4,0,3,263
10001,"Avalon/Avalon","11","Carbonear",22,5,4,367
10001,"Avalon/Avalon","12","Harbour Grace--Bristol's Hope",17,1,3,433
10001,"Avalon/Avalon","13","Carbonear",1,1,4,329
10001,"Avalon/Avalon","14","Carbonear",11,0,6,432
10001,"Avalon/Avalon","15","Carbonear",1,2,2,414
10001,"Avalon/Avalon","16","Carbonear",17,0,8,342
10001,"Avalon/Avalon","17","Carbonear",1,0,4,206
10001,"Avalon/Avalon","18","Carbonear",1,0,6,225
10001,"Avalon/Avalon","19","Harbour Grace",5,1,3,490
10001,"Avalon/Avalon","20","Harbour Grace",7,0,3,368
10001,"Avalon/Avalon","21","Riverhead--Harbour Grace",10,1,1,231
10001,"Avalon/Avalon","22","Harbour Grace South",10,0,14,469
10001,"Avalon/Avalon","23","Harbour Grace",5,0,5,265
10001,"Avalon/Avalon","24","Harbour Grace",3,1,3,293
10001,"Avalon/Avalon","25","Harbour Grace",2,0,3,292
10001,"Avalon/Avalon","26","Bryant's Cove",27,0,4,277
10001,"Avalon/Avalon","27","Upper Island Cove",12,2,1,435
10001,"Avalon/Avalon","28","Upper Island Cove",6,0,4,305
10001,"Avalon/Avalon","29","Upper Island Cove",3,0,2,234
10001,"Avalon/Avalon","30","Upper Island Cove",10,0,7,447
10001,"Avalon/Avalon","31","Spaniard's Bay",7,3,6,379
10001,"Avalon/Avalon","32","Spaniard's Bay",6,0,7,324
10001,"Avalon/Avalon","33","Spaniard's Bay",8,0,2,438
10001,"Avalon/Avalon","34","Tilton",23,0,0,296
10001,"Avalon/Avalon","35","Spaniard's Bay",6,0,5,370
10001,"Avalon/Avalon","36","Spaniard's Bay",3,0,4,264
10001,"Avalon/Avalon","37","Butlerville",3,0,4,263
10001,"Avalon/Avalon","38","Shearstown",21,1,6,607
10001,"Avalon/Avalon","39","Shearstown",4,2,1,322
10001,"Avalon/Avalon","40","Bay Roberts",15,4,10,570
10001,"Avalon/Avalon","41","Bay Roberts",10,4,11,652
10001,"Avalon/Avalon","42","Bay Roberts",3,0,10,506
10001,"Avalon/Avalon","43","Bay Roberts",3,0,4,519
10001,"Avalon/Avalon","44","Bay Roberts",7,0,5,610
10001,"Avalon/Avalon","45","Bay Roberts",17,2,7,597
10001,"Avalon/Avalon","46","Blow Me Down--Port de Grave",10,3,2,411
10001,"Avalon/Avalon","47","Bareneed--The Dock--Black Duck Pond",1,1,6,290
10001,"Avalon/Avalon","48","North River--Otterbury--Birch Hills",4,1,3,232
10001,"Avalon/Avalon","49","North River",17,1,4,404
10001,"Avalon/Avalon","50","Makinsons",32,1,5,537
10001,"Avalon/Avalon","51","Clarke's Beach",12,2,1,459
10001,"Avalon/Avalon","52","Clarke's Beach",5,0,4,393
10001,"Avalon/Avalon","53","Clarke's Beach",7,0,5,352
10001,"Avalon/Avalon","54","South River",16,0,3,255
10001,"Avalon/Avalon","55","South River",13,4,3,312
10001,"Avalon/Avalon","56","Cupids",1,0,3,326
10001,"Avalon/Avalon","57","Cupids",10,0,9,249
10001,"Avalon/Avalon","58","Brigus",11,0,6,410
10001,"Avalon/Avalon","59","Brigus",11,0,4,298
10001,"Avalon/Avalon","60","Marysvale",1,0,2,331
10001,"Avalon/Avalon","61","Marysvale",7,1,5,224
10001,"Avalon/Avalon","62","Colliers",5,2,5,267
10001,"Avalon/Avalon","63","Colliers",6,4,3,271
10001,"Avalon/Avalon","64","Conception Harbour",9,0,0,196
10001,"Avalon/Avalon","65","Conception Harbour",12,1,1,334
10001,"Avalon/Avalon","66","Avondale",7,3,2,213
10001,"Avalon/Avalon","67","Avondale",6,1,5,319
10001,"Avalon/Avalon","68","Harbour Main-Chapel's Cove-Lakeview",6,2,7,213
10001,"Avalon/Avalon","69","Harbour Main-Chapel's Cove-Lakeview",2,0,1,245
10001,"Avalon/Avalon","70","Harbour Main-Chapel's Cove-Lakeview",9,1,1,451
10001,"Avalon/Avalon","71","Holyrood",18,0,4,496
10001,"Avalon/Avalon","72","Holyrood",12,2,5,482
10001,"Avalon/Avalon","73","Holyrood",6,1,1,332
10001,"Avalon/Avalon","74","Holyrood",5,0,4,291
10001,"Avalon/Avalon","75","Holyrood",5,3,3,470
10001,"Avalon/Avalon","76","Conception Bay South",4,1,2,489
10001,"Avalon/Avalon","77","Conception Bay South",10,0,3,635
10001,"Avalon/Avalon","78","Conception Bay South",10,0,1,490
10001,"Avalon/Avalon","79","Conception Bay South",6,1,9,664
10001,"Avalon/Avalon","80","Conception Bay South",14,0,8,662
10001,"Avalon/Avalon","81","Conception Bay South",4,0,2,490
10001,"Avalon/Avalon","82","Conception Bay South",11,0,0,521
10001,"Avalon/Avalon","83","Conception Bay South",8,0,5,446
10001,"Avalon/Avalon","84","Conception Bay South",12,0,4,544
10001,"Avalon/Avalon","85","Conception Bay South",9,0,3,477
10001,"Avalon/Avalon","86","Conception Bay South",12,2,8,612
10001,"Avalon/Avalon","87","Conception Bay South",3,1,5,511
10001,"Avalon/Avalon","88","Conception Bay South",9,2,7,528
10001,"Avalon/Avalon","89","Conception Bay South",3,1,4,388
10001,"Avalon/Avalon","90","Conception Bay South",5,0,5,448
10001,"Avalon/Avalon","91","Conception Bay South",8,1,7,481
10001,"Avalon/Avalon","92","Conception Bay South",2,1,10,424
10001,"Avalon/Avalon","93","Conception Bay South",6,1,7,527
10001,"Avalon/Avalon","94","Conception Bay South",9,1,5,498
10001,"Avalon/Avalon","95","Conception Bay South",5,0,4,602
10001,"Avalon/Avalon","96","Conception Bay South",5,5,5,592
10001,"Avalon/Avalon","97","Conception Bay South",5,3,4,524
10001,"Avalon/Avalon","98","Conception Bay South",5,3,10,501
10001,"Avalon/Avalon","99","Conception Bay South",8,2,2,578
10001,"Avalon/Avalon","100","Conception Bay South",6,3,7,352
10001,"Avalon/Avalon","101","Conception Bay South",7,2,5,510
10001,"Avalon/Avalon","102","Conception Bay South",9,0,11,586
10001,"Avalon/Avalon","103","Conception Bay South",10,0,7,586
10001,"Avalon/Avalon","104","Conception Bay South",6,1,4,436
10001,"Avalon/Avalon","105","Conception Bay South",10,1,5,648
10001,"Avalon/Avalon","106","Conception Bay South",5,0,5,374
10001,"Avalon/Avalon","107","Conception Bay South",5,2,0,407
10001,"Avalon/Avalon","108","Conception Bay South",8,0,7,505
10001,"Avalon/Avalon","109","Conception Bay South",2,0,1,379
10001,"Avalon/Avalon","110","Conception Bay South",6,0,5,371
10001,"Avalon/Avalon","111","Conception Bay South",8,1,1,554
10001,"Avalon/Avalon","112","Conception Bay South",10,0,1,514
10001,"Avalon/Avalon","113","Conception Bay South",11,0,5,514
10001,"Avalon/Avalon","114","Conception Bay South",7,2,4,501
10001,"Avalon/Avalon","115","Conception Bay South",8,0,7,404
10001,"Avalon/Avalon","116","Conception Bay South",7,1,4,455
10001,"Avalon/Avalon","117","Conception Bay South",8,1,3,507
10001,"Avalon/Avalon","118","Conception Bay South",6,1,6,463
10001,"Avalon/Avalon","119","Paradise",6,0,6,466
10001,"Avalon/Avalon","120","Paradise",3,0,8,409
10001,"Avalon/Avalon","121","Paradise",5,1,8,368
10001,"Avalon/Avalon","122","Paradise",4,2,7,428
10001,"Avalon/Avalon","123","Paradise",3,1,5,376
10001,"Avalon/Avalon","124","Paradise",15,0,8,710
10001,"Avalon/Avalon","125","Paradise",1,1,6,398
10001,"Avalon/Avalon","126","Paradise",3,0,5,459
10001,"Avalon/Avalon","127","Paradise",7,0,6,467
10001,"Avalon/Avalon","128","Paradise",16,0,5,493
10001,"Avalon/Avalon","129","Paradise",0,0,11,599
10001,"Avalon/Avalon","130","Paradise",7,1,9,592
10001,"Avalon/Avalon","131","Paradise",5,0,6,491
10001,"Avalon/Avalon","132","Paradise",6,1,5,423
10001,"Avalon/Avalon","133","Paradise",10,1,12,605
10001,"Avalon/Avalon","134","Paradise",5,0,7,512
10001,"Avalon/Avalon","135","Paradise",14,0,6,497
10001,"Avalon/Avalon","136","Paradise",15,1,9,399
10001,"Avalon/Avalon","137","Paradise",10,3,5,574
10001,"Avalon/Avalon","138","Paradise",16,0,2,462
10001,"Avalon/Avalon","139","Paradise",3,0,1,316
10001,"Avalon/Avalon","140","Paradise",7,0,5,497
10001,"Avalon/Avalon","141","Paradise",5,2,7,398
10001,"Avalon/Avalon","142","Paradise",20,1,13,652
10001,"Avalon/Avalon","143","Markland",25,0,7,163
10001,"Avalon/Avalon","144","Long Harbour",0,0,0,239
10001,"Avalon/Avalon","145","Ship Harbour",5,0,0,85
10001,"Avalon/Avalon","146","Fox Harbour",3,0,2,211
10001,"Avalon/Avalon","147","Dunville",27,0,4,299
10001,"Avalon/Avalon","148","Dunville",6,1,6,489
10001,"Avalon/Avalon","149","Freshwater",5,0,2,259
10001,"Avalon/Avalon","150","Freshwater",8,1,5,185
10001,"Avalon/Avalon","151","Jerseyside",5,0,3,267
10001,"Avalon/Avalon","152","Placentia",13,1,24,438
10001,"Avalon/Avalon","153","Placentia",5,1,5,359
10001,"Avalon/Avalon","154","Southeast Placentia",38,3,6,614
10001,"Avalon/Avalon","155","St. Bride's",12,0,14,333
10001,"Avalon/Avalon","156","Point Lance",6,0,1,82
10001,"Avalon/Avalon","157","Branch",21,0,3,178
10001,"Avalon/Avalon","158","Brigus Junction",27,1,19,252
10001,"Avalon/Avalon","159","St. Catherine's",3,0,5,273
10001,"Avalon/Avalon","160","Mount Carmel",19,1,7,304
10001,"Avalon/Avalon","161","Colinet",4,0,0,145
10001,"Avalon/Avalon","162","North Harbour",22,0,1,64
10001,"Avalon/Avalon","163","Riverhead",28,0,2,158
10001,"Avalon/Avalon","164","St. Joseph's",13,0,1,162
10001,"Avalon/Avalon","165","Admirals Beach",3,0,1,109
10001,"Avalon/Avalon","166","St. Mary's",5,0,40,424
10001,"Avalon/Avalon","167","Gaskiers-Point La Haye",4,1,0,143
10001,"Avalon/Avalon","168","St. Vincent's-St. Stephen's-Peter's River",1,0,5,275
10001,"Avalon/Avalon","169","St. Shotts",0,0,1,60
10001,"Avalon/Avalon","170","Mobile",27,0,3,270
10001,"Avalon/Avalon","171","Tors Cove",6,1,15,348
10001,"Avalon/Avalon","172","Bauline East",60,1,3,207
10001,"Avalon/Avalon","173","Cape Broyle",5,0,3,223
10001,"Avalon/Avalon","174","Cape Broyle",11,2,15,443
10001,"Avalon/Avalon","175","Calvert",9,0,1,185
10001,"Avalon/Avalon","176","Ferryland",9,2,2,392
10001,"Avalon/Avalon","177","Fermeuse--Port Kirwan",10,1,2,332
10001,"Avalon/Avalon","178","Renews",3,0,12,286
10001,"Avalon/Avalon","179","Portugal Cove South",1,0,0,98
10001,"Avalon/Avalon","180","Trepassey",9,0,4,241
10001,"Avalon/Avalon","181","Trepassey",0,0,5,216
10001,"Avalon/Avalon","400","Mobile poll/Bureau itinérant",5,0,0,74
10001,"Avalon/Avalon","500","Mobile poll/Bureau itinérant",0,0,6,71
10001,"Avalon/Avalon","501","Mobile poll/Bureau itinérant",0,0,4,81
10001,"Avalon/Avalon","502","Mobile poll/Bureau itinérant",0,0,0,14
10001,"Avalon/Avalon","503","Mobile poll/Bureau itinérant",0,0,0,15
10001,"Avalon/Avalon","504","Mobile poll/Bureau itinérant",0,0,0,14
10001,"Avalon/Avalon","505","Mobile poll/Bureau itinérant",0,0,0,12
10001,"Avalon/Avalon","506","Mobile poll/Bureau itinérant",0,0,0,13
10001,"Avalon/Avalon","507","Mobile poll/Bureau itinérant",0,0,0,16
10001,"Avalon/Avalon","508","Mobile poll/Bureau itinérant",1,0,0,14
10001,"Avalon/Avalon","509","Mobile poll/Bureau itinérant",0,0,0,12
10001,"Avalon/Avalon","510","Mobile poll/Bureau itinérant",Void/Supprimé
10001,"Avalon/Avalon","511","Mobile poll/Bureau itinérant",0,0,2,17
10001,"Avalon/Avalon","512","Mobile poll/Bureau itinérant",0,0,1,101
10001,"Avalon/Avalon","513","Mobile poll/Bureau itinérant",0,0,0,10
10001,"Avalon/Avalon","514","Mobile poll/Bureau itinérant",3,0,0,40
10001,"Avalon/Avalon","515","Mobile poll/Bureau itinérant",4,2,2,74
10001,"Avalon/Avalon","516","Mobile poll/Bureau itinérant",0,0,0,14
10001,"Avalon/Avalon","517","Mobile poll/Bureau itinérant",0,0,0,12
10001,"Avalon/Avalon","518","Mobile poll/Bureau itinérant",1,0,0,18
10001,"Avalon/Avalon","519","Mobile poll/Bureau itinérant",Void/Supprimé
10001,"Avalon/Avalon","520","Mobile poll/Bureau itinérant",0,0,1,35
10001,"Avalon/Avalon","521","Mobile poll/Bureau itinérant",0,0,0,30
10001,"Avalon/Avalon","522","Mobile poll/Bureau itinérant",0,0,0,27
10001,"Avalon/Avalon","523","Mobile poll/Bureau itinérant",0,0,0,7
10001,"Avalon/Avalon","524","Mobile poll/Bureau itinérant",0,0,0,9
10001,"Avalon/Avalon","525","Mobile poll/Bureau itinérant",7,1,3,51
10001,"Avalon/Avalon","526","Mobile poll/Bureau itinérant",0,0,2,94
```

</details>

`cat polldayregistrations_enregistjourduscrutin12010.csv`  

<details>
  <summary>Spoiler text output</summary>

```text
Electoral District Number/Numéro de circonscription,Electoral District Name/Nom de circonscription,Polling Station Number/Numéro du bureau de scrutin,Polling Station Name/Nom du bureau de scrutin,Additions/Ajouts,Corrections/Corrections,Deletions/Suppressions,Electors/Électeurs
12010,"Sydney--Victoria/Sydney--Victoria","1","St. Margaret Village",5,3,4,320
12010,"Sydney--Victoria/Sydney--Victoria","2","Neils Harbour",6,1,0,246
12010,"Sydney--Victoria/Sydney--Victoria","3","Cape North",14,3,8,478
12010,"Sydney--Victoria/Sydney--Victoria","4","White Point",5,3,1,138
12010,"Sydney--Victoria/Sydney--Victoria","5","Ingonish",8,2,6,521
12010,"Sydney--Victoria/Sydney--Victoria","6","Pleasant Bay",6,2,4,138
12010,"Sydney--Victoria/Sydney--Victoria","7","Ingonish Beach",5,0,4,325
12010,"Sydney--Victoria/Sydney--Victoria","8","Little River",7,1,3,306
12010,"Sydney--Victoria/Sydney--Victoria","9","Cape Breton",8,0,6,410
12010,"Sydney--Victoria/Sydney--Victoria","10","Cape Breton",1,0,4,371
12010,"Sydney--Victoria/Sydney--Victoria","11","Cape Breton",4,0,4,358
12010,"Sydney--Victoria/Sydney--Victoria","12","Cape Breton",2,2,4,420
12010,"Sydney--Victoria/Sydney--Victoria","13","Cape Breton",3,5,2,393
12010,"Sydney--Victoria/Sydney--Victoria","14","Cape Breton",2,2,2,338
12010,"Sydney--Victoria/Sydney--Victoria","15","Cape Breton",6,0,6,397
12010,"Sydney--Victoria/Sydney--Victoria","16","Cape Breton",4,1,4,399
12010,"Sydney--Victoria/Sydney--Victoria","17","Cape Breton",0,0,4,303
12010,"Sydney--Victoria/Sydney--Victoria","18","Cape Breton",6,0,3,428
12010,"Sydney--Victoria/Sydney--Victoria","19","Cape Breton",5,1,4,405
12010,"Sydney--Victoria/Sydney--Victoria","20","Cape Breton",3,0,2,357
12010,"Sydney--Victoria/Sydney--Victoria","21","Cape Breton",5,0,6,390
12010,"Sydney--Victoria/Sydney--Victoria","22","Cape Breton",0,0,2,304
12010,"Sydney--Victoria/Sydney--Victoria","23","Cape Breton",0,0,0,303
12010,"Sydney--Victoria/Sydney--Victoria","24","Alder Point",6,0,4,338
12010,"Sydney--Victoria/Sydney--Victoria","25","Little Bras d'Or South Side",4,1,2,291
12010,"Sydney--Victoria/Sydney--Victoria","26","Cape Breton",5,0,4,380
12010,"Sydney--Victoria/Sydney--Victoria","27","Cape Breton",6,1,2,345
12010,"Sydney--Victoria/Sydney--Victoria","28","Cape Breton",1,0,4,483
12010,"Sydney--Victoria/Sydney--Victoria","29","Cape Breton",4,1,1,368
12010,"Sydney--Victoria/Sydney--Victoria","30","Cape Breton",10,0,7,412
12010,"Sydney--Victoria/Sydney--Victoria","31","Point Aconi",1,0,2,314
12010,"Sydney--Victoria/Sydney--Victoria","32","Mill Creek",3,0,1,206
12010,"Sydney--Victoria/Sydney--Victoria","33","Little Bras d'Or",2,0,2,342
12010,"Sydney--Victoria/Sydney--Victoria","34","Cape Breton",2,1,0,293
12010,"Sydney--Victoria/Sydney--Victoria","35","New Victoria",3,2,5,299
12010,"Sydney--Victoria/Sydney--Victoria","36","Victoria Mines",3,3,5,295
12010,"Sydney--Victoria/Sydney--Victoria","37","Scotchtown",2,0,0,259
12010,"Sydney--Victoria/Sydney--Victoria","38","Scotchtown",1,0,2,365
12010,"Sydney--Victoria/Sydney--Victoria","39","Scotchtown",0,0,2,381
12010,"Sydney--Victoria/Sydney--Victoria","40","River Ryan",3,2,1,274
12010,"Sydney--Victoria/Sydney--Victoria","41","River Ryan",3,1,8,370
12010,"Sydney--Victoria/Sydney--Victoria","42","Florence",6,0,3,273
12010,"Sydney--Victoria/Sydney--Victoria","43","Florence",2,0,1,292
12010,"Sydney--Victoria/Sydney--Victoria","44","Florence",6,0,3,356
12010,"Sydney--Victoria/Sydney--Victoria","45","Bras d'Or",2,1,3,276
12010,"Sydney--Victoria/Sydney--Victoria","46","Florence",4,0,3,285
12010,"Sydney--Victoria/Sydney--Victoria","47","Englishtown",1,0,1,181
12010,"Sydney--Victoria/Sydney--Victoria","48","Cape Breton",8,1,2,413
12010,"Sydney--Victoria/Sydney--Victoria","49","Cape Breton",9,0,3,474
12010,"Sydney--Victoria/Sydney--Victoria","50","Cape Breton",2,0,3,422
12010,"Sydney--Victoria/Sydney--Victoria","51","Cape Breton",2,0,2,431
12010,"Sydney--Victoria/Sydney--Victoria","52","Cape Breton",5,0,2,406
12010,"Sydney--Victoria/Sydney--Victoria","53","Cape Breton",9,0,7,410
12010,"Sydney--Victoria/Sydney--Victoria","54","Cape Breton",6,0,3,436
12010,"Sydney--Victoria/Sydney--Victoria","55","Millville Boularderie",3,1,3,362
12010,"Sydney--Victoria/Sydney--Victoria","56","Groves Point",7,0,6,354
12010,"Sydney--Victoria/Sydney--Victoria","57","South Bar",7,1,2,269
12010,"Sydney--Victoria/Sydney--Victoria","58","South Bar",3,1,1,321
12010,"Sydney--Victoria/Sydney--Victoria","59","Cape Breton",3,0,5,289
12010,"Sydney--Victoria/Sydney--Victoria","60","Cape Breton",2,1,1,336
12010,"Sydney--Victoria/Sydney--Victoria","61","Cape Breton",2,0,2,347
12010,"Sydney--Victoria/Sydney--Victoria","62","Lingan Road",8,0,7,370
12010,"Sydney--Victoria/Sydney--Victoria","63","Big Bras d'Or",6,1,3,421
12010,"Sydney--Victoria/Sydney--Victoria","64","Cape Breton",0,0,1,326
12010,"Sydney--Victoria/Sydney--Victoria","65","Cape Breton",3,0,5,373
12010,"Sydney--Victoria/Sydney--Victoria","66","Cape Breton",8,1,2,349
12010,"Sydney--Victoria/Sydney--Victoria","67","Cape Breton",6,3,10,511
12010,"Sydney--Victoria/Sydney--Victoria","68","Cape Breton",3,1,2,350
12010,"Sydney--Victoria/Sydney--Victoria","69","Cape Breton",5,0,3,368
12010,"Sydney--Victoria/Sydney--Victoria","70","Cape Breton",6,0,3,357
12010,"Sydney--Victoria/Sydney--Victoria","71","Cape Breton",4,0,2,338
12010,"Sydney--Victoria/Sydney--Victoria","72","Cape Breton",3,1,3,313
12010,"Sydney--Victoria/Sydney--Victoria","73","Cape Breton",2,0,1,325
12010,"Sydney--Victoria/Sydney--Victoria","74","Cape Breton",7,0,4,354
12010,"Sydney--Victoria/Sydney--Victoria","75","Cape Breton",2,0,7,372
12010,"Sydney--Victoria/Sydney--Victoria","76","Cape Breton",5,0,2,323
12010,"Sydney--Victoria/Sydney--Victoria","77","Cape Breton",3,1,7,317
12010,"Sydney--Victoria/Sydney--Victoria","78","Cape Breton",2,1,5,337
12010,"Sydney--Victoria/Sydney--Victoria","79","Cape Breton",0,0,4,313
12010,"Sydney--Victoria/Sydney--Victoria","80","Cape Breton",3,0,0,324
12010,"Sydney--Victoria/Sydney--Victoria","81","Cape Breton",0,0,6,342
12010,"Sydney--Victoria/Sydney--Victoria","82","Cape Breton",0,0,3,358
12010,"Sydney--Victoria/Sydney--Victoria","83","Cape Breton",0,0,4,357
12010,"Sydney--Victoria/Sydney--Victoria","84","Cape Breton",1,0,7,341
12010,"Sydney--Victoria/Sydney--Victoria","85","Cape Breton",2,0,2,326
12010,"Sydney--Victoria/Sydney--Victoria","86","Cape Breton",4,0,9,457
12010,"Sydney--Victoria/Sydney--Victoria","87","Upper North Sydney",3,0,10,399
12010,"Sydney--Victoria/Sydney--Victoria","88","Edwardsville",0,0,1,289
12010,"Sydney--Victoria/Sydney--Victoria","89","Westmount",1,1,1,409
12010,"Sydney--Victoria/Sydney--Victoria","90","Westmount",1,0,2,376
12010,"Sydney--Victoria/Sydney--Victoria","91","Westmount",11,1,0,381
12010,"Sydney--Victoria/Sydney--Victoria","92","Cape Breton",0,0,13,421
12010,"Sydney--Victoria/Sydney--Victoria","93","Cape Breton",4,1,6,460
12010,"Sydney--Victoria/Sydney--Victoria","94","Cape Breton",6,4,4,357
12010,"Sydney--Victoria/Sydney--Victoria","95","Cape Breton",7,1,4,339
12010,"Sydney--Victoria/Sydney--Victoria","96","Cape Breton",3,0,4,367
12010,"Sydney--Victoria/Sydney--Victoria","97","Cape Breton",4,0,4,291
12010,"Sydney--Victoria/Sydney--Victoria","98","Cape Breton",0,0,7,339
12010,"Sydney--Victoria/Sydney--Victoria","99","Cape Breton",0,0,9,352
12010,"Sydney--Victoria/Sydney--Victoria","100","Cape Breton",3,0,5,365
12010,"Sydney--Victoria/Sydney--Victoria","101","Cape Breton",0,0,2,298
12010,"Sydney--Victoria/Sydney--Victoria","102","Cape Breton",4,0,5,284
12010,"Sydney--Victoria/Sydney--Victoria","103","Cape Breton",1,0,2,403
12010,"Sydney--Victoria/Sydney--Victoria","104","Cape Breton",1,0,4,275
12010,"Sydney--Victoria/Sydney--Victoria","105","Cape Breton",2,0,3,275
12010,"Sydney--Victoria/Sydney--Victoria","106","Cape Breton",3,0,8,343
12010,"Sydney--Victoria/Sydney--Victoria","107","Cape Breton",4,0,2,340
12010,"Sydney--Victoria/Sydney--Victoria","108","Cape Breton",1,0,1,348
12010,"Sydney--Victoria/Sydney--Victoria","109","Cape Breton",4,2,2,311
12010,"Sydney--Victoria/Sydney--Victoria","110","Cape Breton",5,0,2,295
12010,"Sydney--Victoria/Sydney--Victoria","111","Hillside Boularderie",3,0,2,380
12010,"Sydney--Victoria/Sydney--Victoria","112","Cape Breton",0,1,3,282
12010,"Sydney--Victoria/Sydney--Victoria","113","Cape Breton",0,1,8,332
12010,"Sydney--Victoria/Sydney--Victoria","114","Cape Breton",0,0,9,379
12010,"Sydney--Victoria/Sydney--Victoria","115","Cape Breton",0,0,3,399
12010,"Sydney--Victoria/Sydney--Victoria","116","Cape Breton",0,0,3,361
12010,"Sydney--Victoria/Sydney--Victoria","117","Cape Breton",1,0,5,350
12010,"Sydney--Victoria/Sydney--Victoria","118","Georges River",2,0,5,308
12010,"Sydney--Victoria/Sydney--Victoria","119","Scotch Lake",7,1,3,338
12010,"Sydney--Victoria/Sydney--Victoria","120","Cape Breton",15,2,16,658
12010,"Sydney--Victoria/Sydney--Victoria","121","Cape Breton",3,0,10,323
12010,"Sydney--Victoria/Sydney--Victoria","122","Membertou 28B",6,2,4,369
12010,"Sydney--Victoria/Sydney--Victoria","123","Cape Breton",6,2,6,411
12010,"Sydney--Victoria/Sydney--Victoria","124","Membertou 28B",10,1,3,574
12010,"Sydney--Victoria/Sydney--Victoria","125","Cape Breton",4,0,10,369
12010,"Sydney--Victoria/Sydney--Victoria","126","Cape Breton",2,6,7,322
12010,"Sydney--Victoria/Sydney--Victoria","127","Cape Breton",3,1,5,364
12010,"Sydney--Victoria/Sydney--Victoria","128","Cape Breton",3,0,9,351
12010,"Sydney--Victoria/Sydney--Victoria","129","Cape Breton",1,1,2,503
12010,"Sydney--Victoria/Sydney--Victoria","130","Cape Breton",3,1,4,434
12010,"Sydney--Victoria/Sydney--Victoria","131","Cape Breton",5,0,4,478
12010,"Sydney--Victoria/Sydney--Victoria","132","Cape Breton",0,0,1,295
12010,"Sydney--Victoria/Sydney--Victoria","133","Cape Breton",4,0,6,402
12010,"Sydney--Victoria/Sydney--Victoria","134","Cape Breton",1,0,2,370
12010,"Sydney--Victoria/Sydney--Victoria","135","Cape Breton",1,0,3,438
12010,"Sydney--Victoria/Sydney--Victoria","136","Cape Breton",2,0,6,419
12010,"Sydney--Victoria/Sydney--Victoria","137","Leitches Creek",6,0,4,298
12010,"Sydney--Victoria/Sydney--Victoria","138","Westmount",7,0,1,315
12010,"Sydney--Victoria/Sydney--Victoria","139","Coxheath",3,0,1,232
12010,"Sydney--Victoria/Sydney--Victoria","140","Coxheath",4,0,12,373
12010,"Sydney--Victoria/Sydney--Victoria","141","Coxheath",0,4,3,352
12010,"Sydney--Victoria/Sydney--Victoria","142","Coxheath",1,1,4,348
12010,"Sydney--Victoria/Sydney--Victoria","143","Coxheath",4,0,7,446
12010,"Sydney--Victoria/Sydney--Victoria","144","Coxheath",4,1,2,342
12010,"Sydney--Victoria/Sydney--Victoria","145","Blacketts Lake",1,0,1,255
12010,"Sydney--Victoria/Sydney--Victoria","146","South Haven",14,1,2,358
12010,"Sydney--Victoria/Sydney--Victoria","147","Balls Creek",2,1,2,385
12010,"Sydney--Victoria/Sydney--Victoria","148","Frenchvale",5,0,2,378
12010,"Sydney--Victoria/Sydney--Victoria","149","Baddeck",15,9,17,502
12010,"Sydney--Victoria/Sydney--Victoria","150","Boularderie",7,4,4,341
12010,"Sydney--Victoria/Sydney--Victoria","151","Boisdale",2,1,2,244
12010,"Sydney--Victoria/Sydney--Victoria","152","Middle River",3,14,8,378
12010,"Sydney--Victoria/Sydney--Victoria","153","Baddeck",18,2,7,405
12010,"Sydney--Victoria/Sydney--Victoria","154","Baddeck",4,5,6,313
12010,"Sydney--Victoria/Sydney--Victoria","155","Northside East Bay",4,2,6,464
12010,"Sydney--Victoria/Sydney--Victoria","156","Eskasoni 3",5,0,3,479
12010,"Sydney--Victoria/Sydney--Victoria","157","Eskasoni 3",9,2,1,477
12010,"Sydney--Victoria/Sydney--Victoria","158","Eskasoni 3",12,0,2,415
12010,"Sydney--Victoria/Sydney--Victoria","159","Eskasoni 3",15,0,5,569
12010,"Sydney--Victoria/Sydney--Victoria","160","Eskasoni 3",4,1,3,467
12010,"Sydney--Victoria/Sydney--Victoria","161","Wagmatcook 1",8,2,3,416
12010,"Sydney--Victoria/Sydney--Victoria","162","Grand Narrows",1,0,2,235
12010,"Sydney--Victoria/Sydney--Victoria","163","Iona",7,1,5,279
12010,"Sydney--Victoria/Sydney--Victoria","164","Little Narrows",0,0,1,135
12010,"Sydney--Victoria/Sydney--Victoria","400","Cape Breton",1,0,4,138
12010,"Sydney--Victoria/Sydney--Victoria","401","Cape Breton",0,0,1,89
12010,"Sydney--Victoria/Sydney--Victoria","402","Cape Breton",2,0,6,273
12010,"Sydney--Victoria/Sydney--Victoria","500","Mobile poll/Bureau itinérant",0,0,0,11
12010,"Sydney--Victoria/Sydney--Victoria","501","Mobile poll/Bureau itinérant",0,0,3,56
12010,"Sydney--Victoria/Sydney--Victoria","502","Mobile poll/Bureau itinérant",0,0,0,7
12010,"Sydney--Victoria/Sydney--Victoria","503","Mobile poll/Bureau itinérant",0,0,0,51
12010,"Sydney--Victoria/Sydney--Victoria","504","Mobile poll/Bureau itinérant",0,0,0,2
12010,"Sydney--Victoria/Sydney--Victoria","505","Mobile poll/Bureau itinérant",0,0,0,1
12010,"Sydney--Victoria/Sydney--Victoria","506","Mobile poll/Bureau itinérant",0,0,0,3
12010,"Sydney--Victoria/Sydney--Victoria","507","Mobile poll/Bureau itinérant",0,0,7,105
12010,"Sydney--Victoria/Sydney--Victoria","508","Mobile poll/Bureau itinérant",0,0,2,29
12010,"Sydney--Victoria/Sydney--Victoria","509","Mobile poll/Bureau itinérant",0,0,11,97
12010,"Sydney--Victoria/Sydney--Victoria","510","Mobile poll/Bureau itinérant",0,0,1,111
12010,"Sydney--Victoria/Sydney--Victoria","511","Mobile poll/Bureau itinérant",0,0,0,69
12010,"Sydney--Victoria/Sydney--Victoria","512","Mobile poll/Bureau itinérant",0,0,0,12
```

</details>


`cat polldayregistrations_enregistjourduscrutin62001.csv`  

<details>
  <summary>Spoiler text output</summary>

```text
Electoral District Number/Numéro de circonscription,Electoral District Name/Nom de circonscription,Polling Station Number/Numéro du bureau de scrutin,Polling Station Name/Nom du bureau de scrutin,Additions/Ajouts,Corrections/Corrections,Deletions/Suppressions,Electors/Électeurs
62001,"Nunavut/Nunavut","1","Grise Fiord",8,0,0,85
62001,"Nunavut/Nunavut","2","Resolute",7,5,3,122
62001,"Nunavut/Nunavut","3A","Pond Inlet",9,0,4,431
62001,"Nunavut/Nunavut","3B","Pond Inlet",15,3,13,460
62001,"Nunavut/Nunavut","4","Arctic Bay",22,5,9,438
62001,"Nunavut/Nunavut","5A","Taloyoak",7,0,4,259
62001,"Nunavut/Nunavut","5B","Taloyoak",1,0,3,245
62001,"Nunavut/Nunavut","6A","Clyde River",5,0,4,297
62001,"Nunavut/Nunavut","6B","Clyde River",6,1,3,274
62001,"Nunavut/Nunavut","7","Qikiqtarjuaq",14,1,3,355
62001,"Nunavut/Nunavut","8","Igloolik",37,2,13,908
62001,"Nunavut/Nunavut","9A","Kugluktuk",6,0,8,446
62001,"Nunavut/Nunavut","9B","Kugluktuk",10,0,10,463
62001,"Nunavut/Nunavut","10A","Cambridge Bay",17,0,21,429
62001,"Nunavut/Nunavut","10B","Cambridge Bay",15,1,18,437
62001,"Nunavut/Nunavut","10C","Cambridge Bay",10,0,18,413
62001,"Nunavut/Nunavut","11A","Gjoa Haven",17,1,3,349
62001,"Nunavut/Nunavut","11B","Gjoa Haven",16,2,7,382
62001,"Nunavut/Nunavut","12","Kugaaruk",0,0,2,519
62001,"Nunavut/Nunavut","13","Hall Beach",13,0,7,413
62001,"Nunavut/Nunavut","14","Repulse Bay",7,0,5,513
62001,"Nunavut/Nunavut","15","Cape Dorset",22,3,15,708
62001,"Nunavut/Nunavut","16","Pangnirtung",30,1,17,773
62001,"Nunavut/Nunavut","17A","Iqaluit",27,2,32,513
62001,"Nunavut/Nunavut","17B","Iqaluit",33,1,49,507
62001,"Nunavut/Nunavut","17C","Iqaluit",43,1,58,504
62001,"Nunavut/Nunavut","17D","Iqaluit",51,2,66,506
62001,"Nunavut/Nunavut","17E","Iqaluit",32,2,36,538
62001,"Nunavut/Nunavut","17F","Iqaluit",26,4,41,480
62001,"Nunavut/Nunavut","17G","Iqaluit",40,1,46,505
62001,"Nunavut/Nunavut","17H","Iqaluit",35,5,37,545
62001,"Nunavut/Nunavut","17I","Iqaluit",50,3,52,503
62001,"Nunavut/Nunavut","17J","Iqaluit",22,4,33,519
62001,"Nunavut/Nunavut","17K","Iqaluit",34,1,52,496
62001,"Nunavut/Nunavut","18","Kimmirut",22,1,5,264
62001,"Nunavut/Nunavut","19","Baker Lake",34,2,16,1105
62001,"Nunavut/Nunavut","20","Chesterfield Inlet",12,1,7,214
62001,"Nunavut/Nunavut","21A","Rankin Inlet",15,0,22,451
62001,"Nunavut/Nunavut","21B","Rankin Inlet",21,1,23,478
62001,"Nunavut/Nunavut","21C","Rankin Inlet",12,0,15,493
62001,"Nunavut/Nunavut","21D","Rankin Inlet",14,3,22,479
62001,"Nunavut/Nunavut","22","Coral Harbour",15,1,3,462
62001,"Nunavut/Nunavut","23","Whale Cove",8,0,3,255
62001,"Nunavut/Nunavut","24A","Arviat",16,3,13,434
62001,"Nunavut/Nunavut","24B","Arviat",13,5,6,464
62001,"Nunavut/Nunavut","24C","Arviat",18,2,13,469
62001,"Nunavut/Nunavut","25A","Sanikiluaq",5,0,3,246
62001,"Nunavut/Nunavut","25B","Sanikiluaq",9,0,2,259
```

</details>


#### 3. Common header
```text
Electoral District Number/Numéro de circonscription,Electoral District Name/Nom de circonscription,Polling Station Number/Numéro du bureau de scrutin,Polling Station Name/Nom du bureau de scrutin,Additions/Ajouts,Corrections/Corrections,Deletions/Suppressions,Electors/Électeurs
```


#### 4. My solution
```bash
head -1 /home/admin/polldayregistrations_enregistjourduscrutin10001.csv > /home/admin/all.csv && tail -n +2 -q /home/admin/polldayregistrations_enregistjourduscrutin*.csv >> /home/admin/all.csv
```

##### 4.1. Get 1st line with header from 1st file and put it into verification file
`head -1 /home/admin/polldayregistrations_enregistjourduscrutin10001.csv > /home/admin/all.csv`  

##### 4.2. Add all lines from the second one from all files according to the template into verification file
`tail -n +2 -q /home/admin/polldayregistrations_enregistjourduscrutin*.csv >> /home/admin/all.csv`  

##### 4.3. Validate the task
`bash /home/admin/agent/check.sh`  
```console
OK
```

---

#### Author's solution
```console
Clues
1. You can join all the files with cat polldayregistrations_*.csv > concat.csv and then take out all the header lines except the first one, or you can use Python code but there's an easier way using 'awk'
2. One solution: awk 'FNR==1 && NR!=1 {next} {print}' polldayregistrations_*.csv > all.csv
```
