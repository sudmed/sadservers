## Minneapolis: Break a CSV file

**Description**: Break the Comma Separated Valued (CSV) file data.csv in the `/home/admin/` directory into exactly 10 smaller files of about the same size named `data-00.csv`, `data-01.csv`, ... , `data-09.csv` files in the same directory.  
All the files should have the same header (first line with column names) as data.csv. None of the smaller files should be bigger than 32KB.  

Note: to simplify, disregard broken lines in your files (ie, you can break a file at any point, not just at a newline). The resulting files don't have to be proper CSV files.  

**Test**: The "Check My Solution" button runs the script `/home/admin/agent/check.sh`, which you can see and execute.

---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 344
drwxr-xr-x 5 admin admin   4096 Jul 20 16:49 .
drwxr-xr-x 3 root  root    4096 Feb 17  2024 ..
drwx------ 3 admin admin   4096 Feb 17  2024 .ansible
-rw-r--r-- 1 admin admin    220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 admin admin   3526 Mar 27  2022 .bashrc
-rw-r--r-- 1 admin admin    807 Mar 27  2022 .profile
drwx------ 2 admin admin   4096 Feb 17  2024 .ssh
-rw-r--r-- 1 admin admin    422 Jul 20 16:49 README.txt
drwxr-xr-x 2 admin root    4096 Jul 20 16:49 agent
-rw-r--r-- 1 admin admin 312715 Jul 20 16:49 data.csv
```

`cat README.txt`  
```console
# "Minneapolis": Break a CSV file
## Description
Break the Comma Separated Valued (CSV) file data.csv in the /home/admin/ directory into exactly 10 smaller files of about the same size named data-00.csv, data-01.csv, ... , data-09.csv files in the same directory. All the files should be CSV files and have the same header (first line with column names) as data.csv. None of the smaller files should be bigger than 32KB
```


`head -n 400 data.csv`  
<details>
  <summary>Spoiler text output</summary>
  
```console
Province,Electoral District Name/Nom de circonscription,Electoral District Number/Numéro de circonscription,Candidate/Candidat,Candidate Residence/Résidence du candidat,Candidate Occupation/Profession du candidat,Votes Obtained/Votes obtenus,Percentage of Votes Obtained /Pourcentage des votes obtenus,Majority/Majorité,Majority Percentage/Pourcentage de majorité
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Avalon","10001","Ken McDonald Liberal/Libéral","Conception Bay South, N.L./ T.-N.-L.","Retired/Retraité",23528,55.9,     16027,  38.1
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Avalon","10001","Scott Andrews ** No Affiliation/Aucune appartenance","Conception Bay South, N.L./ T.-N.-L.","Parliamentarian/Parlementaire",7501,17.8,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Avalon","10001","Jeannie Baldwin NDP-New Democratic Party/NPD-Nouveau Parti démocratique","St. John's, N.L./ T.-N.-L.","Union Officer/Dirigeante syndicale",6075,14.4,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Avalon","10001","Lorraine E. Barnett Conservative/Conservateur","Paradise, N.L./ T.-N.-L.","Director Regional Affairs/Directrice des affaires régionales",4670,11.1,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Avalon","10001","Krista Byrne-Puumala Green Party/Parti Vert","Conception Bay South, N.L./ T.-N.-L.","Environmental practitioner/Student/Business owner/Spécialiste de l'environnement/étudiant/Prop. ent.",228,.5,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Avalon","10001","Jennifer McCreath Forces et Démocratie - Allier les forces de nos régions/Forces et Démocratie - Allier les forces de nos régions","St. John's, N.L./ T.-N.-L.","Civil Servant/Fonctionnaire",84,.2,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Bonavista--Burin--Trinity","10002","Judy M. Foote ** Liberal/Libéral","St. John's, N.L./ T.-N.-L.","Parliamentarian/Parlementaire",28704,81.8,     25170,  71.7
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Bonavista--Burin--Trinity","10002","Mike Windsor Conservative/Conservateur","Paradise, N.L./ T.-N.-L.","Leadership Consultant/Consultant en direction",3534,10.1,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Bonavista--Burin--Trinity","10002","Jenn Brown NDP-New Democratic Party/NPD-Nouveau Parti démocratique","St. John's, N.L./ T.-N.-L.","Industry Liaison Officer/Agente de liaison de l'industrie",2557,7.3,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Bonavista--Burin--Trinity","10002","Tyler John Colbourne Green Party/Parti Vert","Dartmouth, N.S./ N.-É.","Tour Guide/Guide touristique",297,.8,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Coast of Bays--Central--Notre Dame","10003","Scott Simms ** Liberal/Libéral","Norris Arm, N.L./ T.-N.-L.","Parliamentarian/Parlementaire",26523,74.8,     20044,  56.5
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Coast of Bays--Central--Notre Dame","10003","Kevin George O'Brien Conservative/Conservateur","Gander, N.L./ T.-N.-L.","Retired/Retraité",6479,18.3,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Coast of Bays--Central--Notre Dame","10003","Claudette Menchenton NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Gander, N.L./ T.-N.-L.","Home Decor/Décoration de la maison",2175,6.1,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Coast of Bays--Central--Notre Dame","10003","Elizabeth Perry Green Party/Parti Vert","Halifax, N.S./ N.-É.","Educator/Éducatrice",271,.8,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Labrador","10004","Yvonne Jones ** Liberal/Libéral","Mary's Harbour, N.L./ T.-N.-L.","Parliamentarian/Parlementaire",8878,71.8,      7099,  57.4
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Labrador","10004","Edward Rudkowski NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Toronto, Ont.","Consultant/Consultant",1779,14.4,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Labrador","10004","Peter Penashue Conservative/Conservateur","Sheshatshiu, N.L./ T.-N.-L.","Business Person/Personnes d'affaires",1716,13.9,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Long Range Mountains","10005","Gudie Hutchings Liberal/Libéral","Corner Brook, N.L./ T.-N.-L.","Business Woman/Femmes d'affaires",30889,73.9,     25804,  61.7
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Long Range Mountains","10005","Wayne Ruth Conservative/Conservateur","Kippens, N.L./ T.-N.-L.","Retired/Retraité",5085,12.2,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Long Range Mountains","10005","Devon Babstock NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Rocky Harbour, N.L./ T.-N.-L.","School Administrator/Administrateur scolaire",4739,11.3,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","Long Range Mountains","10005","Terry Cormier Green Party/Parti Vert","Corner Brook, N.L./ T.-N.-L.","Retired Public Servant/Fonctionnaire retraité",1111,2.7,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's East/St. John's-Est","10006","Nick Whalen Liberal/Libéral","St. John's, N.L./ T.-N.-L.","Lawyer, Patent Agent, Trademark Agent /Avocat, Agent de brevets, Agent de marques de comm",20974,46.7,       646,   1.4
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's East/St. John's-Est","10006","Jack Harris ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","St. John's, N.L./ T.-N.-L.","Lawyer/Avocat",20328,45.3,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's East/St. John's-Est","10006","Deanne Stapleton Conservative/Conservateur","St. John's, N.L./ T.-N.-L.","Administrative Assistant/Adjointe administrative",2938,6.5,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's East/St. John's-Est","10006","David Anthony Peters Green Party/Parti Vert","St. John's, N.L./ T.-N.-L.","Telecommunications Analyst/Analyste de télécommunications",500,1.1,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's East/St. John's-Est","10006","Sean Burton Communist/Communiste","St. John's, N.L./ T.-N.-L.","Teacher/Enseignant",140,.3,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's South--Mount Pearl/St. John's-Sud--Mount Pearl","10007","Seamus O'Regan Liberal/Libéral","St. John's, N.L./ T.-N.-L.","Consultant/Consultant",25922,57.9,      9455,  21.1
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's South--Mount Pearl/St. John's-Sud--Mount Pearl","10007","Ryan Cleary ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","St. John's, N.L./ T.-N.-L.","Parliamentarian/Parlementaire",16467,36.8,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's South--Mount Pearl/St. John's-Sud--Mount Pearl","10007","Marek Krol Conservative/Conservateur","St. John's, N.L./ T.-N.-L.","Self-employeed/Travailleur indépendant",2047,4.6,,
"Newfoundland and Labrador/Terre-Neuve-et-Labrador","St. John's South--Mount Pearl/St. John's-Sud--Mount Pearl","10007","Jackson McLean Green Party/Parti Vert","St. John's, N.L./ T.-N.-L.","Assistant Manager of The Seed Company by E.W. Gaze/Directeur adjoint, The Seed Company by E.W. Gaze",365,.8,,
"Prince Edward Island/Île-du-Prince-Édouard","Cardigan","11001","Lawrence MacAulay ** Liberal/Libéral","St. Peters Bay, P.E.I./ Î.-P.-É.","Farmer/Agriculteur",14621,65,     10989,  48.9
"Prince Edward Island/Île-du-Prince-Édouard","Cardigan","11001","Julius Patkai Conservative/Conservateur","Montague, P.E.I./ Î.-P.-É.","Management Consultant/Consultant en gestion",3632,16.2,,
"Prince Edward Island/Île-du-Prince-Édouard","Cardigan","11001","Billy Cann NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Montague, P.E.I./ Î.-P.-É.","Business Owner/Propriétaire d'entreprise",2503,11.1,,
"Prince Edward Island/Île-du-Prince-Édouard","Cardigan","11001","Teresa Doyle Green Party/Parti Vert","Belfast, P.E.I./ Î.-P.-É.","Musician/Musicienne",1434,6.4,,
"Prince Edward Island/Île-du-Prince-Édouard","Cardigan","11001","Christene Squires Christian Heritage Party/Parti de l'Héritage Chrétien","Murray River, P.E.I./ Î.-P.-É.","Unemployed/Sans emploi",295,1.3,,
"Prince Edward Island/Île-du-Prince-Édouard","Charlottetown","11002","Sean Casey ** Liberal/Libéral","Charlottetown, P.E.I./ Î.-P.-É.","Lawyer/Avocat",11910,56.3,      7013,  33.1
"Prince Edward Island/Île-du-Prince-Édouard","Charlottetown","11002","Joe Byrne NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Charlottetown, P.E.I./ Î.-P.-É.","Community Organizer/Organisateur communautaire",4897,23.1,,
"Prince Edward Island/Île-du-Prince-Édouard","Charlottetown","11002","Ron MacMillan Conservative/Conservateur","Charlottetown, P.E.I./ Î.-P.-É.","Lawyer/Avocat",3136,14.8,,
"Prince Edward Island/Île-du-Prince-Édouard","Charlottetown","11002","Becka Viau Green Party/Parti Vert","Charlottetown, P.E.I./ Î.-P.-É.","Artist/Artiste",1222,5.8,,
"Prince Edward Island/Île-du-Prince-Édouard","Egmont","11003","Bobby Morrissey Liberal/Libéral","Tignish, P.E.I./ Î.-P.-É.","Businessman/Homme d'affaires",10521,49.3,      4336,  20.3
"Prince Edward Island/Île-du-Prince-Édouard","Egmont","11003","Gail Shea ** Conservative/Conservateur","Tignish, P.E.I./ Î.-P.-É.","Parliamentarian/Parlementaire",6185,29,,
"Prince Edward Island/Île-du-Prince-Édouard","Egmont","11003","Herb Dickieson NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Howlan, P.E.I./ Î.-P.-É.","Family Physician/Médecin de famille",4097,19.2,,
"Prince Edward Island/Île-du-Prince-Édouard","Egmont","11003","Nils Ling Green Party/Parti Vert","Breadalbane, P.E.I./ Î.-P.-É.","Writer/Écrivain",559,2.6,,
"Prince Edward Island/Île-du-Prince-Édouard","Malpeque","11004","Wayne Easter ** Liberal/Libéral","North Wiltshire, P.E.I./ Î.-P.-É.","Farmer/Parliamentarian/Agriculteur-Parlementaire",13950,62.1,     10003,  44.5
"Prince Edward Island/Île-du-Prince-Édouard","Malpeque","11004","Stephen Stewart Conservative/Conservateur","Kensington, P.E.I./ Î.-P.-É.","Business Owner/Propriétaire d'entreprise",3947,17.6,,
"Prince Edward Island/Île-du-Prince-Édouard","Malpeque","11004","Leah-Jane Hayward NDP-New Democratic Party/NPD-Nouveau Parti démocratique","North Wiltshire, P.E.I./ Î.-P.-É.","Tour Guide/Guide touristique",2509,11.2,,
"Prince Edward Island/Île-du-Prince-Édouard","Malpeque","11004","Lynne Lund Green Party/Parti Vert","Clinton, P.E.I./ Î.-P.-É.","Business Owner/Propriétaire d'entreprise",2066,9.2,,
"Nova Scotia/Nouvelle-Écosse","Cape Breton--Canso","12001","Rodger Cuzner ** Liberal/Libéral","Glace Bay, N.S./ N.-É.","Parliamentarian/Parlementaire",32163,74.4,     25917,  59.9
"Nova Scotia/Nouvelle-Écosse","Cape Breton--Canso","12001","Adam Daniel Rodgers Conservative/Conservateur","Boylston, N.S./ N.-É.","Lawyer/Avocat",6246,14.4,,
"Nova Scotia/Nouvelle-Écosse","Cape Breton--Canso","12001","Michelle Smith NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Whycocomagh, N.S./ N.-É.","Farmer/Agricultrice",3547,8.2,,
"Nova Scotia/Nouvelle-Écosse","Cape Breton--Canso","12001","Maria Goretti Coady Green Party/Parti Vert","North East Margaree, N.S./ N.-É.","Nurse/Infirmière",1281,3,,
"Nova Scotia/Nouvelle-Écosse","Central Nova/Nova-Centre","12002","Sean Fraser Liberal/Libéral","Merigomish, N.S./ N.-É.","Barrister and Solicitor/Avocat",25909,58.5,     14491,  32.7
"Nova Scotia/Nouvelle-Écosse","Central Nova/Nova-Centre","12002","Fred DeLorey Conservative/Conservateur","Embrun, Ont.","Political Advisor/Conseiller politique",11418,25.8,,
"Nova Scotia/Nouvelle-Écosse","Central Nova/Nova-Centre","12002","Ross Landry NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Pictou, N.S./ N.-É.","Retired/Retraité",4532,10.2,,
"Nova Scotia/Nouvelle-Écosse","Central Nova/Nova-Centre","12002","David Hachey Green Party/Parti Vert","Scotsburn, N.S./ N.-É.","Farmer/Agriculteur",1834,4.1,,
"Nova Scotia/Nouvelle-Écosse","Central Nova/Nova-Centre","12002","Alexander J. MacKenzie Independent/Indépendant","Pictou Landing, N.S./ N.-É.","Self-employed/Travailleur indépendant",570,1.3,,
"Nova Scotia/Nouvelle-Écosse","Cumberland--Colchester","12003","Bill Casey Liberal/Libéral","Amherst, N.S./ N.-É.","Retired/Retraité",29527,63.7,     17270,  37.3
"Nova Scotia/Nouvelle-Écosse","Cumberland--Colchester","12003","Scott Armstrong ** Conservative/Conservateur","Brookfield, N.S./ N.-É.","Educator/Éducateur",12257,26.5,,
"Nova Scotia/Nouvelle-Écosse","Cumberland--Colchester","12003","Wendy Robinson NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Stewiacke, N.S./ N.-É.","Municipal Politician/Politicienne municipale",2647,5.7,,
"Nova Scotia/Nouvelle-Écosse","Cumberland--Colchester","12003","Jason Matthew Blanch Green Party/Parti Vert","Amherst, N.S./ N.-É.","Counsellor/Conseiller",1650,3.6,,
"Nova Scotia/Nouvelle-Écosse","Cumberland--Colchester","12003","Kenneth Jackson Independent/Indépendant","Amherst, N.S./ N.-É.","Self-employed/Travailleur indépendant",181,.4,,
"Nova Scotia/Nouvelle-Écosse","Cumberland--Colchester","12003","Richard Trueman Plett Independent/Indépendant","Amherst, N.S./ N.-É.","Self-employed/Travailleur indépendant",70,.2,,
"Nova Scotia/Nouvelle-Écosse","Dartmouth--Cole Harbour","12004","Darren Fisher Liberal/Libéral","Dartmouth, N.S./ N.-É.","Municipal Councillor/Conseiller municipal",30407,58.2,     17650,  33.8
"Nova Scotia/Nouvelle-Écosse","Dartmouth--Cole Harbour","12004","Robert Chisholm ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Porters Lake, N.S./ N.-É.","Parliamentarian/Parlementaire",12757,24.4,,
"Nova Scotia/Nouvelle-Écosse","Dartmouth--Cole Harbour","12004","Jason Cole Conservative/Conservateur","Dartmouth, N.S./ N.-É.","Clergy/Membre du clergé",7331,14,,
"Nova Scotia/Nouvelle-Écosse","Dartmouth--Cole Harbour","12004","Brynn Nheiley Green Party/Parti Vert","Halifax, N.S./ N.-É.","Urban planner/Urbaniste",1775,3.4,,
"Nova Scotia/Nouvelle-Écosse","Halifax","12005","Andy Fillmore Liberal/Libéral","Halifax, N.S./ N.-É.","City Planner/Urbaniste",27431,51.7,      8269,  15.6
"Nova Scotia/Nouvelle-Écosse","Halifax","12005","Megan Leslie ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Halifax, N.S./ N.-É.","Parliamentarian/Parlementaire",19162,36.1,,
"Nova Scotia/Nouvelle-Écosse","Halifax","12005","Irvine Carvery Conservative/Conservateur","Halifax, N.S./ N.-É.","Retired/Retraité",4564,8.6,,
"Nova Scotia/Nouvelle-Écosse","Halifax","12005","Thomas Trappenberg Green Party/Parti Vert","Hatchet Lake, N.S./ N.-É.","Professor/Professeur",1745,3.3,,
"Nova Scotia/Nouvelle-Écosse","Halifax","12005","Allan Bezanson Marxist-Leninist/Marxiste-Léniniste","Halifax, N.S./ N.-É.","Retired/Rétraité",130,.2,,
"Nova Scotia/Nouvelle-Écosse","Halifax West/Halifax-Ouest","12006","Geoff Regan ** Liberal/Libéral","Bedford, N.S./ N.-É.","Parliamentarian/Parlementaire",34377,68.6,     26540,  53.0
"Nova Scotia/Nouvelle-Écosse","Halifax West/Halifax-Ouest","12006","Michael McGinnis Conservative/Conservateur","Halifax, N.S./ N.-É.","Site supervisor/Surveillant du chantier",7837,15.6,,
"Nova Scotia/Nouvelle-Écosse","Halifax West/Halifax-Ouest","12006","Joanne Hussey NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Halifax, N.S./ N.-É.","Research Consultant/Conseillère en recherche",5894,11.8,,
"Nova Scotia/Nouvelle-Écosse","Halifax West/Halifax-Ouest","12006","Richard Henryk Zurawski Green Party/Parti Vert","Halifax, N.S./ N.-É.","Lecturer/Conférencier",1971,3.9,,
"Nova Scotia/Nouvelle-Écosse","Kings--Hants","12007","Scott Brison ** Liberal/Libéral","Cheverie, N.S./ N.-É.","Parliamentarian/Parlementaire",33026,70.7,     24349,  52.2
"Nova Scotia/Nouvelle-Écosse","Kings--Hants","12007","David Morse Conservative/Conservateur","New Minas, N.S./ N.-É.","Retired/Retraité",8677,18.6,,
"Nova Scotia/Nouvelle-Écosse","Kings--Hants","12007","Hugh Curry NDP-New Democratic Party/NPD-Nouveau Parti démocratique","White Rock, N.S./ N.-É.","Vintner's Assistant/Assistant Vigneron",2998,6.4,,
"Nova Scotia/Nouvelle-Écosse","Kings--Hants","12007","Will Cooper Green Party/Parti Vert","Wolfville, N.S./ N.-É.","Artist/Artiste",1569,3.4,,
"Nova Scotia/Nouvelle-Écosse","Kings--Hants","12007","Megan Brown-Hodges Rhinoceros/Rhinocéros","Canning, N.S./ N.-É.","Stay-at-home Mother/Mère de famille",184,.4,,
"Nova Scotia/Nouvelle-Écosse","Kings--Hants","12007","Edd Twohig Independent/Indépendant","Kentville, N.S./ N.-É.","Retired Chartered Accountant/Comptable agréé retraité",132,.3,,
"Nova Scotia/Nouvelle-Écosse","Kings--Hants","12007","Clifford James Williams Independent/Indépendant","Hantsport, N.S./ N.-É.","Sales Account Manager/Gestionnaire des comptes des ventes",100,.2,,
"Nova Scotia/Nouvelle-Écosse","Sackville--Preston--Chezzetcook","12008","Darrell Samson Liberal/Libéral","Fall River, N.S./ N.-É.","Supt. of Schools/Surintendant des écoles",23161,48,      6548,  13.6
"Nova Scotia/Nouvelle-Écosse","Sackville--Preston--Chezzetcook","12008","Peter Stoffer ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Fall River, N.S./ N.-É.","Parliamentarian/Parlementaire",16613,34.4,,
"Nova Scotia/Nouvelle-Écosse","Sackville--Preston--Chezzetcook","12008","Robert Thomas Strickland Conservative/Conservateur","Beaver Bank, N.S./ N.-É.","Software Consultant/Consultant de logiciels",7186,14.9,,
"Nova Scotia/Nouvelle-Écosse","Sackville--Preston--Chezzetcook","12008","Mike Montgomery Green Party/Parti Vert","Lower Sackville, N.S./ N.-É.","Security Manager/Directeur de la sécurité",1341,2.8,,
"Nova Scotia/Nouvelle-Écosse","South Shore--St. Margarets","12009","Bernadette Jordan Liberal/Libéral","LaHave, N.S./ N.-É.","Development Officer/Agente de développement",30045,56.9,     18140,  34.4
"Nova Scotia/Nouvelle-Écosse","South Shore--St. Margarets","12009","Richard Clark Conservative/Conservateur","Barrington Passage, N.S./ N.-É.","Policy Director/Directeur de la politique",11905,22.6,,
"Nova Scotia/Nouvelle-Écosse","South Shore--St. Margarets","12009","Alex Godbold NDP-New Democratic Party/NPD-Nouveau Parti démocratique","New Germany, N.S./ N.-É.","Teacher/Enseignant",8883,16.8,,
"Nova Scotia/Nouvelle-Écosse","South Shore--St. Margarets","12009","Richard Biggar Green Party/Parti Vert","Hubbards, N.S./ N.-É.","Correctional officer/Agent correctionnel",1534,2.9,,
"Nova Scotia/Nouvelle-Écosse","South Shore--St. Margarets","12009","Trevor Bruhm Independent/Indépendant","Pleasantville, N.S./ N.-É.","Insulator Mechanic/Mécanicien des isolateurs",257,.5,,
"Nova Scotia/Nouvelle-Écosse","South Shore--St. Margarets","12009","Ryan Barry Communist/Communiste","North River, N.S./ N.-É.","Residential Rehabilitation Worker/Agent de remise en état des logements",151,.3,,
"Nova Scotia/Nouvelle-Écosse","Sydney--Victoria","12010","Mark Eyking ** Liberal/Libéral","Big Bras d'Or, N.S./ N.-É.","Farmer/Agriculteur",29995,73.2,     24644,  60.1
"Nova Scotia/Nouvelle-Écosse","Sydney--Victoria","12010","Monika Dutt NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Sydney, N.S./ N.-É.","Physician/Médecin",5351,13.1,,
"Nova Scotia/Nouvelle-Écosse","Sydney--Victoria","12010","John Douglas Chiasson Conservative/Conservateur","Ottawa, Ont.","Ministerial Staffer/Membre du personnel ministériel",4360,10.6,,
"Nova Scotia/Nouvelle-Écosse","Sydney--Victoria","12010","Adrianna MacKinnon Green Party/Parti Vert","Membertou, N.S./ N.-É.","Teacher/Enseignante",1026,2.5,,
"Nova Scotia/Nouvelle-Écosse","Sydney--Victoria","12010","Wayne James Hiscock Libertarian/Libertarien","Balls Creek, N.S./ N.-É.","Geomatics Engineering technologist /Technologue en génie géomatique",242,.6,,
"Nova Scotia/Nouvelle-Écosse","West Nova/Nova-Ouest","12011","Colin Fraser Liberal/Libéral","Summerville, N.S./ N.-É.","Lawyer/Avocat",28775,63,     16859,  36.9
"Nova Scotia/Nouvelle-Écosse","West Nova/Nova-Ouest","12011","Arnold LeBlanc Conservative/Conservateur","Digby, N.S./ N.-É.","Executive Regional Assistant to Parliamentarian/Adjoint exécutif régional d'un parlementaire",11916,26.1,,
"Nova Scotia/Nouvelle-Écosse","West Nova/Nova-Ouest","12011","Greg Foster NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Tusket, N.S./ N.-É.","Teacher/Enseignant",3084,6.8,,
"Nova Scotia/Nouvelle-Écosse","West Nova/Nova-Ouest","12011","Clark Walton Green Party/Parti Vert","Paradise, N.S./ N.-É.","Retired Military/Militaire retraité",1904,4.2,,
"New Brunswick/Nouveau-Brunswick","Acadie--Bathurst","13001","Serge Cormier Liberal/Libéral","Caraquet, N.B./ N.-B.","Public Servant/Fonctionnaire",25845,50.7,      5766,  11.3
"New Brunswick/Nouveau-Brunswick","Acadie--Bathurst","13001","Jason Godin NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Maisonnette, N.B./ N.-B.","Student/Étudiant",20079,39.4,,
"New Brunswick/Nouveau-Brunswick","Acadie--Bathurst","13001","Riba Girouard-Riordon Conservative/Conservateur","Pokeshaw, N.B./ N.-B.","Retired professor/Professeuse a la retaite",3852,7.6,,
"New Brunswick/Nouveau-Brunswick","Acadie--Bathurst","13001","Dominique Breau Green Party/Parti Vert","Pont-Landry, N.B./ N.-B.","Artist/Artiste",1187,2.3,,
"New Brunswick/Nouveau-Brunswick","Beauséjour","13002","Dominic LeBlanc ** Liberal/Libéral","Grande-Digue, N.B./ N.-B.","Parliamentarian/Parlementaire",36534,69,     28525,  53.9
"New Brunswick/Nouveau-Brunswick","Beauséjour","13002","Hélène Boudreau NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Sackville, N.B./ N.-B.","Registered Nurse/Infirmière autorisée",8009,15.1,,
"New Brunswick/Nouveau-Brunswick","Beauséjour","13002","Ann Bastarache Conservative/Conservateur","Haut-Saint-Antoine, N.B./ N.-B.","Realtor/Agente d'immeuble",6017,11.4,,
"New Brunswick/Nouveau-Brunswick","Beauséjour","13002","Kevin King Green Party/Parti Vert","Ammon, N.B./ N.-B.","Engineering Technologist/Ingénieur technologue",2376,4.5,,
"New Brunswick/Nouveau-Brunswick","Fredericton","13003","Matt DeCourcey Liberal/Libéral","Fredericton, N.B./ N.-B.","Public Servant/Fonctionnaire",23016,49.3,      9736,  20.8
"New Brunswick/Nouveau-Brunswick","Fredericton","13003","Keith Ashfield ** Conservative/Conservateur","Lincoln, N.B./ N.-B.","Parliamentarian/Parlementaire",13280,28.4,,
"New Brunswick/Nouveau-Brunswick","Fredericton","13003","Mary Lou Babineau Green Party/Parti Vert","Fredericton, N.B./ N.-B.","Professor/Professeur",5804,12.4,,
"New Brunswick/Nouveau-Brunswick","Fredericton","13003","Sharon Scott-Levesque NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Durham Bridge, N.B./ N.-B.","Nurse/Infirmière",4622,9.9,,
"New Brunswick/Nouveau-Brunswick","Fundy Royal","13004","Alaina Lockhart Liberal/Libéral","Sussex, N.B./ N.-B.","Business Owner/Propriétaire d'entreprise",19136,40.9,      1775,   3.8
"New Brunswick/Nouveau-Brunswick","Fundy Royal","13004","Rob Moore ** Conservative/Conservateur","Quispamsis, N.B./ N.-B.","Parliamentarian/Parlementaire",17361,37.1,,
"New Brunswick/Nouveau-Brunswick","Fundy Royal","13004","Jennifer McKenzie NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Bay View, N.B./ N.-B.","Engineer/Ingénieur",8204,17.5,,
"New Brunswick/Nouveau-Brunswick","Fundy Royal","13004","Stephanie Coburn Green Party/Parti Vert","Head of Millstream, N.B./ N.-B.","Retired/Retraité",1823,3.9,,
"New Brunswick/Nouveau-Brunswick","Fundy Royal","13004","David Raymond Amos Independent/Indépendant","not available/non disponible","Mechanic/Mécanicien",296,.6,,
"New Brunswick/Nouveau-Brunswick","Madawaska--Restigouche","13005","René Arseneault Liberal/Libéral","Balmoral, N.B./ N.-B.","Lawyer/Avocat",20778,55.7,     11108,  29.8
"New Brunswick/Nouveau-Brunswick","Madawaska--Restigouche","13005","Rosaire L'Italien NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Moncton, N.B./ N.-B.","Retired/Retraité",9670,25.9,,
"New Brunswick/Nouveau-Brunswick","Madawaska--Restigouche","13005","Bernard Valcourt ** Conservative/Conservateur","Edmundston, N.B./ N.-B.","Lawyer/Avocat",6151,16.5,,
"New Brunswick/Nouveau-Brunswick","Madawaska--Restigouche","13005","Françoise Aubin Green Party/Parti Vert","Moncton, N.B./ N.-B.","Student/Étudiante",707,1.9,,
"New Brunswick/Nouveau-Brunswick","Miramichi--Grand Lake","13006","Pat Finnigan Liberal/Libéral","Rogersville, N.B./ N.-B.","Business Owner/Propriétaire d'entreprise",17202,47.3,      4726,  13.0
"New Brunswick/Nouveau-Brunswick","Miramichi--Grand Lake","13006","Tilly O'Neill-Gordon ** Conservative/Conservateur","Miramichi, N.B./ N.-B.","Primary Teacher/Enseignante au primaire",12476,34.3,,
"New Brunswick/Nouveau-Brunswick","Miramichi--Grand Lake","13006","Patrick Colford NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Warwick Settlement, N.B./ N.-B.","President - New Brunswick Federation of Labour/Président - FTTNB",5588,15.4,,
"New Brunswick/Nouveau-Brunswick","Miramichi--Grand Lake","13006","Matthew Ian Clark Green Party/Parti Vert","Moncton, N.B./ N.-B.","Accounting Technician / Cust. Service/Technicien en comptabilité/Service à la clientèle",1098,3,,
"New Brunswick/Nouveau-Brunswick","Moncton--Riverview--Dieppe","13007","Ginette Petitpas Taylor Liberal/Libéral","Moncton, N.B./ N.-B.","Codiac RCMP Victim Services Coordinator/Coordonnatrice de l'aide aux vict., GRC de Codiac",30054,57.8,     18886,  36.3
"New Brunswick/Nouveau-Brunswick","Moncton--Riverview--Dieppe","13007","Robert Goguen ** Conservative/Conservateur","Moncton, N.B./ N.-B.","Parliamentarian/Parlementaire",11168,21.5,,
"New Brunswick/Nouveau-Brunswick","Moncton--Riverview--Dieppe","13007","Luc LeBlanc NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Moncton, N.B./ N.-B.","Historian/Historien",8420,16.2,,
"New Brunswick/Nouveau-Brunswick","Moncton--Riverview--Dieppe","13007","Luc Melanson Green Party/Parti Vert","Dieppe, N.B./ N.-B.","Sales Representative/Représentant des ventes",2399,4.6,,
"New Brunswick/Nouveau-Brunswick","New Brunswick Southwest/Nouveau-Brunswick-Sud-Ouest","13008","Karen Ludwig Liberal/Libéral","Saint Andrews, N.B./ N.-B.","Associate Dean/Doyenne associée",16656,43.9,      2031,   5.4
"New Brunswick/Nouveau-Brunswick","New Brunswick Southwest/Nouveau-Brunswick-Sud-Ouest","13008","John Williamson ** Conservative/Conservateur","Saint Andrews, N.B./ N.-B.","Parliamentarian/Parlementaire",14625,38.6,,
"New Brunswick/Nouveau-Brunswick","New Brunswick Southwest/Nouveau-Brunswick-Sud-Ouest","13008","Andrew Graham NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Saint John, N.B./ N.-B.","Carpenter/Charpentier",4768,12.6,,
"New Brunswick/Nouveau-Brunswick","New Brunswick Southwest/Nouveau-Brunswick-Sud-Ouest","13008","Gayla MacIntosh Green Party/Parti Vert","Fredericton, N.B./ N.-B.","Taxi Driver/Chauffeuse de taxi",1877,4.9,,
"New Brunswick/Nouveau-Brunswick","Saint John--Rothesay","13009","Wayne Long Liberal/Libéral","Saint John, N.B./ N.-B.","President of Saint John Sea Dogs/Président des Saint John Sea Dogs",20634,48.8,      7719,  18.3
"New Brunswick/Nouveau-Brunswick","Saint John--Rothesay","13009","Rodney Weston ** Conservative/Conservateur","Saint John, N.B./ N.-B.","Parliamentarian/Parlementaire",12915,30.5,,
"New Brunswick/Nouveau-Brunswick","Saint John--Rothesay","13009","AJ Griffin NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Quispamsis, N.B./ N.-B.","Technical Writer/Rédactrice technique",7411,17.5,,
"New Brunswick/Nouveau-Brunswick","Saint John--Rothesay","13009","Sharon Murphy Green Party/Parti Vert","Rothesay, N.B./ N.-B.","Businesswoman/Femme d'affaires",1321,3.1,,
"New Brunswick/Nouveau-Brunswick","Tobique--Mactaquac","13010","TJ Harvey Liberal/Libéral","Glassville, N.B./ N.-B.","Businessman/Homme d'affaires",17909,46.6,      3684,   9.6
"New Brunswick/Nouveau-Brunswick","Tobique--Mactaquac","13010","Richard Bragdon Conservative/Conservateur","Keswick Ridge, N.B./ N.-B.","Clergyman/Membre du clergé",14225,37,,
"New Brunswick/Nouveau-Brunswick","Tobique--Mactaquac","13010","Robert Kitchen NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Nackawic, N.B./ N.-B.","Salesman/Vendeur",4334,11.3,,
"New Brunswick/Nouveau-Brunswick","Tobique--Mactaquac","13010","Terry Wishart Green Party/Parti Vert","Fredericton, N.B./ N.-B.","Community Organizer/Organisateur communautaire",1959,5.1,,
"Quebec/Québec","Abitibi--Baie-James--Nunavik--Eeyou","24001","Romeo Saganash ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Val-d'Or, Que./ Qc","Parliamentarian/Parlementaire",12778,37,      1684,   4.9
"Quebec/Québec","Abitibi--Baie-James--Nunavik--Eeyou","24001","Pierre Dufour Liberal/Libéral","Val-d'Or, Que./ Qc","Administrator/Administrateur",11094,32.1,,
"Quebec/Québec","Abitibi--Baie-James--Nunavik--Eeyou","24001","Luc Ferland Bloc Québécois/Bloc Québécois","Québec, Que./ Qc","Consultant/Consultant",6398,18.5,,
"Quebec/Québec","Abitibi--Baie-James--Nunavik--Eeyou","24001","Steven Hébert Conservative/Conservateur","Ottawa, Ont.","Political Advisor/Conseiller Politique",3211,9.3,,
"Quebec/Québec","Abitibi--Baie-James--Nunavik--Eeyou","24001","Patrick Benoît Green Party/Parti Vert","Malartic, Que./ Qc","Operator/Opérateur",779,2.3,,
"Quebec/Québec","Abitibi--Baie-James--Nunavik--Eeyou","24001","Mario Gagnon Rhinoceros/Rhinocéros","Malartic, Que./ Qc","Special Education Teacher/Éducateur spécialisé",258,.7,,
"Quebec/Québec","Abitibi--Témiscamingue","24002","Christine Moore ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Dupuy, Que./ Qc","Parliamentarian/Nurse/Parlementaire-Infirmier",20636,41.5,      5903,  11.9
"Quebec/Québec","Abitibi--Témiscamingue","24002","Claude Thibault Liberal/Libéral","Rouyn-Noranda, Que./ Qc","Exports Commissioner/Commissaire à l'exportation",14733,29.6,,
"Quebec/Québec","Abitibi--Témiscamingue","24002","Yvon Moreau Bloc Québécois/Bloc Québécois","Rouyn-Noranda, Que./ Qc","Journalist/Journaliste",9651,19.4,,
"Quebec/Québec","Abitibi--Témiscamingue","24002","Benoit Fortin Conservative/Conservateur","Gatineau, Que./ Qc","Director/Directeur",3425,6.9,,
"Quebec/Québec","Abitibi--Témiscamingue","24002","Aline Bégin Green Party/Parti Vert","Gatineau, Que./ Qc","Teacher/Enseignante",859,1.7,,
"Quebec/Québec","Abitibi--Témiscamingue","24002","Pascal Le Fou Gélinas Rhinoceros/Rhinocéros","Rouyn-Noranda, Que./ Qc","Artist/Artiste",425,.9,,
"Quebec/Québec","Ahuntsic-Cartierville","24003","Mélanie Joly Liberal/Libéral","Montréal, Que./ Qc","Lawyer/Avocate",26026,46.8,      9342,  16.8
"Quebec/Québec","Ahuntsic-Cartierville","24003","Maria Mourani ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Blainville, Que./ Qc","Sociologist and criminologist/Sociolgue et criminologue",16684,30,,
"Quebec/Québec","Ahuntsic-Cartierville","24003","Nicolas Bourdon Bloc Québécois/Bloc Québécois","Montréal, Que./ Qc","Professor/Professeur",7346,13.2,,
"Quebec/Québec","Ahuntsic-Cartierville","24003","Wiliam Moughrabi Conservative/Conservateur","Montréal, Que./ Qc","Store Manager/Director de Magasin",4051,7.3,,
"Quebec/Québec","Ahuntsic-Cartierville","24003","Gilles Mercier Green Party/Parti Vert","Montréal, Que./ Qc","Work Inspector/Inspecteur du travail",1175,2.1,,
"Quebec/Québec","Ahuntsic-Cartierville","24003","Catherine Gascon-David Rhinoceros/Rhinocéros","Montréal, Que./ Qc","Student/Étudiante",285,.5,,
"Quebec/Québec","Alfred-Pellan","24004","Angelo Iacono Liberal/Libéral","Montréal, Que./ Qc","Lawyer/Avocat",24557,44.5,     11332,  20.5
"Quebec/Québec","Alfred-Pellan","24004","Rosane Doré Lefebvre ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Laval, Que./ Qc","Parliamentarian/Parlementaire",13225,24,,
"Quebec/Québec","Alfred-Pellan","24004","Daniel St-Hilaire Bloc Québécois/Bloc Québécois","Montréal, Que./ Qc","Professional athletics coach/Entraineur professionnel en athlétisme",9836,17.8,,
"Quebec/Québec","Alfred-Pellan","24004","Gabriel Purcarus Conservative/Conservateur","LaSalle, Que./ Qc","Courtier Immobilier/Realtor",6259,11.3,,
"Quebec/Québec","Alfred-Pellan","24004","Lynda Briguene Green Party/Parti Vert","Laval, Que./ Qc","President and Director /Présidente-Directrice",1089,2,,
"Quebec/Québec","Alfred-Pellan","24004","Renata Isopo Independent/Indépendant","Laval, Que./ Qc","Teacher/Enseignante",203,.4,,
"Quebec/Québec","Argenteuil--La Petite-Nation","24005","Stéphane Lauzon Liberal/Libéral","Gatineau, Que./ Qc","Municipal Councillor/Conseiller municipal",22093,43.3,      9443,  18.5
"Quebec/Québec","Argenteuil--La Petite-Nation","24005","Chantal Crête NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Lac-Simon, Que./ Qc","Educational Psychologist/Researcher/Psychopédagogue/Chercheure",12650,24.8,,
"Quebec/Québec","Argenteuil--La Petite-Nation","24005","Jonathan Beauchamp Bloc Québécois/Bloc Québécois","Ripon, Que./ Qc","Development Officer/Agent de développement",9525,18.7,,
"Quebec/Québec","Argenteuil--La Petite-Nation","24005","Maxime Hupé-Labelle Conservative/Conservateur","Thurso, Que./ Qc","Political Analyst/Analyste Politique",5680,11.1,,
"Quebec/Québec","Argenteuil--La Petite-Nation","24005","Audrey Lamarche Green Party/Parti Vert","Gatineau, Que./ Qc","Sports Therapist/Orthotherapist/Thérapeute sportive-Orthothérapeute",1118,2.2,,
"Quebec/Québec","Avignon--La Mitis--Matane--Matapédia","24006","Rémi Massé Liberal/Libéral","Saint-Léandre, Que./ Qc","Director General/Directeur général",14378,39.5,      6737,  18.5
"Quebec/Québec","Avignon--La Mitis--Matane--Matapédia","24006","Kédina Fleury-Samson Bloc Québécois/Bloc Québécois","Mont-Joli, Que./ Qc","Entrepreneur/Entrepreneure",7641,21,,
"Quebec/Québec","Avignon--La Mitis--Matane--Matapédia","24006","Joël Charest NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Price, Que./ Qc","Political Attaché/Attaché politique",7340,20.2,,
"Quebec/Québec","Avignon--La Mitis--Matane--Matapédia","24006","Jean-François Fortin ** Forces et Démocratie - Allier les forces de nos régions/Forces et Démocratie - Allier les forces de nos régions","Sainte-Flavie, Que./ Qc","Political science instructor/Enseignant en sciences politiques",4229,11.6,,
"Quebec/Québec","Avignon--La Mitis--Matane--Matapédia","24006","André Savoie Conservative/Conservateur","Québec, Que./ Qc","Storekeeper/Commerçant",2228,6.1,,
"Quebec/Québec","Avignon--La Mitis--Matane--Matapédia","24006","Sherri Springle Green Party/Parti Vert","Vancouver, B.C./ C.-B.","Student/Étudiante",365,1,,
"Quebec/Québec","Avignon--La Mitis--Matane--Matapédia","24006","Éric Normand Rhinoceros/Rhinocéros","Rimouski, Que./ Qc","Artist/Musician/Artiste/Musicien",175,.5,,
"Quebec/Québec","Beauce","24007","Maxime Bernier ** Conservative/Conservateur","Saint-Georges, Que./ Qc","Lawyer/Avocat",32910,58.9,     20468,  36.6
"Quebec/Québec","Beauce","24007","Adam Veilleux Liberal/Libéral","Saint-Benoît-Labre, Que./ Qc","Businessman/Homme d'affaires",12442,22.3,,
"Quebec/Québec","Beauce","24007","Daniel Royer NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Saint-Georges, Que./ Qc","Teacher/Enseignant",5443,9.7,,
"Quebec/Québec","Beauce","24007","Stéphane Trudel Bloc Québécois/Bloc Québécois","Trois-Rivières, Que./ Qc","Student-Researcher/Étudiant-Chercheur",4144,7.4,,
"Quebec/Québec","Beauce","24007","Céline Brown MacDonald Green Party/Parti Vert","Ottawa, Ont.","Information Officer/Agente d'information",943,1.7,,
"Quebec/Québec","Beauport--Côte-de-Beaupré--Île d'Orléans--Charlevoix","24020","Sylvie Boucher Conservative/Conservateur","Saint-Joachim, Que./ Qc","Manager/Gérante",16903,33.5,      3347,   6.6
"Quebec/Québec","Beauport--Côte-de-Beaupré--Île d'Orléans--Charlevoix","24020","Jean-Roger Vigneau Liberal/Libéral","Saint-Aimé-des-Lacs, Que./ Qc","Manager/Gestionnaire",13556,26.9,,
"Quebec/Québec","Beauport--Côte-de-Beaupré--Île d'Orléans--Charlevoix","24020","Sébastien Dufour Bloc Québécois/Bloc Québécois","Boischatel, Que./ Qc","Assistant Chief of Staff/Directeur de cabinet adjoint",9650,19.1,,
"Quebec/Québec","Beauport--Côte-de-Beaupré--Île d'Orléans--Charlevoix","24020","Jonathan Tremblay ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Beaupré, Que./ Qc","Parliamentarian/Parlementaire",9306,18.4,,
"Quebec/Québec","Beauport--Côte-de-Beaupré--Île d'Orléans--Charlevoix","24020","Patrick Kerr Green Party/Parti Vert","Québec, Que./ Qc","Engineer/Ingénieur",859,1.7,,
"Quebec/Québec","Beauport--Côte-de-Beaupré--Île d'Orléans--Charlevoix","24020","Mario Desjardins Pelchat Forces et Démocratie - Allier les forces de nos régions/Forces et Démocratie - Allier les forces de nos régions","Québec, Que./ Qc","Self-employed/Travailleur autonome",182,.4,,
"Quebec/Québec","Beauport--Limoilou","24008","Alupa Clarke Conservative/Conservateur","Québec, Que./ Qc","Military/Militaire",15461,30.6,      2580,   5.1
"Quebec/Québec","Beauport--Limoilou","24008","Raymond Côté ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Québec, Que./ Qc","Parliamentarian/Parlementaire",12881,25.5,,
"Quebec/Québec","Beauport--Limoilou","24008","Antoine Bujold Liberal/Libéral","Boischatel, Que./ Qc","Restaurateur/Restaurateur",12854,25.4,,
"Quebec/Québec","Beauport--Limoilou","24008","Doni Berberi Bloc Québécois/Bloc Québécois","Québec, Que./ Qc","Administrator/Administrateur",7467,14.8,,
"Quebec/Québec","Beauport--Limoilou","24008","Dalila Elhak Green Party/Parti Vert","Québec, Que./ Qc","Real Estate Broker/Courtière immobilière",1220,2.4,,
"Quebec/Québec","Beauport--Limoilou","24008","Francis Bedard Libertarian/Libertarien","Québec, Que./ Qc","Retired/Retraité",423,.8,,
"Quebec/Québec","Beauport--Limoilou","24008","Claude Moreau Marxist-Leninist/Marxiste-Léniniste","Québec, Que./ Qc","Stationary Mechanic/Mécanicien de machines fixes",128,.3,,
"Quebec/Québec","Beauport--Limoilou","24008","Bladimir Laborit Forces et Démocratie - Allier les forces de nos régions/Forces et Démocratie - Allier les forces de nos régions","Québec, Que./ Qc","Assistant Manager/Assistant gérant",124,.2,,
"Quebec/Québec","Bécancour--Nicolet--Saurel","24009","Louis Plamondon ** Bloc Québécois/Bloc Québécois","Sorel-Tracy, Que./ Qc","Parliamentarian/Parlementaire",20871,40,      8205,  15.7
"Quebec/Québec","Bécancour--Nicolet--Saurel","24009","Claude Carpentier Liberal/Libéral","Sorel-Tracy, Que./ Qc","Retired/Retraité",12666,24.3,,
"Quebec/Québec","Bécancour--Nicolet--Saurel","24009","Nicolas Tabah NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Sorel-Tracy, Que./ Qc","Telecommunications Trainer /Formateur en télécommunications",11531,22.1,,
"Quebec/Québec","Bécancour--Nicolet--Saurel","24009","Yves Laberge Conservative/Conservateur","Québec, Que./ Qc","Part-time Professor/Professeur à temps partiel",5955,11.4,,
"Quebec/Québec","Bécancour--Nicolet--Saurel","24009","Corina Bastiani Green Party/Parti Vert","Sorel-Tracy, Que./ Qc","Consultant/Consultante",1182,2.3,,
"Quebec/Québec","Bellechasse--Les Etchemins--Lévis","24010","Steven Blaney ** Conservative/Conservateur","Lévis, Que./ Qc","Parliamentarian/Parlementaire",31872,50.9,     18911,  30.2
"Quebec/Québec","Bellechasse--Les Etchemins--Lévis","24010","Jacques Turgeon Liberal/Libéral","Lévis, Que./ Qc","Producer/Producteur",12961,20.7,,
"Quebec/Québec","Bellechasse--Les Etchemins--Lévis","24010","Jean-Luc Daigle NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Lévis, Que./ Qc","Retired/Retraité",8516,13.6,,
"Quebec/Québec","Bellechasse--Les Etchemins--Lévis","24010","Antoine Dubé Bloc Québécois/Bloc Québécois","Lévis, Que./ Qc","Recreologist/Récréologue",7217,11.5,,
"Quebec/Québec","Bellechasse--Les Etchemins--Lévis","24010","André Bélisle Green Party/Parti Vert","Frampton, Que./ Qc","Environmentalist/Environnementaliste",2032,3.2,,
"Quebec/Québec","Beloeil--Chambly","24011","Matthew Dubé ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Saint-Basile-le-Grand, Que./ Qc","Parliamentarian/Parlementaire",20641,31.1,      1147,   1.7
"Quebec/Québec","Beloeil--Chambly","24011","Karine Desjardins Liberal/Libéral","Otterburn Park, Que./ Qc","Businesswoman/Femme d'affaires",19494,29.3,,
"Quebec/Québec","Beloeil--Chambly","24011","Yves Lessard Bloc Québécois/Bloc Québécois","Saint-Basile-le-Grand, Que./ Qc","Agricultural producer/Producteur agricole",18387,27.7,,
"Quebec/Québec","Beloeil--Chambly","24011","Claude Chalhoub Conservative/Conservateur","Delson, Que./ Qc","Fleet Manager/Gestionnaire de flotte",6173,9.3,,
"Quebec/Québec","Beloeil--Chambly","24011","Fodé Kerfalla Yansané Green Party/Parti Vert","Chambly, Que./ Qc","Urban Planning Consultant/Conseiller en aménagement",1498,2.3,,
"Quebec/Québec","Beloeil--Chambly","24011","Michael Maher Libertarian/Libertarien","Carignan, Que./ Qc","Sound Technician/Technicien en sonorisation",245,.4,,
"Quebec/Québec","Berthier--Maskinongé","24012","Ruth Ellen Brosseau ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Trois-Rivières-Ouest, Que./ Qc","Parliamentarian/Parlementaire",22942,42.2,      8905,  16.4
"Quebec/Québec","Berthier--Maskinongé","24012","Yves Perron Bloc Québécois/Bloc Québécois","Saint-Félix-de-Valois, Que./ Qc","Teacher/Enseignant",14037,25.8,,
"Quebec/Québec","Berthier--Maskinongé","24012","Pierre Destrempes Liberal/Libéral","Montréal, Que./ Qc","Real Estate Businessman/Homme d'affaires immobilières",11032,20.3,,
"Quebec/Québec","Berthier--Maskinongé","24012","Marianne Foucrault Conservative/Conservateur","Saint-Félix-de-Valois, Que./ Qc","Student/Étudiante",5548,10.2,,
"Quebec/Québec","Berthier--Maskinongé","24012","Cate Burton Green Party/Parti Vert","Halifax, N.S./ N.-É.","Student/Étudiante",847,1.6,,
"Quebec/Québec","Bourassa","24015","Emmanuel Dubourg ** Liberal/Libéral","Saint-Lambert, Que./ Qc","Chartered Professional Accountant/Comptable professionel agréé",22234,54.1,     15185,  36.9
"Quebec/Québec","Bourassa","24015","Gilles Léveillé Bloc Québécois/Bloc Québécois","Montréal, Que./ Qc","Retired/Retraité",7049,17.1,,
"Quebec/Québec","Bourassa","24015","Dolmine Laguerre NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Repentigny, Que./ Qc","Public Servant/Fonctionnaire",6144,14.9,,
"Quebec/Québec","Bourassa","24015","Jason Potasso-Justino Conservative/Conservateur","Montréal-Nord, Que./ Qc","Unemployed/Sans emploi",3819,9.3,,
"Quebec/Québec","Bourassa","24015","Maxime Charron Green Party/Parti Vert","Coquitlam, B.C./ C.-B.","Business Consulting/Consultant d'entreprises",886,2.2,,
"Quebec/Québec","Bourassa","24015","Julie Demers Independent/Indépendant","Montréal-Nord, Que./ Qc","Writer-Translator/Rédactrice-Traductrice",669,1.6,,
"Quebec/Québec","Bourassa","24015","Claude Brunelle Marxist-Leninist/Marxiste-Léniniste","Montréal, Que./ Qc","Retired/Retraité",229,.6,,
"Quebec/Québec","Bourassa","24015","Jean-Marie Floriant Ndzana Forces et Démocratie - Allier les forces de nos régions/Forces et Démocratie - Allier les forces de nos régions","Montréal, Que./ Qc","Strategy Consultant/Consultant en stratégies",99,.2,,
"Quebec/Québec","Brome--Missisquoi","24016","Denis Paradis Liberal/Libéral","Saint-Armand, Que./ Qc","Lawyer/Avocat",25744,43.9,     11361,  19.4
"Quebec/Québec","Brome--Missisquoi","24016","Catherine Lusson NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Bromont, Que./ Qc","Political Attaché/Attachée politique",14383,24.5,,
"Quebec/Québec","Brome--Missisquoi","24016","Patrick Melchior Bloc Québécois/Bloc Québécois","Farnham, Que./ Qc","Director Youth Centre/Directeur Maison des jeunes",10252,17.5,,
"Quebec/Québec","Brome--Missisquoi","24016","Charles Poulin Conservative/Conservateur","Magog, Que./ Qc","Financial Security Advisor/Conseiller en sécurité financière",6724,11.5,,
"Quebec/Québec","Brome--Missisquoi","24016","Cindy Moynan Green Party/Parti Vert","Knowlton, Que./ Qc","Computer Manufacturing Operator/Opératrice à la fabrication d'ordinateurs",1377,2.3,,
"Quebec/Québec","Brome--Missisquoi","24016","Patrick Paine Forces et Démocratie - Allier les forces de nos régions/Forces et Démocratie - Allier les forces de nos régions","Brigham, Que./ Qc","Consultant/Consultant",195,.3,,
"Quebec/Québec","Brossard--Saint-Lambert","24017","Alexandra Mendès Liberal/Libéral","Brossard, Que./ Qc","Politician/Politicien",28818,50.3,     14743,  25.7
"Quebec/Québec","Brossard--Saint-Lambert","24017","Hoang Mai ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Brossard, Que./ Qc","Parliamentarian/Parlementaire",14075,24.6,,
"Quebec/Québec","Brossard--Saint-Lambert","24017","Qais Hamidi Conservative/Conservateur","Brossard, Que./ Qc","Businessman/Homme d'affaires",7215,12.6,,
"Quebec/Québec","Brossard--Saint-Lambert","24017","Suzanne Lachance Bloc Québécois/Bloc Québécois","Longueuil, Que./ Qc","Retired/Retraitée",6071,10.6,,
"Quebec/Québec","Brossard--Saint-Lambert","24017","Fang Hu Green Party/Parti Vert","Montréal, Que./ Qc","Engineer/Ingénieur",1081,1.9,,
"Quebec/Québec","Charlesbourg--Haute-Saint-Charles","24019","Pierre Paul-Hus Conservative/Conservateur","Québec, Que./ Qc","Publisher/Éditeur",24608,42.2,     11083,  19.0
"Quebec/Québec","Charlesbourg--Haute-Saint-Charles","24019","Jean Côté Liberal/Libéral","Québec, Que./ Qc","Marketing and Communications Consultant/Consultant en marketing et communication",13525,23.2,,
"Quebec/Québec","Charlesbourg--Haute-Saint-Charles","24019","Anne-Marie Day ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Québec, Que./ Qc","Parliamentarian/Parlementaire",11690,20.1,,
"Quebec/Québec","Charlesbourg--Haute-Saint-Charles","24019","Marc Antoine Turmel Bloc Québécois/Bloc Québécois","Québec, Que./ Qc","Student/Étudiant",7177,12.3,,
"Quebec/Québec","Charlesbourg--Haute-Saint-Charles","24019","Nathalie Baudet Green Party/Parti Vert","Varennes, Que./ Qc","Housewife/Mère au foyer",1256,2.2,,
"Quebec/Québec","Châteauguay--Lacolle","24021","Brenda Shanahan Liberal/Libéral","Châteauguay, Que./ Qc","Financial Educator/Éducatrice Financière",20245,39.1,      7630,  14.7
"Quebec/Québec","Châteauguay--Lacolle","24021","Sophie Stanké Bloc Québécois/Bloc Québécois","Greenfield Park, Que./ Qc","Instructor/Animatrice",12615,24.4,,
"Quebec/Québec","Châteauguay--Lacolle","24021","Sylvain Chicoine ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Delson, Que./ Qc","Parlementarian/Parlementaire",11986,23.1,,
"Quebec/Québec","Châteauguay--Lacolle","24021","Philippe St-Pierre Conservative/Conservateur","Châteauguay, Que./ Qc","Municipal Officer/Fonctionnaire municipal",5805,11.2,,
"Quebec/Québec","Châteauguay--Lacolle","24021","Jency Mercier Green Party/Parti Vert","Verdun, Que./ Qc","STM Driver/Chauffeuse STM",982,1.9,,
"Quebec/Québec","Châteauguay--Lacolle","24021","Linda Sullivan Marxist-Leninist/Marxiste-Léniniste","Montréal, Que./ Qc","Project Manager/Gestionnaire de projets",149,.3,,
"Quebec/Québec","Chicoutimi--Le Fjord","24022","Denis Lemieux Liberal/Libéral","Chicoutimi, Que./ Qc","Retired/Retraité",13619,31.1,       600,   1.4
"Quebec/Québec","Chicoutimi--Le Fjord","24022","Dany Morin ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Saint-Fulgence, Que./ Qc","Parliamentarian/Parlementaire",13019,29.7,,
"Quebec/Québec","Chicoutimi--Le Fjord","24022","Élise Gauthier Bloc Québécois/Bloc Québécois","Chicoutimi, Que./ Qc","Retired/Retraitée",8990,20.5,,
"Quebec/Québec","Chicoutimi--Le Fjord","24022","Caroline Ste-Marie Conservative/Conservateur","Chicoutimi, Que./ Qc","Lawyer/Avocate",7270,16.6,,
"Quebec/Québec","Chicoutimi--Le Fjord","24022","Dany St-Gelais Green Party/Parti Vert","Chicoutimi, Que./ Qc","Music Teacher/Professeur de musique",907,2.1,,
"Quebec/Québec","Compton--Stanstead","24023","Marie-Claude Bibeau Liberal/Libéral","Sherbrooke, Que./ Qc","Manager/Gestionnaire",20582,36.9,      5282,   9.5
"Quebec/Québec","Compton--Stanstead","24023","Jean Rousseau ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Georgeville, Que./ Qc","Parliamentarian/Parlementaire",15300,27.4,,
"Quebec/Québec","Compton--Stanstead","24023","France Bonsant Bloc Québécois/Bloc Québécois","Sherbrooke, Que./ Qc","Retired/Retraitée",11551,20.7,,
"Quebec/Québec","Compton--Stanstead","24023","Gustavo Labrador Conservative/Conservateur","Sherbrooke, Que./ Qc","Communications Operator/Préposé aux communications",6978,12.5,,
"Quebec/Québec","Compton--Stanstead","24023","Korie Marshall Green Party/Parti Vert","Valemount, B.C./ C.-B.","Editor/Journalist/Rédactrice/journaliste",1085,1.9,,
"Quebec/Québec","Compton--Stanstead","24023","Kévin Côté Rhinoceros/Rhinocéros","Compton, Que./ Qc","Teacher/Enseignant",315,.6,,
"Quebec/Québec","Dorval--Lachine--LaSalle","24024","Anju Dhillon Liberal/Libéral","LaSalle, Que./ Qc","Lawyer/Avocat",29974,54.9,     18205,  33.3
"Quebec/Québec","Dorval--Lachine--LaSalle","24024","Isabelle Morin ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Lachine, Que./ Qc","Parliamentarian/Parlementaire",11769,21.6,,
"Quebec/Québec","Dorval--Lachine--LaSalle","24024","Daniela Chivu Conservative/Conservateur","Montréal, Que./ Qc","Administrator/Administratrice",6049,11.1,,
"Quebec/Québec","Dorval--Lachine--LaSalle","24024","Jean-Frédéric Vaudry Bloc Québécois/Bloc Québécois","Lachine, Que./ Qc","Student/Étudiant",5338,9.8,,
"Quebec/Québec","Dorval--Lachine--LaSalle","24024","Vincent J. Carbonneau Green Party/Parti Vert","Montréal, Que./ Qc","Student/Étudiant",1245,2.3,,
"Quebec/Québec","Dorval--Lachine--LaSalle","24024","Soulèye Ndiaye Independent/Indépendant","Sainte-Catherine, Que./ Qc","Financial Agent/Agent financier",230,.4,,
"Quebec/Québec","Drummond","24025","François Choquette ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Drummondville, Que./ Qc","Teacher/Enseignant",15833,30.5,      2040,   3.9
"Quebec/Québec","Drummond","24025","Pierre Côté Liberal/Libéral","Drummondville, Que./ Qc","Self-employed/Travailleur autonome",13793,26.5,,
"Quebec/Québec","Drummond","24025","Diane Bourgeois Bloc Québécois/Bloc Québécois","Saint-Lucien, Que./ Qc","Retired/Retraitée",11862,22.8,,
"Quebec/Québec","Drummond","24025","Pascale Déry Conservative/Conservateur","Côte-Saint-Luc, Que./ Qc","Journalist/Journaliste",9221,17.7,,
"Quebec/Québec","Drummond","24025","Émile Coderre Green Party/Parti Vert","Saint-Germain, Que./ Qc","Student/Étudiant",1270,2.4,,
"Quebec/Québec","Gaspésie--Les Îles-de-la-Madeleine","24026","Diane Lebouthillier Liberal/Libéral","Sainte-Thérèse-de-Gaspé, Que./ Qc","Councillor/Élu municipal",15345,38.7,      2460,   6.2
"Quebec/Québec","Gaspésie--Les Îles-de-la-Madeleine","24026","Philip Toone ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Maria, Que./ Qc","Notary/Notaire",12885,32.5,,
"Quebec/Québec","Gaspésie--Les Îles-de-la-Madeleine","24026","Nicolas Roussy Bloc Québécois/Bloc Québécois","Québec, Que./ Qc","Communications and Public Management Consultant/Consultant en communications et gestion publique",8289,20.9,,
"Quebec/Québec","Gaspésie--Les Îles-de-la-Madeleine","24026","Jean-Pierre Pigeon Conservative/Conservateur","Sainte-Anne-des-Monts, Que./ Qc","Financial Services Advisor/Conseiller en services financiers",2398,6.1,,
"Quebec/Québec","Gaspésie--Les Îles-de-la-Madeleine","24026","Jim Morrison Green Party/Parti Vert","Sainte-Anne-des-Monts, Que./ Qc","Retired Consultant/Consultant retraité",400,1,,
"Quebec/Québec","Gaspésie--Les Îles-de-la-Madeleine","24026","Max Boudreau Rhinoceros/Rhinocéros","Gaspé, Que./ Qc","Fisherman/Pêcheur",300,.8,,
"Quebec/Québec","Gatineau","24027","Steven MacKinnon Liberal/Libéral","Gatineau, Que./ Qc","Manager/Gestionnaire",31076,53.8,     15724,  27.2
"Quebec/Québec","Gatineau","24027","Françoise Boivin ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Gatineau, Que./ Qc","Parliamentarian/Parlementaire",15352,26.6,,
"Quebec/Québec","Gatineau","24027","Philippe Boily Bloc Québécois/Bloc Québécois","Gatineau, Que./ Qc","Financial Education Advisor/Conseiller en éducation financière",5455,9.4,,
"Quebec/Québec","Gatineau","24027","Luc Angers Conservative/Conservateur","Gatineau, Que./ Qc","Teacher/Enseignant",4733,8.2,,
"Quebec/Québec","Gatineau","24027","Guy Dostaler Green Party/Parti Vert","Val-des-Monts, Que./ Qc","Carpenter/Charpentier",942,1.6,,
"Quebec/Québec","Gatineau","24027","Guy J. Bellavance Independent/Indépendant","Gatineau, Que./ Qc","Federal Public Servant/Fonctionnaire fédéral",148,.3,,
"Quebec/Québec","Gatineau","24027","Pierre Soublière Marxist-Leninist/Marxiste-Léniniste","Gatineau, Que./ Qc","Teacher/Enseignant",94,.2,,
"Quebec/Québec","Hochelaga","24028","Marjolaine Boutin-Sweet ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Montréal, Que./ Qc","Parliamentarian/Parlementaire",16034,30.9,       500,   1.0
"Quebec/Québec","Hochelaga","24028","Marwah Rizqy Liberal/Libéral","Montréal, Que./ Qc","Taxation Professor/Professeure de Fiscalité",15534,29.9,,
"Quebec/Québec","Hochelaga","24028","Simon Marchand Bloc Québécois/Bloc Québécois","Montréal, Que./ Qc","Manager - Fraud Prevention/Gestionnaire - Prévention fraude",14389,27.7,,
"Quebec/Québec","Hochelaga","24028","Alexandre Dang Conservative/Conservateur","Gatineau, Que./ Qc","Student/Étudiant",3555,6.8,,
"Quebec/Québec","Hochelaga","24028","Anne-Marie Saint-Cerny Green Party/Parti Vert","Val-David, Que./ Qc","Environmentalist/Environnementaliste",1654,3.2,,
"Quebec/Québec","Hochelaga","24028","Nicolas Lemay Rhinoceros/Rhinocéros","Montréal, Que./ Qc","Cook/Cuisinier",411,.8,,
"Quebec/Québec","Hochelaga","24028","Marianne Breton Fontaine Communist/Communiste","Montréal, Que./ Qc","Student/Étudiante",179,.3,,
"Quebec/Québec","Hochelaga","24028","Christine Dandenault Marxist-Leninist/Marxiste-Léniniste","Montréal, Que./ Qc","Administrative Assistant/Adjointe administrative",148,.3,,
"Quebec/Québec","Honoré-Mercier","24029","Pablo Rodriguez Liberal/Libéral","Montréal, Que./ Qc","Administrator/Administrateur",29211,56.5,     20733,  40.1
"Quebec/Québec","Honoré-Mercier","24029","Paulina Ayala ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Montréal, Que./ Qc","History and Geography Professor/Professeure d'histoire et de géographie",8478,16.4,,
"Quebec/Québec","Honoré-Mercier","24029","Audrey Beauséjour Bloc Québécois/Bloc Québécois","Montréal, Que./ Qc","Desk Officer/Agente de bureau",6680,12.9,,
"Quebec/Québec","Honoré-Mercier","24029","Guy Croteau Conservative/Conservateur","Montréal, Que./ Qc","Professional Chartered Accountant/Comptable professionnel agréé",6226,12.1,,
"Quebec/Québec","Honoré-Mercier","24029","Angela Budilean Green Party/Parti Vert","Sherbrooke, Que./ Qc","Mechanical Technician/Technicien en mécanique",814,1.6,,
"Quebec/Québec","Honoré-Mercier","24029","Dayana Dejean Forces et Démocratie - Allier les forces de nos régions/Forces et Démocratie - Allier les forces de nos régions","Repentigny, Que./ Qc","Mortgage Advisor/Conseillère hypotécaire",168,.3,,
"Quebec/Québec","Honoré-Mercier","24029","Yves Le Seigle Marxist-Leninist/Marxiste-Léniniste","Montréal, Que./ Qc","Home Maintenance/Entretien ménager",81,.2,,
"Quebec/Québec","Hull--Aylmer","24030","Greg Fergus Liberal/Libéral","Gatineau, Que./ Qc","Consultant/Consultant",28478,51.4,     11006,  19.9
"Quebec/Québec","Hull--Aylmer","24030","Nycole Turmel ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Gatineau, Que./ Qc","Parliamentarian/Parlementaire",17472,31.5,,
"Quebec/Québec","Hull--Aylmer","24030","Étienne Boulrice Conservative/Conservateur","Gatineau, Que./ Qc","Special Assistant/Driver/Adjoint spécial-Chauffeur",4278,7.7,,
"Quebec/Québec","Hull--Aylmer","24030","Maude Chouinard-Boucher Bloc Québécois/Bloc Québécois","Gatineau, Que./ Qc","Student/Étudiante",3625,6.5,,
"Quebec/Québec","Hull--Aylmer","24030","Roger Fleury Green Party/Parti Vert","Gatineau, Que./ Qc","Retired Teacher/Enseignant retraité",1035,1.9,,
"Quebec/Québec","Hull--Aylmer","24030","Sean J. Mulligan Christian Heritage Party/Parti de l'Héritage Chrétien","Aylmer, Que./ Qc","Federal Government Employee (Passport)/Employé du gouvernement fédéral (Passeport)",291,.5,,
"Quebec/Québec","Hull--Aylmer","24030","Luc Desjardins Independent/Indépendant","Gatineau, Que./ Qc","Automotive Mechanic/Technicien garagiste automobile",160,.3,,
"Quebec/Québec","Hull--Aylmer","24030","Gabriel Girard-Bernier Marxist-Leninist/Marxiste-Léniniste","Gatineau, Que./ Qc","Wine Steward/Sommelier",101,.2,,
"Quebec/Québec","Joliette","24031","Gabriel Ste-Marie Bloc Québécois/Bloc Québécois","Joliette, Que./ Qc","Economist/Économiste",18875,33.3,      2880,   5.1
"Quebec/Québec","Joliette","24031","Michel Bourgeois Liberal/Libéral","Joliette, Que./ Qc","Storekeeper / Administrator/Commerçant / Administrateur",15995,28.2,,
"Quebec/Québec","Joliette","24031","Danielle Landreville NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Joliette, Que./ Qc","Retired/Retraitée",14566,25.7,,
"Quebec/Québec","Joliette","24031","Soheil Eid Conservative/Conservateur","Joliette, Que./ Qc","Pulmonologist/Médecin pneumologue ",5705,10.1,,
"Quebec/Québec","Joliette","24031","Mathieu Morin Green Party/Parti Vert","Notre-Dame-des-Prairies, Que./ Qc","Student/Étudiant",1335,2.4,,
"Quebec/Québec","Joliette","24031","Robert D. Morais Forces et Démocratie - Allier les forces de nos régions/Forces et Démocratie - Allier les forces de nos régions","Saint-Alphonse-Rodriguez, Que./ Qc","Filmmaker/Cinéaste",213,.4,,
"Quebec/Québec","Jonquière","24032","Karine Trudel NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Saguenay, Que./ Qc","Canada Post Letter Carrier/Factrice à Postes Canada",14039,29.2,       339,   0.7
"Quebec/Québec","Jonquière","24032","Marc Pettersen Liberal/Libéral","Chicoutimi, Que./ Qc","Menuisier/Carpenter",13700,28.5,,
"Quebec/Québec","Jonquière","24032","Jean-François Caron Bloc Québécois/Bloc Québécois","Jonquière, Que./ Qc","Political Aide/Attaché politique",11202,23.3,,
"Quebec/Québec","Jonquière","24032","Ursula Larouche Conservative/Conservateur","Jonquière, Que./ Qc","Biologist/Biologiste",8124,16.9,,
"Quebec/Québec","Jonquière","24032","Carmen Budilean Green Party/Parti Vert","Fredericton, N.B./ N.-B.","Organizer (Management)/Organisatrice (gestion)",656,1.4,,
"Quebec/Québec","Jonquière","24032","Marielle Couture Rhinoceros/Rhinocéros","Sainte-Rose-du-Nord, Que./ Qc","Graphic Designer/Designer graphique",382,.8,,
"Quebec/Québec","La Pointe-de-l'Île","24033","Mario Beaulieu Bloc Québécois/Bloc Québécois","Pointe-aux-Trembles, Que./ Qc","President/Educator/Président-Éducateur",18545,33.6,      2768,   5.0
"Quebec/Québec","La Pointe-de-l'Île","24033","Marie-Chantale Simard Liberal/Libéral","Montréal, Que./ Qc","Scientific-Medical Liaison/Liaison médicale scientifique",15777,28.6,,
"Quebec/Québec","La Pointe-de-l'Île","24033","Ève Péclet ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Montréal, Que./ Qc","Parlementarian/Parlementaire",14777,26.8,,
"Quebec/Québec","La Pointe-de-l'Île","24033","Guy Morissette Conservative/Conservateur","Saint-Laurent, Que./ Qc","Specialist, Member Services at ICCRC /Spécialiste, service aux membres au CRCIC",4408,8,,
"Quebec/Québec","La Pointe-de-l'Île","24033","David J. Cox Green Party/Parti Vert","Montréal, Que./ Qc","Writer/Écrivain",1130,2,,
"Quebec/Québec","La Pointe-de-l'Île","24033","Ben 97 Benoit Rhinoceros/Rhinocéros","Montréal, Que./ Qc","Television cameraman/Caméraman de télévision ",358,.6,,
"Quebec/Québec","La Pointe-de-l'Île","24033","Jean-François Larose Forces et Démocratie - Allier les forces de nos régions/Forces et Démocratie - Allier les forces de nos régions","L'Assomption, Que./ Qc","Parliamentarian/Parlementaire",135,.2,,
"Quebec/Québec","La Pointe-de-l'Île","24033","Geneviève Royer Marxist-Leninist/Marxiste-Léniniste","Montréal, Que./ Qc","Teacher/Enseignante",96,.2,,
"Quebec/Québec","La Prairie","24034","Jean-Claude Poissant Liberal/Libéral","Saint-Philippe, Que./ Qc","Farmer/Agriculteur",20993,36.5,      5886,  10.2
"Quebec/Québec","La Prairie","24034","Christian Picard Bloc Québécois/Bloc Québécois","Chambly, Que./ Qc","Consultant/Consultant",15107,26.2,,
"Quebec/Québec","La Prairie","24034","Pierre Chicoine NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Sainte-Catherine, Que./ Qc","Supervisor/Superviseur",13174,22.9,,
"Quebec/Québec","La Prairie","24034","Yves Perras Conservative/Conservateur","Saint-Constant, Que./ Qc","Real Estate Agent/Courtier immobilier",6859,11.9,,
"Quebec/Québec","La Prairie","24034","Joanne Tomas Green Party/Parti Vert","Brossard, Que./ Qc","Market Research/Étude de marché",1235,2.1,,
"Quebec/Québec","La Prairie","24034","Normand Chouinard Marxist-Leninist/Marxiste-Léniniste","Saint-Philippe-de-La Prairie, Que./ Qc","Truck Driver/Camionneur",204,.4,,
"Quebec/Québec","Lac-Saint-Jean","24035","Denis Lebel ** Conservative/Conservateur","Roberval, Que./ Qc","Parliamentarian/Parlementaire",18393,33.3,      2658,   4.8
"Quebec/Québec","Lac-Saint-Jean","24035","Gisèle Dallaire NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Alma, Que./ Qc","Industrial Psychologist/Psychologue industrielle",15735,28.5,,
"Quebec/Québec","Lac-Saint-Jean","24035","Sabin Simard Liberal/Libéral","Alma, Que./ Qc","Consultant/Consultant",10193,18.4,,
"Quebec/Québec","Lac-Saint-Jean","24035","Sabin Gaudreault Bloc Québécois/Bloc Québécois","Alma, Que./ Qc","Retired/Retraité",10152,18.4,,
"Quebec/Québec","Lac-Saint-Jean","24035","Laurence Requilé Green Party/Parti Vert","Saint-Paulin, Que./ Qc","Florist/Fleuriste",806,1.5,,
"Quebec/Québec","Lac-Saint-Louis","24036","Francis Scarpaleggia ** Liberal/Libéral","Kirkland, Que./ Qc","Parliamentarian/Parlementaire",39965,64.1,     29108,  46.7
"Quebec/Québec","Lac-Saint-Louis","24036","Eric Girard Conservative/Conservateur","Mont-Royal, Que./ Qc","Treasurer/Trésorier",10857,17.4,,
"Quebec/Québec","Lac-Saint-Louis","24036","Ryan Young NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Sainte-Anne-de-Bellevue, Que./ Qc","Teacher/Enseignant",7997,12.8,,
"Quebec/Québec","Lac-Saint-Louis","24036","Bradford Dean Green Party/Parti Vert","Montréal, Que./ Qc","Community Worker/Travailleur communautaire",1812,2.9,,
"Quebec/Québec","Lac-Saint-Louis","24036","Gabriel Bernier Bloc Québécois/Bloc Québécois","Montréal, Que./ Qc","Student/Étudiant",1681,2.7,,
"Quebec/Québec","LaSalle--Émard--Verdun","24037","David Lametti Liberal/Libéral","Montréal, Que./ Qc","Professor, McGill University/Professeur, Université McGill",23603,43.9,      8037,  14.9
"Quebec/Québec","LaSalle--Émard--Verdun","24037","Hélène LeBlanc ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Montréal, Que./ Qc","Parliamentarian/Agrologist/Parlementaire-Agronome",15566,29,,
"Quebec/Québec","LaSalle--Émard--Verdun","24037","Gilbert Paquette Bloc Québécois/Bloc Québécois","Sainte-Béatrix, Que./ Qc","Teacher and Director/Professeur et directeur",9164,17,,
"Quebec/Québec","LaSalle--Émard--Verdun","24037","Mohammad Zamir Conservative/Conservateur","LaSalle, Que./ Qc","Contractor/Entrepreneur",3713,6.9,,
"Quebec/Québec","LaSalle--Émard--Verdun","24037","Lorraine Banville Green Party/Parti Vert","Montréal, Que./ Qc","Oceanographer/Scientific Communicator/Océanographe-Communicatrice scientifique",1717,3.2,,
"Quebec/Québec","Laurentides--Labelle","24038","David Graham Liberal/Libéral","Sainte-Lucie-des-Laurentides, Que./ Qc","Parliamentary Assistant/Adjoint parlementaire",20277,32.1,      1485,   2.4
"Quebec/Québec","Laurentides--Labelle","24038","Johanne Régimbald Bloc Québécois/Bloc Québécois","Mont-Tremblant, Que./ Qc","Communication Self-employed Worker/Travailleuse autonome en communication",18792,29.7,,
"Quebec/Québec","Laurentides--Labelle","24038","Simon-Pierre Landry NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Sainte-Agathe-des-Monts, Que./ Qc","Doctor/Médecin",16644,26.3,,
"Quebec/Québec","Laurentides--Labelle","24038","Sylvain Charron Conservative/Conservateur","Sainte-Anne-des-Lacs, Que./ Qc","Tax Expert/Spécialiste en fiscalité",6209,9.8,,
"Quebec/Québec","Laurentides--Labelle","24038","Niloufar Hedjazi Green Party/Parti Vert","Montréal, Que./ Qc","Computer Engineer/Ingénieure en informatique",1251,2,,
"Quebec/Québec","Laurier--Sainte-Marie","24039","Hélène Laverdière ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Montréal, Que./ Qc","Parliamentarian/Parlementaire",20929,38.3,      5230,   9.6
"Quebec/Québec","Laurier--Sainte-Marie","24039","Gilles Duceppe Bloc Québécois/Bloc Québécois","Montréal, Que./ Qc","Political Commentator/Commentateur politique",15699,28.7,,
"Quebec/Québec","Laurier--Sainte-Marie","24039","Christine Poirier Liberal/Libéral","Montréal, Que./ Qc","Contractor/Entrepreneur",12938,23.7,,
"Quebec/Québec","Laurier--Sainte-Marie","24039","Daniel Gaudreau Conservative/Conservateur","Montréal, Que./ Qc","Security Guard/Gardien de sécurité",2242,4.1,,
"Quebec/Québec","Laurier--Sainte-Marie","24039","Cyrille Giraud Green Party/Parti Vert","Montréal, Que./ Qc","Compliance Auditor/Vérificateur en conformité",1904,3.5,,
"Quebec/Québec","Laurier--Sainte-Marie","24039","Stéphane Beaulieu Libertarian/Libertarien","Ottawa, Ont.","Foreign Service Officer/Agent du service extérieur",604,1.1,,
"Quebec/Québec","Laurier--Sainte-Marie","24039","Julien Bernatchez Independent/Indépendant","Montréal, Que./ Qc","Poet/Poète",160,.3,,
"Quebec/Québec","Laurier--Sainte-Marie","24039","Serge Lachapelle Marxist-Leninist/Marxiste-Léniniste","Montréal, Que./ Qc","Journalist/Journaliste",103,.2,,
"Quebec/Québec","Laurier--Sainte-Marie","24039","Pierre Fontaine Communist/Communiste","Montréal, Que./ Qc","Hospital Worker/Travailleur d'hôpital",102,.2,,
"Quebec/Québec","Laval--Les Îles","24040","Fayçal El-Khoury Liberal/Libéral","Laval, Que./ Qc","Retired Engineer/Ingénieur retraité",25857,47.7,     15147,  27.9
"Quebec/Québec","Laval--Les Îles","24040","François Pilon ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Laval, Que./ Qc","Parliamentarian/Parlementaire",10710,19.8,,
"Quebec/Québec","Laval--Les Îles","24040","Roland Dick Conservative/Conservateur","Laval, Que./ Qc","Architecture and construction/Architecture et construction",9811,18.1,,
"Quebec/Québec","Laval--Les Îles","24040","Nancy Redhead Bloc Québécois/Bloc Québécois","Laval, Que./ Qc","Businesswoman/Femme d'affaires",6731,12.4,,
"Quebec/Québec","Laval--Les Îles","24040","Faiza R'Guiba-Kalogerakis Green Party/Parti Vert","Montréal, Que./ Qc","Researcher-Lecturer/Chercheuse-Conférencière",921,1.7,,
"Quebec/Québec","Laval--Les Îles","24040","Yvon Breton Marxist-Leninist/Marxiste-Léniniste","Montréal, Que./ Qc","Translator/Traducteur",175,.3,,
"Quebec/Québec","Lévis--Lotbinière","24042","Jacques Gourde ** Conservative/Conservateur","Saint-Narcisse-de-Beaurivage, Que./ Qc","Parlementarian/Parlementaire",31357,50.1,     17795,  28.4
"Quebec/Québec","Lévis--Lotbinière","24042","Claude Boucher Liberal/Libéral","Saint-Antoine-de-Tilly, Que./ Qc","Consultant/Consultant",13562,21.7,,
"Quebec/Québec","Lévis--Lotbinière","24042","Hélène Bilodeau NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Lévis, Que./ Qc","Retired/Retraitée",9246,14.8,,
"Quebec/Québec","Lévis--Lotbinière","24042","Steve Gagné Bloc Québécois/Bloc Québécois","Québec, Que./ Qc","Investment Advisor/Conseiller en placements",7163,11.4,,
"Quebec/Québec","Lévis--Lotbinière","24042","Tina Biello Green Party/Parti Vert","Nanoose Bay, B.C./ C.-B.","Coordinator/Coordinatrice",1124,1.8,,
"Quebec/Québec","Lévis--Lotbinière","24042","François Belanger ATN/ADN","Lévis, Que./ Qc","Environment/Environnement",136,.2,,
"Quebec/Québec","Longueuil--Charles-LeMoyne","24041","Sherry Romanado Liberal/Libéral","Greenfield Park, Que./ Qc","College Administrator and Faculty Lecturer/Administrateur collégial et chargée de cours",18301,35.4,      4327,   8.4
"Quebec/Québec","Longueuil--Charles-LeMoyne","24041","Philippe Cloutier Bloc Québécois/Bloc Québécois","Longueuil, Que./ Qc","Teacher/Enseignant",13974,27,,
"Quebec/Québec","Longueuil--Charles-LeMoyne","24041","Sadia Groguhé ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Longueuil, Que./ Qc","Parliamentarian/Parlementaire",12468,24.1,,
"Quebec/Québec","Longueuil--Charles-LeMoyne","24041","Thomas Barré Conservative/Conservateur","Saint-Lambert, Que./ Qc","Bachelor of Political Science/Bachelier en sciences politiques",4961,9.6,,
"Quebec/Québec","Longueuil--Charles-LeMoyne","24041","Mario Leclerc Green Party/Parti Vert","Greenfield Park, Que./ Qc","Fonctionnaire/Public Servant",1510,2.9,,
"Quebec/Québec","Longueuil--Charles-LeMoyne","24041","Matthew Iakov Liberman Rhinoceros/Rhinocéros","Longueuil, Que./ Qc","Student/Étudiant",325,.6,,
"Quebec/Québec","Longueuil--Charles-LeMoyne","24041","Pierre Chénier Marxist-Leninist/Marxiste-Léniniste","Montréal, Que./ Qc","Printer/Imprimeur",168,.3,,
"Quebec/Québec","Longueuil--Saint-Hubert","24043","Pierre Nantel ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Longueuil, Que./ Qc","Parliamentarian/Parlementaire",18171,31.2,       703,   1.2
"Quebec/Québec","Longueuil--Saint-Hubert","24043","Michael O'Grady Liberal/Libéral","Longueuil, Que./ Qc","Self-employed Worker/Travailleur autonome",17468,30,,
"Quebec/Québec","Longueuil--Saint-Hubert","24043","Denis Trudel Bloc Québécois/Bloc Québécois","Longueuil, Que./ Qc","Comedian/Comédien",15873,27.3,,
"Quebec/Québec","Longueuil--Saint-Hubert","24043","John Sedlak Conservative/Conservateur","Longueuil, Que./ Qc","Representative In-house/répresentant interne",5087,8.7,,
"Quebec/Québec","Longueuil--Saint-Hubert","24043","Casandra Poitras Green Party/Parti Vert","Saint-Hubert, Que./ Qc","Student/Étudiante",1447,2.5,,
"Quebec/Québec","Longueuil--Saint-Hubert","24043","Affine Lwalalika Forces et Démocratie - Allier les forces de nos régions/Forces et Démocratie - Allier les forces de nos régions","Saint-Hubert, Que./ Qc","International Investigations Agent/Agente aux investigations internationales",153,.3,,
"Quebec/Québec","Louis-Hébert","24044","Joël Lightbound Liberal/Libéral","Québec, Que./ Qc","Lawyer/Avocat",21516,34.8,      4727,   7.7
"Quebec/Québec","Louis-Hébert","24044","Jean-Pierre Asselin Conservative/Conservateur","Québec, Que./ Qc","Manufacturer agent/Agent manufacturier",16789,27.2,,
"Quebec/Québec","Louis-Hébert","24044","Denis Blanchette ** NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Québec, Que./ Qc","Parliamentarian/Parlementaire",12850,20.8,,
"Quebec/Québec","Louis-Hébert","24044","Caroline Pageau Bloc Québécois/Bloc Québécois","Québec, Que./ Qc","Translator/Traductrice",8900,14.4,,
"Quebec/Québec","Louis-Hébert","24044","Andrée-Anne Beaudoin-Julien Green Party/Parti Vert","Québec, Que./ Qc","Student/Étudiante",1561,2.5,,
"Quebec/Québec","Louis-Hébert","24044","Stefan Jetchick Christian Heritage Party/Parti de l'Héritage Chrétien","Québec, Que./ Qc","Translator/Traducteur",128,.2,,
"Quebec/Québec","Louis-Saint-Laurent","24045","Gérard Deltell Conservative/Conservateur","Québec, Que./ Qc","Annuitant/Rentier",32637,50.5,     18785,  29.0
"Quebec/Québec","Louis-Saint-Laurent","24045","Youri Rousseau Liberal/Libéral","Québec, Que./ Qc","Manager- DND/Gestionnaire-Défense Nationale",13852,21.4,,
"Quebec/Québec","Louis-Saint-Laurent","24045","G. Daniel Caron NDP-New Democratic Party/NPD-Nouveau Parti démocratique","Québec, Que./ Qc","Economist/Diplomat/Économiste-Diplomate",10296,15.9,,
"Quebec/Québec","Louis-Saint-Laurent","24045","Ronald Sirard Bloc Québécois/Bloc Québécois","Québec, Que./ Qc","Lawyer/Avocat",6688,10.3,,
"Quebec/Québec","Louis-Saint-Laurent","24045","Michel Savard Green Party/Parti Vert","Québec, Que./ Qc","Retired/Retraité",1210,1.9,,
"Quebec/Québec","Manicouagan","24046","Marilène Gill Bloc Québécois/Bloc Québécois","Ragueneau, Que./ Qc","College Teacher/Enseignante au collégial",17338,41.3,      4995,  11.9
"Quebec/Québec","Manicouagan","24046","Mario Tremblay Liberal/Libéral","Longue-Rive, Que./ Qc","Sales Representative/Représantant aux ventes",12343,29.4,,
```

</details>


`cat /home/admin/agent/check.sh`  
```bash
#!/usr/bin/bash
# DO NOT MODIFY THIS FILE ("Check My Solution" will fail)
cd /home/admin
expected_header=$(head -n 1 data.csv)
threshold=$((32 * 1024))
minlines=100
for i in {0..9}; do
    file="data-0$i.csv"
    if [[ -f "$file" ]]; then
        file_header=$(head -n 1 "$file")
        if [[ "$file_header" != "$expected_header" ]]; then
            echo -n "NO"
            exit
        fi
        filesize=$(stat -c%s "$file")
        if (( filesize > threshold )); then
            echo -n "NO"
            exit
        fi
        lines=$(wc -l < "$file")
        if (( lines < minlines )); then
            echo -n "NO"
            exit
        fi
    else
        echo -n "NO"
        exit
    fi
