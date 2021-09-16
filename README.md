# k3s-tutorial

k3s tutorial is a self-learning toolkit to learn k8s on a raspberry pi cluster

## Force deployment on worker node

Deployment is distributed automatically by k8s accross nodes based on available resource (eg. memory). However if you want to force deployment on a particular node, in this case worker node, you can do so using [nodeSelector](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/).

First, create the label for the node.

```
kubectl label nodes raspiworker1 nodetype=worker
```

Then, add the `nodeSelector` in the `spec` of the deployment to select the node(s).