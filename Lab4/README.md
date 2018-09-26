# Lab4

This lab demostrate how Priority works.

## Preparation

It's usually unpredictable to observer how a specific Priority work. An easy way is to use PriorifyConfigPolicy file which will be cover in further labs.

Another hacky way is to modify source code to manually change the weight. In this lab, we're going this way.

First thing is to apply [most-request.diff](most-request.diff) which enabled `MostRequestdPriority` and set its weight to `200`.

## Compile the source and spin up a 3-node cluster

> It's necessary to try Priority on a multi-node cluster. In a single-node cluster, Priorites function are simply skipped.

This labs uses [kubeadm-dind-cluster](https://github.com/kubernetes-sigs/kubeadm-dind-cluster) to launch the cluster.

```console
NAME          STATUS    ROLES     AGE       VERSION
kube-master   Ready     master    67m       v1.13.0-alpha.0.1383+488f0fcda3f9ba-dirty
kube-node-1   Ready     <none>    66m       v1.13.0-alpha.0.1383+488f0fcda3f9ba-dirty
kube-node-2   Ready     <none>    66m       v1.13.0-alpha.0.1383+488f0fcda3f9ba-dirty
kube-node-3   Ready     <none>    66m       v1.13.0-alpha.0.1383+488f0fcda3f9ba-dirty
```

## Create most-request1.yaml

As of now, there are no workloads running on the cluster. We create the first deployment. It will land on a node randomly:

```console
NAME                      READY     STATUS    RESTARTS   AGE       IP           NODE          NOMINATED NODE
lab4-1-bdcbd9588-w2gpb    1/1       Running   0          10s       10.244.2.7   kube-node-1   <none>
```

And from scheduler log, we can see 3 nodes are scored equaly:

```console
I0926 19:56:10.719831       1 generic_scheduler.go:729] Host kube-node-1 => Score 100227
I0926 19:56:10.719836       1 generic_scheduler.go:729] Host kube-node-2 => Score 100227
I0926 19:56:10.719840       1 generic_scheduler.go:729] Host kube-node-3 => Score 100227
```

## Create most-request2.yaml

The 2nd deployment lands on "kube-node-1" as well.

```console
NAME                      READY     STATUS    RESTARTS   AGE       IP           NODE          NOMINATED NODE
lab4-1-bdcbd9588-w2gpb    1/1       Running   0          32s       10.244.2.7   kube-node-1   <none>
lab4-2-6d9bcf54f5-s8wtv   1/1       Running   0          5s        10.244.2.8   kube-node-1   <none>
```

Form scheduelr log, it can be confirmed that it's `MostRequestdPriority` starts to take effect.

```console
I0926 19:56:37.126659       1 generic_scheduler.go:729] Host kube-node-1 => Score 100425
I0926 19:56:37.126670       1 generic_scheduler.go:729] Host kube-node-2 => Score 100227
I0926 19:56:37.126675       1 generic_scheduler.go:729] Host kube-node-3 => Score 100227
```

## Create most-request3.yaml

To double check, we create the 3rd deployment. As expected, it lands on "kube-node-1" again.

```console
NAME                      READY     STATUS    RESTARTS   AGE       IP           NODE          NOMINATED NODE
lab4-1-bdcbd9588-w2gpb    1/1       Running   0          63s       10.244.2.7   kube-node-1   <none>
lab4-2-6d9bcf54f5-s8wtv   1/1       Running   0          36s       10.244.2.8   kube-node-1   <none>
lab4-3-64d64f654b-gcfrr   1/1       Running   0          2s        10.244.2.9   kube-node-1   <none>
```

And log proves that also:

```console
I0926 19:57:11.049870       1 generic_scheduler.go:729] Host kube-node-3 => Score 100227
I0926 19:57:11.049879       1 generic_scheduler.go:729] Host kube-node-1 => Score 100622
I0926 19:57:11.049901       1 generic_scheduler.go:729] Host kube-node-2 => Score 100227
```