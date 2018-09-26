# Lab1

In this lab, we will demostrate how preemption works in Kubernetes default scheduler.

## Preparation

```console
$ mkdir kubeadm && cd kubeadm
$ grep certificate-authority-data /etc/kubernetes/admin.conf | awk '{print $NF}' | base64 -d > cacert
$ grep client-certificate-data /etc/kubernetes/admin.conf | awk '{print $NF}' | base64 -d > cert
$ grep client-key-data /etc/kubernetes/admin.conf | awk '{print $NF}' | base64 -d > key
$ APISERVER=$(grep server: /etc/kubernetes/admin.conf | awk '{print $NF}')

$ # curl should return api endpoints of apiserver
$ curl --key key --cert cert --cacert cacert ${APISERVER}
```

## Patch fake GPU resources

```console
$ node="xyz" # replace with your own node name
$ curl --key key --cert cert --cacert cacert \
-H "Content-Type: application/json-patch+json" \
--request PATCH \
--data '[{"op": "add", "path": "/status/capacity/ibm.com~1gpu", "value": "5"}]' \
${APISERVER}/api/v1/nodes/${node}/status
```

## Create 4 `PriorityClass`es

```console
$ k create -f priority-classes.yaml
priorityclass.scheduling.k8s.io/p1 created
priorityclass.scheduling.k8s.io/p2 created
priorityclass.scheduling.k8s.io/p3 created
priorityclass.scheduling.k8s.io/p4 created
$ k get pc
NAME                      CREATED AT
p1                        2018-09-25T22:40:04Z
p2                        2018-09-25T22:40:04Z
p3                        2018-09-25T22:40:04Z
p4                        2018-09-25T22:40:04Z
system-cluster-critical   2018-07-10T17:58:38Z
system-node-critical      2018-07-10T17:58:38Z
```

## Create priority1,2,3.yaml

```console
NAME                      READY     STATUS    RESTARTS   AGE
lab1-1-ffd86b9c9-7t92d    1/1       Running   0          1m
lab1-2-5b88c6fcff-58lnx   1/1       Running   0          46s
lab1-3-847787cff5-gp66s   1/1       Running   0          25s
```

## Create priority4.yaml

```console
NAME                      READY     STATUS    RESTARTS   AGE
lab1-1-ffd86b9c9-knxrt    0/1       Pending   0          10m
lab1-2-5b88c6fcff-5x9p4   0/1       Pending   0          10m
lab1-3-847787cff5-h2knh   1/1       Running   0          10m
lab1-4-57fb9db7d7-sxdts   1/1       Running   0          10m
```