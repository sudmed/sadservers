## "Saint John": what is writing to this log file?

Description: A developer created a testing program that is continuously writing to a log file /var/log/bad.log and filling up disk. You can check for example with tail -f /var/log/bad.log.
This program is no longer needed. Find it and terminate it.  
Test: The log file size doesn't change (within a time interval bigger than the rate of change of the log file).  

```bash
kill -9 $(fuser -v /var/log/bad.log)
```

```bash
$ cat /home/admin/agent/check.sh
#!/usr/bin/bash

log_file="/var/log/bad.log"

if [ -f "$log_file" ]; then
    current_size=$(stat -c %s "$log_file")
    sleep 0.5
    new_size=$(stat -c %s "$log_file")

    if [ "$current_size" -eq "$new_size" ]; then
        echo -n "OK"
    else
        echo -n "NO"
    fi
else
    echo -n "NO"
fi
```
