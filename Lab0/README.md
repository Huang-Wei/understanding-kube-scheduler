# Lab0

This lab demostrates how kube-scheduler works with kube-controller-manager and kubelet.

## Pause some components

- pause kube-controller-manager
    ```console
    $ docker pause $(docker ps | grep controller-manager | grep -v pause | awk '{print $1}')
    ```console

- pause kube-scheduler
    ```console
    $ docker pause $(docker ps | grep scheduler | grep -v pause | awk '{print $1}')
    ```

## Deploy a deployment

```console
$ k create -f pause-deploy.yaml
deployment.apps/pause created
$ k get po
No resources found.
```

This indicates although `Deployment` API object is persistened to etcd store. But controller-manager (paused by us earlier) has no chance to kickoff `Pod`.

## Unpause kube-controller-manager

```console
$ docker unpause $(docker ps | grep controller-manager | grep Paused | awk '{print $1}')
$ k get po -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP        NODE      NOMINATED NODE
pause-77cdbd74df-j48pk   0/1       Pending   0          7s        <none>    <none>    <none>
```

Pod `pause-77cdbd74df-j48pk` is created with `Pending` status, which indicates controller-manager works well.

## Unpause kube-scheduler

```console
$ docker unpause $(docker ps | grep scheduler | grep Paused | awk '{print $1}')
$ k get po -o wide -w
NAME                     READY     STATUS    RESTARTS   AGE       IP        NODE      NOMINATED NODE
pause-77cdbd74df-j48pk   0/1       Pending   0          1m        <none>    <none>    <none>
pause-77cdbd74df-j48pk   0/1       Pending   0         1m        <none>    wei-k8s-worker2   <none>
pause-77cdbd74df-j48pk   0/1       ContainerCreating   0         1m        <none>    wei-k8s-worker2   <none>
pause-77cdbd74df-j48pk   0/1       ContainerCreating   0         2m        <none>    wei-k8s-worker2   <none>
pause-77cdbd74df-j48pk   1/1       Running   0         2m        192.168.2.64   wei-k8s-worker2   <none>
```

This indicates scheduler starts to work - schedule `Pending` pod to a node.