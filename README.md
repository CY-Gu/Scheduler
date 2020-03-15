---
title: Kubernetes Scheduler
tags: kubernetes, k8s, Scheduler, Taint & Tolerant, Affinity, Selector
---

# Kubernetes Scheduler

## kube-scheduler

一般來說，kube-scheduler 負責分配 Pod 到叢集内的節點上，它監聽 kube-apiserver，並查詢還未分配 Node 的 Pod，然後根據調度policy為這些 Pod 分配節點

## 創建 Scheduler

創建新的Scheduler時，可以利用etc/kubernetes/manifest下的kube-scheduler.yaml來創建，改個 **--scheduler-name=\<scheduler-name\>** 就可以了，例如:
```=
apiVersion: v1                                                               
kind: Pod 
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    - --scheduler-name=haha-my-scheduler
```

但要注意port不可以重複使用，如何檢查port可以利用 **netstat -natulp | grep \<port-number\>** 來檢查，例如kube-scheduler占用 10251，就查詢 **netstat -natulp | grep 10251**，而**netstat -natulp | grep 10252**則是被kube-controller-manager占用，因此新的Scheduler可使用10253 port。


## 指定 Pod 採用哪個 Scheduler

在 pod-def.yaml 加上 **schedulerName: \<Scheduler-name\>** ，例如
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: nginx-container
      image: nginx
schedulerName: my-scheduler
```
然後可用 **sudo kubectl get events | grep \<scheduler-name\>** 來查詢紀錄

## Pod 調度方式

### Label & Selector

Label & Selector方式是最簡單也最直觀的調度方式，一般來說不會使用(因為太過簡單，無法因應實際環境)，但可以帶入一些k8s中基本 scheduling 的觀念。在 pod-def.yaml 中的 **metadata** 之下加上```labels```的欄位。
```=
metadata:
  name: my-pod
  labels:
    app: test
    function: front-end
  annotations:
    imageregistry: "https://hub.docker.com/"
```
:::info
 **annotations** 代表註釋，可以用來記錄例如版本、時間等其他資訊 
:::

要查詢有此label的Pod，可用以下指令
```=
kubectl get po --selector app=test
```
而在Replica set或Replication Controller等物件中若要選取這個Pod，可在 **spec**中加入```selector```，例如：
```=
spec: 
  replicas: 5
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
        function: front-end
```
---

真正在k8s中實用的有兩種 Pod 調度方式，一種是主動的，一種則是被動的


:::success
 **主動 : 指定 Pod 運行在特定的 Node 上**，有三種方法：
 
*    **nodeSlector**
*    **nodeAffinity**
*    **podAffinity** 
---
**被動 : 指定 Pod "不要"運行在特定的 Node 上**

*    **Taints & tolerations**

:::




---

### Thank you! :sheep: 

You can find me on

- george4908090@gmail.com