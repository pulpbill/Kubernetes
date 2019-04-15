### Bulk actions:

Delete pods with state not running:
```
for o in $(kubectl get pods -n <namespace> | grep -v NAME | grep -v Running | awk '{print $1}'); do kubectl delete pod $o -n <namespace>; done
```

Force object deletion by adding "--grace-period=0 --force" to kubectl delete command. 

Creating random secrets k8s (if you need it for test something):
```
for o in `seq 10000`; do kubectl create secret generic $o -n <namespace> ; done
```

Get the log from a running container by running cat against it:
```
kubectl exec <pod> -c <container> -n <namespace> -it -- cat /path/myapp/logs/log.log > /some-path/some-folder/log.log
```

Get Pod IP and which object owns it:
```
for o in $(kubectl get pods -n <namespace> | grep -v NAME | awk '{print $1}'); do kubectl describe pod $o -n <namespace> | grep IP -A1; done
```
