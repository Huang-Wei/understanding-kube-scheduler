# Lab3

This lab runs a series of pods with NodeAffinity and PodAntiAffinity to demostrate how predicate works.

## Preparation

```console
$ k label no <node1> region=r1 zone=z1
$ k label no <node2> region=r1 zone=z2
$ k get no --show-labels
```

## Create node-affinity-deploy.yaml

## Create pod-antiaffinity-deploy.yaml