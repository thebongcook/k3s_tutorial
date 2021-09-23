# How to ping a pod using DNS

DNS for service and pods can be used as described [here](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/).

## Using Serice and Pod

Check file [alpine-pods.yml](alpine-pods.yml) for configuration.

A general pod can be addressed as `pod-ip-address.namespace.pod.cluster-domain.example`.

```
kubectl exec alpine01 -- ping 10-42-0-200.default.pod.cluster.local
PING 10-42-0-200.default.pod.cluster.local (10.42.0.200): 56 data bytes
64 bytes from 10.42.0.200: seq=0 ttl=64 time=0.271 ms
64 bytes from 10.42.0.200: seq=1 ttl=64 time=0.299 ms
64 bytes from 10.42.0.200: seq=2 ttl=64 time=0.291 ms
```

After service is created, it can be referred using `hostname.subdomain.namespace.svc.cluster-domain.example`.

```
kubectl exec alpine01 -- ping alpine02.default-subdomain.default.svc.cluster.local
PING alpine02.default-subdomain.default.svc.cluster.local (10.42.0.203): 56 data bytes
64 bytes from 10.42.0.203: seq=0 ttl=64 time=0.532 ms
64 bytes from 10.42.0.203: seq=1 ttl=64 time=0.232 ms
64 bytes from 10.42.0.203: seq=2 ttl=64 time=0.278 ms
64 bytes from 10.42.0.203: seq=3 ttl=64 time=0.279 ms
```
## Using Service and Deployments (possible but not realistic)

Check file [alpine-deployments.yml](alpine-deployments.yml) for configuration.

If you use 1 pod per deployment, it can access the intended pod.

```
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

If you use multiple pod per deployment, it will access the last pod created.

``` 
kubectl get pods -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
alpine01-7659ddbc-hsdb4   1/1     Running   0          37m   10.42.1.64   raspiworker1   <none>           <none>
alpine02-cc49f7c4-2crlz   1/1     Running   0          33m   10.42.1.65   raspiworker1   <none>           <none>
alpine02-cc49f7c4-996n5   1/1     Running   0          33m   10.42.1.66   raspiworker1   <none>           <none>
```

```
kubectl exec alpine01-7659ddbc-hsdb4 -- ping alpine02.alpine02-subdomain.default.svc.cluster.local
PING alpine02.alpine02-subdomain.default.svc.cluster.local (10.42.1.66): 56 data bytes
64 bytes from 10.42.1.66: seq=0 ttl=64 time=0.361 ms
64 bytes from 10.42.1.66: seq=1 ttl=64 time=0.220 ms
64 bytes from 10.42.1.66: seq=2 ttl=64 time=0.212 ms
```