done
echo -n "OK"
```

#### 2. Make file with header
`head -1 /home/admin/data.csv > /home/admin/header.txt`  
`cat /home/admin/header.txt`  
```text
Province,Electoral District Name/Nom de circonscription,Electoral District Number/Numéro de circonscription,Candidate/Candidat,Candidate Residence/Résidence du candidat,Candidate Occupation/Profession du candidat,Votes Obtained/Votes obtenus,Percentage of Votes Obtained /Pourcentage des votes obtenus,Majority/Majorité,Majority Percentage/Pourcentage de majorité
```

#### 3. Split the `data.csv` file on 10 smaller files
`split -d -n 10 /home/admin/data.csv /home/admin/data-`  
`la -la`  
```console
total 668
drwxr-xr-x 5 admin admin   4096 Sep  7 16:27 .
drwxr-xr-x 3 root  root    4096 Feb 17  2024 ..
drwx------ 3 admin admin   4096 Feb 17  2024 .ansible
-rw-r--r-- 1 admin admin    220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 admin admin   3526 Mar 27  2022 .bashrc
-rw-r--r-- 1 admin admin    807 Mar 27  2022 .profile
drwx------ 2 admin admin   4096 Feb 17  2024 .ssh
-rw-r--r-- 1 admin admin    422 Jul 20 16:49 README.txt
drwxr-xr-x 2 admin root    4096 Jul 20 16:49 agent
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-00
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-01
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-02
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-03
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-04
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-05
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-06
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-07
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-08
-rw-r--r-- 1 admin admin  31276 Sep  7 16:27 data-09
-rw-r--r-- 1 admin admin 312715 Jul 20 16:49 data.csv
-rw-r--r-- 1 admin admin    372 Sep  7 16:26 header.txt
```

#### 4. Add header to all files except `data-00`
```bash
for i in {0..9}; do
    if [[ "$i" != "00" ]]; then
        cat /home/admin/header.txt /home/admin/data-0$i > /home/admin/data-0$i.csv
    fi
