---
title: Kubernetes Scheduler
tags: kubernetes, k8s, Scheduler
---

# Kubernetes Scheduler

## Scheduler

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

---

### Thank you! :sheep: 

You can find me on

- george4908090@gmail.com