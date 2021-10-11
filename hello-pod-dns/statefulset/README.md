## Using Service and StatefulSet
Check file [nginx-statefulset.yml](nginx-statefulset.yml) for configuration.

## Creation of PVC
Please note, `storageClassName: local-path` is required for PVC to be create on k3s as mentioned [here](https://rancher.com/docs/k3s/latest/en/storage/#pvc-yaml)

```
spec:
  volumeClaimTemplates:
    spec:
      storageClassName: local-path
```

## After creation of PVC and StatefulSet

```
kubectl get pvc -o wide
NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
local-path-pvc-web-0   Bound    pvc-23dd9dc6-4feb-41c1-b92a-72fcfb52239e   1Gi        RWO            local-path     91s   Filesystem
local-path-pvc-web-1   Bound    pvc-b5595cc5-cffc-41d6-a1fc-1b6c08eb7c2c   1Gi        RWO            local-path     27s   Filesystem

kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
web-0   1/1     Running   0          98s   10.42.1.109   raspiworker1   <none>           <none>
web-1   1/1     Running   0          34s   10.42.1.111   raspiworker1   <none>           <none>
```

## Auto-healing of pods in `kind: StatefulSet`