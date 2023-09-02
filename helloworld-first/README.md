# How to deploy nginx pods

## Deploy using kubectl

```
k3s_tutorial/helloworld-first$ sudo kubectl apply -f .
deployment.apps/nginx created
service/nginx-nodeport created

sudo kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
nginx-6cfb8dcb7b-lbbcg   1/1     Running   0          35s   10.42.0.17   shubham-mini14   <none>           <none>
nginx-6cfb8dcb7b-k2sp2   1/1     Running   0          35s   10.42.0.16   shubham-mini14   <none>           <none>
nginx-6cfb8dcb7b-dlxzw   1/1     Running   0          35s   10.42.0.15   shubham-mini14   <none>           <none>
nginx-6cfb8dcb7b-89xns   1/1     Running   0          35s   10.42.0.18   shubham-mini14   <none>           <none>
nginx-6cfb8dcb7b-fnpgz   1/1     Running   0          34s   10.42.0.20   shubham-mini14   <none>           <none>
nginx-6cfb8dcb7b-f9zp5   1/1     Running   0          35s   10.42.0.19   shubham-mini14   <none>           <none>

```

## Check node IP and port from below

```
sudo kubectl get svc -o wide
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE    SELECTOR
kubernetes       ClusterIP   10.43.0.1      <none>        443/TCP        65m    <none>
nginx-nodeport   NodePort    10.43.172.35   <none>        80:31111/TCP   3m5s   app=nginx
```

Access <NODE-IP:NODE-PORT>

## Delete deployment

```
k3s_tutorial/helloworld-first$ sudo kubectl delete -f .
deployment.apps "nginx" deleted
service "nginx-nodeport" deleted
```
