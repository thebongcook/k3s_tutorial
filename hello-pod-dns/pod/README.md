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

## Auto-healing of pods in `kind: Pod`
When a pod (alpine02) is deleted, it is **not** auto-healed and replaced by another one

### `kubectl delete pods` method
```
kubectl apply -f hello-pod-dns/alpine-pods.yml 
service/default-subdomain created
pod/alpine01 created
pod/alpine02 created

kubectl delete pods alpine02
pod "alpine02" deleted


Every 2.0s: kubectl get pods -o wide                     raspberrypi: Sun Sep 26 01:38:24 2021

NAME       READY   STATUS        RESTARTS   AGE   IP           NODE           NOMINATED NODE
 READINESS GATES
alpine01   1/1     Running       0          59s   10.42.1.84   raspiworker1   <none>
 <none>
alpine02   0/1     Terminating   0          59s   10.42.1.85   raspiworker1   <none>
 <none>
```

### Shutdown node on which pod is scheduled
```
Every 2.0s: kubectl get nodes -o wide                                              raspberrypi: Sun Sep 26 01:50:14 2021

NAME           STATUS     ROLES                  AGE    VERSION        INTERNAL-IP     EXTERNAL-IP   OS-IMAGE
              KERNEL-VERSION   CONTAINER-RUNTIME
raspiworker3   Ready      <none>                 5d6h   v1.21.4+k3s1   192.168.0.203   <none>        Raspbian GNU/Linux
10 (buster)   5.10.60-v8+      containerd://1.4.9-k3s1
raspiworker1   NotReady   <none>                 5d6h   v1.21.4+k3s1   192.168.0.201   <none>        Raspbian GNU/Linux
10 (buster)   5.10.60-v8+      containerd://1.4.9-k3s1
raspberrypi    Ready      control-plane,master   9d     v1.21.4+k3s1   192.168.0.200   <none>        Raspbian GNU/Linux
10 (buster)   5.10.60-v7l+     containerd://1.4.9-k3s1
```

Pods show `Running` but (obviously) don't respond
```
Every 2.0s: kubectl get pods -o wide                                         raspberrypi: Sun Sep 26 01:52:01 2021

NAME       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
alpine02   1/1     Running   1          9m11s   10.42.1.91   raspiworker1   <none>           <none>
alpine01   1/1     Running   1          14m     10.42.1.90   raspiworker1   <none>           <none
```

```
kubectl exec alpine01 -- ping alpine02.default-subdomain.default.svc.cluster.local
Error from server: error dialing backend: dial tcp 192.168.0.201:10250: connect: no route to host
```
```
kubectl delete -f hello-pod-dns/alpine-deployments.yml 
Error from server (NotFound): error when deleting "hello-pod-dns/alpine-deployments.yml": services "alpine01-subdomain" not found
Error from server (NotFound): error when deleting "hello-pod-dns/alpine-deployments.yml": deployments.apps "alpine01" not found
Error from server (NotFound): error when deleting "hello-pod-dns/alpine-deployments.yml": services "alpine02-subdomain" not found
Error from server (NotFound): error when deleting "hello-pod-dns/alpine-deployments.yml": deployments.apps "alpine02" not found
```