done
```

`ls -la`  
```console
total 988
drwxr-xr-x 5 admin admin   4096 Sep  7 16:27 .
drwxr-xr-x 3 root  root    4096 Feb 17  2024 ..
drwx------ 3 admin admin   4096 Feb 17  2024 .ansible
-rw-r--r-- 1 admin admin    220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 admin admin   3526 Mar 27  2022 .bashrc
-rw-r--r-- 1 admin admin    807 Mar 27  2022 .profile
drwx------ 2 admin admin   4096 Feb 17  2024 .ssh
-rw-r--r-- 1 admin admin    422 Jul 20 16:49 README.txt
drwxr-xr-x 2 admin root    4096 Jul 20 16:49 agent
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-00
-rw-r--r-- 1 admin admin  31643 Sep  7 16:27 data-00.csv
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-01
-rw-r--r-- 1 admin admin  31643 Sep  7 16:27 data-01.csv
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-02
-rw-r--r-- 1 admin admin  31643 Sep  7 16:27 data-02.csv
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-03
-rw-r--r-- 1 admin admin  31643 Sep  7 16:27 data-03.csv
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-04
-rw-r--r-- 1 admin admin  31643 Sep  7 16:27 data-04.csv
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-05
-rw-r--r-- 1 admin admin  31643 Sep  7 16:27 data-05.csv
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-06
-rw-r--r-- 1 admin admin  31643 Sep  7 16:27 data-06.csv
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-07
-rw-r--r-- 1 admin admin  31643 Sep  7 16:27 data-07.csv
-rw-r--r-- 1 admin admin  31271 Sep  7 16:27 data-08
-rw-r--r-- 1 admin admin  31643 Sep  7 16:27 data-08.csv
-rw-r--r-- 1 admin admin  31276 Sep  7 16:27 data-09
-rw-r--r-- 1 admin admin  31648 Sep  7 16:27 data-09.csv
-rw-r--r-- 1 admin admin 312715 Jul 20 16:49 data.csv
-rw-r--r-- 1 admin admin    372 Sep  7 16:26 header.txt
```


#### 5. Validate the task
`bash /home/admin/agent/check.sh`  
```console
OK
```

---

#### Clues
```console
1. Check the "split" command: man split
2. split -d -n 10 data.csv data- breaks the data file into 10 files data-00 to data-09 of 31KB each but only data-00 has the CSV header.
3. To add the header to all smaller files except data-00: first we get the header as a file: head -1 data.csv > header.txt and then we can do: for i in {0..9}; do cat header.txt data-0$i > data-0$i.csv; done
```
