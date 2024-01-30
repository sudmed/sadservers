# "Ivujivik": Parlez-vous Français?

**Description:** Given the CSV file `/home/admin/table_tableau11.csv`, find the Electoral District Name/Nom de circonscription that has the largest number of Rejected Ballots/Bulletins rejetés and also has a population of less than 100,000.  

The initial CSV file may be corrupted or invalid in a way that can be fixed without changing its data.  

Installed in the VM are: Python3, Go, sqlite3, [miller](https://miller.readthedocs.io/en/latest/) directly and PostgreSQL, MySQL in Docker images.  

Save the solution in the `/home/admin/mysolution`, with the name as it is in the file, for example: `echo "Trois-Rivières" > ~/mysolution`.  

**Test:** `md5sum /home/admin/mysolution` returns `e399d171f21839a65f8f8ab55ed1e1a1`.  

---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 88
drwxr-xr-x 6 admin admin  4096 Sep 26 01:58 .
drwxr-xr-x 3 root  root   4096 Sep 17 16:44 ..
drwx------ 3 admin admin  4096 Sep 20 15:52 .ansible
-rw------- 1 admin admin    57 Sep 20 15:58 .bash_history
-rw-r--r-- 1 admin admin   220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin  3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin  4096 Sep 20 15:56 .config
-rw-r--r-- 1 admin admin   807 Aug  4  2021 .profile
drwx------ 2 admin admin  4096 Sep 17 16:44 .ssh
drwxr-xr-x 2 admin root   4096 Sep 26 01:59 agent
-rw-r--r-- 1 admin admin 47553 Sep 26 01:58 table_tableau11.csv
```

`cat ./agent/check.sh`  
```console
#!/usr/bin/bash
res=$(md5sum /home/admin/mysolution |awk '{print $1}')
res=$(echo $res|tr -d '\r')

if [[ "$res" = "e399d171f21839a65f8f8ab55ed1e1a1" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`head table_tableau11.csv`  
```console
Province,Electoral District Name/Nom de circonscription,Electoral District Number/Numéro de circonscription,Population,Electors/Électeurs,Polling Stations/Bureaux de scrutin,Valid Ballots/Bulletins valides,Percentage of Valid Ballots /Pourcentage des bulletins valides,Rejected Ballots/Bulletins rejetés,Percentage of Rejected Ballots /Pourcentage des bulletins rejetés,Total Ballots Cast/Total des bulletins déposés,Percentage of Voter Turnout/Pourcentage de la participation électorale,Elected Candidate/Candidat élu
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Avalon",10001,81540,68487,220,42086,99.6,162,.4,42248,61.7,"McDonald, Ken Liberal/Libéral"
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Bonavista--Burin--Trinity",10002,76704,62462,260,35092,99.5,173,.5,35265,56.5,"Foote, Judy M. Liberal/Libéral"
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Coast of Bays--Central--Notre Dame",10003,78092,64226,233,35448,99.6,145,.4,35593,55.4,"Simms, Scott Liberal/Libéral"
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Labrador",10004,26728,20045,84,12373,99.6,53,.4,12426,62,"Jones, Yvonne Liberal/Libéral"
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Long Range Mountains",10005,87592,71918,253,41824,99.7,108,.3,41932,58.3,"Hutchings, Gudie Liberal/Libéral"
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's East/St. John's-Est",10006,81936,66304,186,44880,99.8,111,.2,44991,67.9,"Whalen, Nick Liberal/Libéral"
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's South--Mount Pearl/St. John's-Sud--Mount Pearl",10007,81944,67596,185,44801,99.7,133,.3,44934,66.5,"O'Regan, Seamus Liberal/Libéral"
"Prince Edward Island/Île-du-Prince-Édouard","Cardigan",11001,36005,28889,90,22485,99.6,96,.4,22581,78.2,"MacAulay, Lawrence Liberal/Libéral"
"Prince Edward Island/Île-du-Prince-Édouard","Charlottetown",11002,34562,28129,82,21165,99.5,99,.5,21264,75.6,"Casey, Sean Liberal/Libéral"
```


#### 2. Use [mlr](https://miller.readthedocs.io/en/latest/10min/) utility (miller)
##### Miller is a command-line tool for querying, shaping, and reformatting data files in various formats including CSV, TSV, JSON, and JSON Lines

##### 2.1. Print our csv file
`mlr --csv cat /home/admin/table_tableau11.csv`  
```console
Province,Electoral District Name/Nom de circonscription,Electoral District Number/Numéro de circonscription,Population,Electors/Électeurs,Polling Stations/Bureaux de scrutin,Valid Ballots/Bulletins valides,Percentage of Valid Ballots /Pourcentage des bulletins valides,Rejected Ballots/Bulletins rejetés,Percentage of Rejected Ballots /Pourcentage des bulletins rejetés,Total Ballots Cast/Total des bulletins déposés,Percentage of Voter Turnout/Pourcentage de la participation électorale,Elected Candidate/Candidat élu
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Avalon,10001,81540,68487,220,42086,99.6,162,.4,42248,61.7,"McDonald, Ken Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Bonavista--Burin--Trinity,10002,76704,62462,260,35092,99.5,173,.5,35265,56.5,"Foote, Judy M. Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Coast of Bays--Central--Notre Dame,10003,78092,64226,233,35448,99.6,145,.4,35593,55.4,"Simms, Scott Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Labrador,10004,26728,20045,84,12373,99.6,53,.4,12426,62,"Jones, Yvonne Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Long Range Mountains,10005,87592,71918,253,41824,99.7,108,.3,41932,58.3,"Hutchings, Gudie Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,St. John's East/St. John's-Est,10006,81936,66304,186,44880,99.8,111,.2,44991,67.9,"Whalen, Nick Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,St. John's South--Mount Pearl/St. John's-Sud--Mount Pearl,10007,81944,67596,185,44801,99.7,133,.3,44934,66.5,"O'Regan, Seamus Liberal/Libéral"
Prince Edward Island/Île-du-Prince-Édouard,Cardigan,11001,36005,28889,90,22485,99.6,96,.4,22581,78.2,"MacAulay, Lawrence Liberal/Libéral"
Prince Edward Island/Île-du-Prince-Édouard,Charlottetown,11002,34562,28129,82,21165,99.5,99,.5,21264,75.6,"Casey, Sean Liberal/Libéral"
Prince Edward Island/Île-du-Prince-Édouard,Egmont,11003,34598,27858,86,21362,99.6,87,.4,21449,77,"Morrissey, Bobby Liberal/Libéral"
Prince Edward Island/Île-du-Prince-Édouard,Malpeque,11004,35039,28629,81,22472,99.5,102,.5,22574,78.9,"Easter, Wayne Liberal/Libéral"
```

##### 2.2. Filter file for population of less than 100,000
`mlr --icsv --ocsv filter '$Population < 100000' table_tableau11.csv > population.csv`  
`head population.csv`  
```console
Province,Electoral District Name/Nom de circonscription,Electoral District Number/Numéro de circonscription,Population,Electors/Électeurs,Polling Stations/Bureaux de scrutin,Valid Ballots/Bulletins valides,Percentage of Valid Ballots /Pourcentage des bulletins valides,Rejected Ballots/Bulletins rejetés,Percentage of Rejected Ballots /Pourcentage des bulletins rejetés,Total Ballots Cast/Total des bulletins déposés,Percentage of Voter Turnout/Pourcentage de la participation électorale,Elected Candidate/Candidat élu
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Avalon,10001,81540,68487,220,42086,99.6,162,.4,42248,61.7,"McDonald, Ken Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Bonavista--Burin--Trinity,10002,76704,62462,260,35092,99.5,173,.5,35265,56.5,"Foote, Judy M. Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Coast of Bays--Central--Notre Dame,10003,78092,64226,233,35448,99.6,145,.4,35593,55.4,"Simms, Scott Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Labrador,10004,26728,20045,84,12373,99.6,53,.4,12426,62,"Jones, Yvonne Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Long Range Mountains,10005,87592,71918,253,41824,99.7,108,.3,41932,58.3,"Hutchings, Gudie Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,St. John's East/St. John's-Est,10006,81936,66304,186,44880,99.8,111,.2,44991,67.9,"Whalen, Nick Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,St. John's South--Mount Pearl/St. John's-Sud--Mount Pearl,10007,81944,67596,185,44801,99.7,133,.3,44934,66.5,"O'Regan, Seamus Liberal/Libéral"
Prince Edward Island/Île-du-Prince-Édouard,Cardigan,11001,36005,28889,90,22485,99.6,96,.4,22581,78.2,"MacAulay, Lawrence Liberal/Libéral"
Prince Edward Island/Île-du-Prince-Édouard,Charlottetown,11002,34562,28129,82,21165,99.5,99,.5,21264,75.6,"Casey, Sean Liberal/Libéral"
```


##### 2.3. Sort the previously filtered file in numerically ascending order
`mlr --icsv --ocsv sort -n 'Rejected Ballots/Bulletins rejetés' population.csv > sorted.csv`  
`head sorted.csv`  
```console
Province,Electoral District Name/Nom de circonscription,Electoral District Number/Numéro de circonscription,Population,Electors/Électeurs,Polling Stations/Bureaux de scrutin,Valid Ballots/Bulletins valides,Percentage of Valid Ballots /Pourcentage des bulletins valides,Rejected Ballots/Bulletins rejetés,Percentage of Rejected Ballots /Pourcentage des bulletins rejetés,Total Ballots Cast/Total des bulletins déposés,Percentage of Voter Turnout/Pourcentage de la participation électorale,Elected Candidate/Candidat élu
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Labrador,10004,26728,20045,84,12373,99.6,53,.4,12426,62,"Jones, Yvonne Liberal/Libéral"
Prince Edward Island/Île-du-Prince-Édouard,Egmont,11003,34598,27858,86,21362,99.6,87,.4,21449,77,"Morrissey, Bobby Liberal/Libéral"
Prince Edward Island/Île-du-Prince-Édouard,Cardigan,11001,36005,28889,90,22485,99.6,96,.4,22581,78.2,"MacAulay, Lawrence Liberal/Libéral"
Prince Edward Island/Île-du-Prince-Édouard,Charlottetown,11002,34562,28129,82,21165,99.5,99,.5,21264,75.6,"Casey, Sean Liberal/Libéral"
Prince Edward Island/Île-du-Prince-Édouard,Malpeque,11004,35039,28629,81,22472,99.5,102,.5,22574,78.9,"Easter, Wayne Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Long Range Mountains,10005,87592,71918,253,41824,99.7,108,.3,41932,58.3,"Hutchings, Gudie Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,St. John's East/St. John's-Est,10006,81936,66304,186,44880,99.8,111,.2,44991,67.9,"Whalen, Nick Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,St. John's South--Mount Pearl/St. John's-Sud--Mount Pearl,10007,81944,67596,185,44801,99.7,133,.3,44934,66.5,"O'Regan, Seamus Liberal/Libéral"
Newfoundland and Labrador/Terre-Neuve-et-Labrador,Coast of Bays--Central--Notre Dame,10003,78092,64226,233,35448,99.6,145,.4,35593,55.4,"Simms, Scott Liberal/Libéral"
```


##### 2.4. Cutting out and printing one table column
`mlr --icsv --ocsv cut -f 'Rejected Ballots/Bulletins rejetés' sorted.csv`  
```console
Rejected Ballots/Bulletins rejetés
53
87
96
99
102
108
111
133
145
162
173
178
180
181
188
201
202
205
220
226
233
236
241
248
256
259
271
274
311
320
336
348
395
409
416
493
601
609
645
679
716
745
769
777
800
820
844
846
881
899
909
941
941
958
996
1098
1214
1226
```


##### 2.5. Cutting out and printing two table columns
`mlr --icsv --ocsv cut -f 'Rejected Ballots/Bulletins rejetés','Electoral District Name/Nom de circonscription' sorted.csv`  
```console
Electoral District Name/Nom de circonscription,Rejected Ballots/Bulletins rejetés
Labrador,53
Egmont,87
Cardigan,96
Charlottetown,99
Malpeque,102
Long Range Mountains,108
St. John's East/St. John's-Est,111
St. John's South--Mount Pearl/St. John's-Sud--Mount Pearl,133
Coast of Bays--Central--Notre Dame,145
Avalon,162
Bonavista--Burin--Trinity,173
Cumberland--Colchester,178
Sackville--Preston--Chezzetcook,180
Halifax West/Halifax-Ouest,181
Fredericton,188
Dartmouth--Cole Harbour,201
Kings--Hants,202
Saint John--Rothesay,205
New Brunswick Southwest/Nouveau-Brunswick-Sud-Ouest,220
South Shore--St. Margarets,226
Central Nova/Nova-Centre,233
Sydney--Victoria,236
Fundy Royal,241
Tobique--Mactaquac,248
Miramichi--Grand Lake,256
Halifax,259
West Nova/Nova-Ouest,271
Cape Breton--Canso,274
Moncton--Riverview--Dieppe,311
Beauséjour,320
Acadie--Bathurst,336
Madawaska--Restigouche,348
Gaspésie--Les Îles-de-la-Madeleine,395
Saint-Laurent,409
Avignon--La Mitis--Matane--Matapédia,416
Rimouski-Neigette--Témiscouata--Les Basques,493
Argenteuil--La Petite-Nation,601
Abitibi--Baie-James--Nunavik--Eeyou,609
Manicouagan,645
Alfred-Pellan,679
Brome--Missisquoi,716
Chicoutimi--Le Fjord,745
Marc-Aurèle-Fortin,769
Montmagny--L'Islet--Kamouraska--Rivière-du-Loup,777
Pierre-Boucher--Les Patriotes--Verchères,800
Québec,820
Berthier--Maskinongé,844
Beauport--Côte-de-Beaupré--Île d'Orléans--Charlevoix,846
Montarville,881
Jonquière,899
Châteauguay--Lacolle,909
Beauport--Limoilou,941
Mégantic--L'Érable,941
Bécancour--Nicolet--Saurel,958
La Prairie,996
Drummond,1098
Saint-Hyacinthe--Bagot,1214
Montcalm,1226
```


#### 3. As we can see, answer is the last value
```console
Montcalm,1226
```


`echo Montcalm > ~/mysolution`  
`cat ~/mysolution`  
```console
Montcalm
```


#### 4. Validate the task
`md5sum /home/admin/mysolution`  
```console
e399d171f21839a65f8f8ab55ed1e1a1  /home/admin/mysolution
```

`cd /home/admin/agent && ./check.sh `  
```console
OK
```

---

### AUTHOR'S SOLUTION
#### I. Import csv to sqlite3
`sqlite3`  
```console
SQLite version 3.34.1 2021-01-20 14:10:07
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite> .import --csv /home/admin/table_tableau11.csv elect
/home/admin/table_tableau11.csv:101: expected 13 columns but found 12 - filling the rest with NULL
sqlite>
```

##### Error in line 101: there are fewer columns than necessary, apparently a comma is missing
`nano +101 table_tableau11.csv`  
```console
"Quebec/Québec""Saint-Léonard--Saint-Michel",24069,110649,76746,205,44531,98.5,689,1.5,45220,58.9,"Di Iorio, Nicola Liberal/Libéral"
```

#### II. Add missing comma between 1st and 2nd columns
`nano +101 table_tableau11.csv`  
```console
"Quebec/Québec","Saint-Léonard--Saint-Michel",24069,110649,76746,205,44531,98.5,689,1.5,45220,58.9,"Di Iorio, Nicola Liberal/Libéral"
```

#### III. Try importing again and show the schema
`sqlite> .import --csv /home/admin/table_tableau11.csv elect`  
`sqlite> .schema elect`  
```console
CREATE TABLE IF NOT EXISTS "elect"(
  "Province" TEXT,
  "Electoral District Name/Nom de circonscription" TEXT,
  "Electoral District Number/Numéro de circonscription" TEXT,
  "Population" TEXT,
  "Electors/Électeurs" TEXT,
  "Polling Stations/Bureaux de scrutin" TEXT,
  "Valid Ballots/Bulletins valides" TEXT,
  "Percentage of Valid Ballots /Pourcentage des bulletins valides" TEXT,
  "Rejected Ballots/Bulletins rejetés" TEXT,
  "Percentage of Rejected Ballots /Pourcentage des bulletins rejetés" TEXT,
  "Total Ballots Cast/Total des bulletins déposés" TEXT,
  "Percentage of Voter Turnout/Pourcentage de la participation électorale" TEXT,
  "Elected Candidate/Candidat élu" TEXT
);
```

##### We have a table with the data but the numbers are represented as text. But we can copy the schema output and change "Population" and "Rejected Ballots/Bulletins rejetés" from TEXT to INTEGER

#### IV. Recreate the table with INTEGER fields
```console
sqlite> CREATE TABLE IF NOT EXISTS "elections"(
  "Province" TEXT,
  "Electoral District Name/Nom de circonscription" TEXT,
  "Electoral District Number/Numéro de circonscription" TEXT,
  "Population" INTEGER,
  "Electors/Électeurs" TEXT,
  "Polling Stations/Bureaux de scrutin" TEXT,
  "Valid Ballots/Bulletins valides" TEXT,
  "Percentage of Valid Ballots /Pourcentage des bulletins valides" TEXT,
  "Rejected Ballots/Bulletins rejetés" INTEGER,
  "Percentage of Rejected Ballots /Pourcentage des bulletins rejetés" TEXT,
  "Total Ballots Cast/Total des bulletins déposés" TEXT,
  "Percentage of Voter Turnout/Pourcentage de la participation électorale" TEXT,
  "Elected Candidate/Candidat élu" TEXT
);
```

#### V. Try importing once more
```console
sqlite> .import --csv /home/admin/table_tableau11.csv elections
```

#### VI. Select needed value now
`sqlite> select "Electoral District Name/Nom de circonscription" from elections where "Population" < 100000 order by "Rejected Ballots/Bulletins rejetés" desc limit 1;`  
```console
Montcalm
```
