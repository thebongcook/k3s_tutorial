# How to ping a pod using DNS

DNS for service and pods can be used as described [here](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/).

There are three ways to create two distinct pods which can ping each other

- [Using Serice and Pod (doesn't auto-heal)](pod/README.md)
- [Using Service and Deployments (possible but not realistic)](deployment/README.md)
- [Using Service and StatefulSet](statefulset/README.md)



