# "Unimak Island": Fun with Mr Jason

**Description:** Using the file `station_information.json`, find the `station_id` where "has_kiosk" is false and "capacity" is greater than 30.  

Save the `station_id` of the solution in the `/home/admin/mysolution` file, for example: `echo "ec040a94-4de7-4fb3-aea0-ec5892034a69" > ~/mysolution`.  

You can use the installed utilities jq, gron, jid as well as Python3 and Golang.  


---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 1156
drwxr-xr-x 6 admin admin    4096 Sep 26 21:49 .
drwxr-xr-x 3 root  root     4096 Sep 17 16:44 ..
drwx------ 3 admin admin    4096 Sep 20 15:52 .ansible
-rw------- 1 admin admin     118 Jan 29 06:42 .bash_history
-rw-r--r-- 1 admin admin     220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin    3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin    4096 Sep 20 15:56 .config
-rw-r--r-- 1 admin admin     807 Aug  4  2021 .profile
drwx------ 2 admin admin    4096 Sep 17 16:44 .ssh
drwxr-xr-x 2 admin root     4096 Sep 26 21:49 agent
-rw-r--r-- 1 admin admin 1140013 Sep 26 21:49 station_information.json
```

`cat agent/check.sh `  
```console
#!/usr/bin/bash
res=$(md5sum /home/admin/mysolution |awk '{print $1}')
res=$(echo $res|tr -d '\r')

if [[ "$res" = "8d8414808b15d55dad857fd5aeb2aebc" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`cat station_information.json | jq | head -n 50`  
```console
{
  "data": {
    "stations": [
      {
        "eightd_has_key_dispenser": false,
        "rental_methods": [
          "KEY",
          "CREDITCARD"
        ],
        "external_id": "c00ef46d-fcde-48e2-afbd-0fb595fe3fa7",
        "station_id": "c00ef46d-fcde-48e2-afbd-0fb595fe3fa7",
        "rental_uris": {
          "ios": "https://bkn.lft.to/lastmile_qr_scan",
          "android": "https://bkn.lft.to/lastmile_qr_scan"
        },
        "region_id": "71",
        "capacity": 3,
        "short_name": "4920.13",
        "electric_bike_surcharge_waiver": false,
        "has_kiosk": true,
        "name": "South St & Broad St",
        "lon": -74.01089876890182,
        "lat": 40.70188858913366,
        "station_type": "classic",
        "eightd_station_services": []
      },
      {
        "eightd_has_key_dispenser": false,
        "rental_methods": [
          "KEY",
          "CREDITCARD"
        ],
        "external_id": "c638ec67-9ac0-416f-944f-619926144931",
        "station_id": "c638ec67-9ac0-416f-944f-619926144931",
        "rental_uris": {
          "ios": "https://bkn.lft.to/lastmile_qr_scan",
          "android": "https://bkn.lft.to/lastmile_qr_scan"
        },
        "region_id": "71",
        "capacity": 87,
        "short_name": "6432.11",
        "electric_bike_surcharge_waiver": false,
        "has_kiosk": true,
        "name": "E 40 St & Park Ave",
        "lon": -73.978326,
        "lat": 40.750756,
        "station_type": "classic",
        "eightd_station_services": []
      },
      {
```


#### 2. Sort objects
`cat station_information.json | jq -S --indent 2 > stasorted.json`  
`head -n 20 stasorted.json`  
```console
{
  "data": {
    "stations": [
      {
        "capacity": 3,
        "eightd_has_key_dispenser": false,
        "eightd_station_services": [],
        "electric_bike_surcharge_waiver": false,
        "external_id": "c00ef46d-fcde-48e2-afbd-0fb595fe3fa7",
        "has_kiosk": true,
        "lat": 40.70188858913366,
        "lon": -74.01089876890182,
        "name": "South St & Broad St",
        "region_id": "71",
        "rental_methods": [
          "KEY",
          "CREDITCARD"
        ],
        "rental_uris": {
          "android": "https://bkn.lft.to/lastmile_qr_scan",
```


#### 3. Filter for `data` field
`cat stasorted.json | jq '.data.stations' > data.stations.json`  
`head -n 20 data.stations.json`  
```console
[
  {
    "capacity": 3,
    "eightd_has_key_dispenser": false,
    "eightd_station_services": [],
    "electric_bike_surcharge_waiver": false,
    "external_id": "c00ef46d-fcde-48e2-afbd-0fb595fe3fa7",
    "has_kiosk": true,
    "lat": 40.70188858913366,
    "lon": -74.01089876890182,
    "name": "South St & Broad St",
    "region_id": "71",
    "rental_methods": [
      "KEY",
      "CREDITCARD"
    ],
    "rental_uris": {
      "android": "https://bkn.lft.to/lastmile_qr_scan",
      "ios": "https://bkn.lft.to/lastmile_qr_scan"
    },
```


#### 4. Select fields
`cat data.stations.json | jq '.[] | select(.has_kiosk == false and .capacity > 30)' > notkiosk.capacity.json`  
`cat notkiosk.capacity.json`  
```json
{
  "capacity": 32,
  "eightd_has_key_dispenser": false,
  "eightd_station_services": [],
  "electric_bike_surcharge_waiver": false,
  "external_id": "05c5e17c-7aa9-49b7-9da3-9db4858ec1fc",
  "has_kiosk": false,
  "lat": 40.75414519263519,
  "lon": -73.9960889518261,
  "name": "W 35 St & 9 Ave",
  "region_id": "71",
  "rental_methods": [
    "KEY",
    "CREDITCARD"
  ],
  "rental_uris": {
    "android": "https://bkn.lft.to/lastmile_qr_scan",
    "ios": "https://bkn.lft.to/lastmile_qr_scan"
  },
  "short_name": "6569.09",
  "station_id": "05c5e17c-7aa9-49b7-9da3-9db4858ec1fc",
  "station_type": "classic"
}
```


#### 5. Get 'station_id'
`cat notkiosk.capacity.json | jq '.station_id'`  
```console
"05c5e17c-7aa9-49b7-9da3-9db4858ec1fc"
```


#### 6. Set 'station_id' to file
`cat notkiosk.capacity.json | jq '.station_id' | tr -d '"' > /home/admin/mysolution`  
`cat /home/admin/mysolution`  
```console
05c5e17c-7aa9-49b7-9da3-9db4858ec1fc
```


#### 7. Validate the task
`md5sum /home/admin/mysolution`  
```console
8d8414808b15d55dad857fd5aeb2aebc  /home/admin/mysolution
```

`cd ~/agent && ./check.sh`  
```console
OK
```


---

### AUTHOR'S SOLUTION
`jq '.data.stations[] | select(.has_kiosk==false and .capacity>30) | .station_id' station_information.json`  
