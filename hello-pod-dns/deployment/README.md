## Using Service and Deployments (possible but not realistic)

Check file [alpine-deployments.yml](alpine-deployments.yml) for configuration.

## Creation of Deployment
If you use 1 pod per deployment, the domain name will refer to the intended pod.

```
kubectl apply -f hello-pod-dns/alpine-deployments.yml 
service/alpine01-subdomain created
deployment.apps/alpine01 created
service/alpine02-subdomain created
deployment.apps/alpine02 created

kubectl get pods -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
alpine01-7659ddbc-hsdb4   1/1     Running   0          37m   10.42.1.64   raspiworker1   <none>           <none>
alpine02-cc49f7c4-2crlz   1/1     Running   0          33m   10.42.1.65   raspiworker1   <none> 
```

```
kubectl exec alpine01-7659ddbc-hsdb4 -- ping alpine02.alpine02-subdomain.default.svc.cluster.local
PING alpine02.alpine02-subdomain.default.svc.cluster.local (10.42.1.65): 56 data bytes
64 bytes from 10.42.1.65: seq=0 ttl=64 time=0.375 ms
64 bytes from 10.42.1.65: seq=1 ttl=64 time=0.194 ms
64 bytes from 10.42.1.65: seq=2 ttl=64 time=0.227 ms

```

If you use multiple pod per deployment, the domain name will refer to the last pod created.

```
kubectl apply -f hello-pod-dns/alpine-deployments.yml 
service/alpine01-subdomain created
deployment.apps/alpine01 created
service/alpine02-subdomain created
deployment.apps/alpine02 created

kubectl get pods -o wide
NAME                      READY   STATUS    RESTARTS   AGE    IP            NODE           NOMINATED NODE   READINESS GATES
alpine02-cc49f7c4-z4qk7   1/1     Running   0          115s   10.42.0.212   raspberrypi    <none>           <none>
alpine01-7659ddbc-q2tb8   1/1     Running   0          115s   10.42.1.73    raspiworker1   <none>           <none>
alpine02-cc49f7c4-nzdnl   1/1     Running   0          115s   10.42.1.72    raspiworker1   <none>           <none>
```

```
kubectl exec alpine01-7659ddbc-hsdb4 -- ping alpine02.alpine02-subdomain.default.svc.cluster.local
PING alpine02.alpine02-subdomain.default.svc.cluster.local (10.42.1.66): 56 data bytes
64 bytes from 10.42.1.66: seq=0 ttl=64 time=0.361 ms
64 bytes from 10.42.1.66: seq=1 ttl=64 time=0.220 ms
64 bytes from 10.42.1.66: seq=2 ttl=64 time=0.212 ms
```

## Auto-healing of pods in `kind: Deployment`

When a pod (alpine02) is deleted, it is auto-healed and replaced by another one

```
kubectl delete pods alpine02-cc49f7c4-nzdnl
pod "alpine02-cc49f7c4-nzdnl" deleted

pi@raspberrypi:~ $ kubectl get pods -o wide
NAME                      READY   STATUS        RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
alpine01-7659ddbc-q2tb8   1/1     Running       4          4h37m   10.42.1.73   raspiworker1   <none>           <none>
alpine02-cc49f7c4-82pt5   1/1     Running       4          4h35m   10.42.1.74   raspiworker1   <none>           <none>
alpine02-cc49f7c4-nzdnl   1/1     Terminating   4          4h37m   10.42.1.72   raspiworker1   <none>           <none>
alpine02-cc49f7c4-6955h   1/1     Running       0          10s     10.42.1.75   raspiworker1   <none>           <none>
```

### Shutdown node on which pod is scheduled
