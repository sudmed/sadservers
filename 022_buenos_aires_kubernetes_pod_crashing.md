# "Buenos Aires": Kubernetes Pod Crashing

**Description:** There are two brothers (pods) `logger` and `logshipper` living in the default namespace. Unfortunately, `logshipper` has an issue (crashlooping) and is forbidden to see what `logger` is trying to say. Could you help fix Logshipper? You can check the status of the pods with `kubectl get pods`.  

Do not change the K8S definition of the `logshipper` pod. Use "sudo".

**Test:** `kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].state.waiting.reason // .status.phase)"'` returns `Running`.  

