# k3s-tutorial

k3s tutorial is a self-learning toolkit to learn k8s on a raspberry pi cluster

## Force deployment on worker node

Deployment is distributed automatically by k8s accross nodes based on available resource (eg. memory). However if you want to force deployment on a particular node, in this case worker node, you can do so using [nodeSelector](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/).

First, create the label for the node.

```
kubectl label nodes raspiworker1 nodetype=worker
```

Then, add the `nodeSelector` in the `spec` of the deployment to select the node(s).

```
spec:
  template:
    spec:
      nodeSelector:
        nodetype: worker
```

## Spread pods accross zones

This is similar to zones (and also regions) in GCP.
The original motivation was to distribute workload between both powerful Raspberry Pi 4 and weak Raspberry Pi 4 nodes. By default all pods would be deployed on the Pi 4.

We can use [topologySpreadConstraints](https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/) to spread pods accross zones (and effectively nodes).

First, create the label for the node.

```
kubectl label nodes raspberrypi zone=zoneA
kubectl label nodes raspiworker1 zone=zoneB
```

Then, add the `topologySpreadConstraints` in the `spec` of the deployment to select the node(s).

```
spec:
  template:
    spec:
      topologySpreadConstraints:
      - maxSkew: 3
        topologyKey: zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: hello-world
```