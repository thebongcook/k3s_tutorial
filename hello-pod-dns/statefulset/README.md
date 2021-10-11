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

## Creation of PVC and StatefulSet
The PVC are created automatically thanks to k3s's dynamic persistent volume provisioning.
However, it's not auto deleted with the pods, so it needs to be deleted manually as shown [here](#manually-delete-pvc).

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
When a pod (web-0) is deleted, it is auto-healed and replaced by another one. Note the (internal) IP of the new pod is different from the deleted one.

```
kubectl delete pods web-0 &
[1] 14951

kubectl get pods -o wide
NAME    READY   STATUS        RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
web-1   1/1     Running       0          5m26s   10.42.1.31   raspiworker1   <none>           <none>
web-0   1/1     Terminating   0          86s     10.42.1.32   raspiworker1   <none>           <none>

kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
web-1   1/1     Running   0          5m42s   10.42.1.31   raspiworker1   <none>           <none>
web-0   1/1     Running   0          5s      10.42.1.33   raspiworker1   <none>           <none>
```

DNS works same as before, now refering to the new IP.

```
kubectl exec web-1 -- ping web-0.default-subdomain.default.svc.cluster.local
PING web-0.default-subdomain.default.svc.cluster.local (10.42.1.33): 56 data bytes
64 bytes from 10.42.1.33: seq=0 ttl=64 time=0.318 ms
64 bytes from 10.42.1.33: seq=1 ttl=64 time=0.202 ms
64 bytes from 10.42.1.33: seq=2 ttl=64 time=0.236 ms
```

## Manually delete PVC

```
kubectl delete --all pvc
persistentvolumeclaim "misc-web-0" deleted
persistentvolumeclaim "misc-web-1" deleted
```