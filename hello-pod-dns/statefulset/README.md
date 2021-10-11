## Using Service and StatefulSet
Check file [alpine-statefulset.yml](alpine-statefulset.yml) for configuration.

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
www-web-0   Bound    pvc-23dd9dc6-4feb-41c1-b92a-72fcfb52239e   1Gi        RWO            local-path     91s   Filesystem
www-web-1   Bound    pvc-b5595cc5-cffc-41d6-a1fc-1b6c08eb7c2c   1Gi        RWO            local-path     27s   Filesystem

kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
web-0   1/1     Running   0          30s   10.42.1.30   raspiworker1   <none>           <none>
web-1   1/1     Running   0          26s   10.42.1.31   raspiworker1   <none>           <none>
```

After service is created, it can be referred using `hostname.subdomain.namespace.svc.cluster-domain.example`.

```
kubectl exec web-1 -- ping web-0.default-subdomain.default.svc.cluster.local
PING web-0.default-subdomain.default.svc.cluster.local (10.42.1.30): 56 data bytes
64 bytes from 10.42.1.30: seq=0 ttl=64 time=0.567 ms
64 bytes from 10.42.1.30: seq=1 ttl=64 time=0.240 ms
64 bytes from 10.42.1.30: seq=2 ttl=64 time=0.223 ms
```

## Auto-healing of pods in `kind: StatefulSet`