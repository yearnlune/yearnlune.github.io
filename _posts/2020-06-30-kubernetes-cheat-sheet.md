---
tags: 
    - k8s
    - kubernetes
title: 쿠버네티스 치트시트
date: 2020/06/30
author: 김동환
description:  쿠버네티스 치트시트
disabled: false
categories:
  - kubernetes
---
# Kubernetes

## kubernetes?

## DELETE

### Delete pod

```
$ kubectl delete pod {POD_NAME} (-n {NAMESPACE} | --all-namespaces)

$ kubectl delete pod kubernetes-dashboard-57df4db6b-44dkm -n kube-system
```

### Force delete pod

```
$ kubectl delete pods {POD_NAME} (-n {NAMESPACE} | --all-namespaces) --grace-period=0 --force

$ kubectl delete pod kubernetes-dashboard-57df4db6b-44dkm -n kube-system --grace-period=0 --force
```

## Copy

> 파드와 노드 간의 복사하거나 붙여넣기

```
# kubectl cp <file-spec-src> <file-spec-dest> [options]

Options:
  -c, --container='': Container name. If omitted, the first container in the pod will be chosen
      --no-preserve=false: The copied file/directory's ownership and permissions will not be preserved in the container
```

### Copy node to pod

```
# kubectl cp {NODE_FILE} {NAMESPACE}/{POD_NAME}:{POD_FILE} [-c {CONATAINER_NAME}]
$ kubectl cp /tmp/foo kube-database/mariadb-master-0:/tmp/foo

$ kubectl cp /tmp/foo kube-database/mongodb-0:/tmp/foo -c mongo-db-container
```

### Copy pod to node

```
# kubectl cp {NAMESPACE}/{POD_NAME}:{POD_FILE} {NODE_FILE}

$ kubectl cp kube-database/mariadb-master-0:/tmp/foo /tmp/foo
```

---

## Taint

> 해당 노드에서 파드들을 실행할 수 없도록 제외하는 것이다.
Node affinity와는 반대로 해당 노드가 파드를 제외할 수 있다.

### Option

`NoSchedule` : toleration이 없을 경우 해당 노드에 pod가 생성 되지 않음, 기존 pod는 유지

`PreferNoSchedule` : toleration이 없을 경우 해당 노드에 pod가 생성 되지 않도록 시도함(Soft), 자원이 부족한 경우 해당 노드에 할당 될 수 있음

`NoExecute` :  toleration이 없을 경우 해당 노드에 pod가 생성 되지 않으며 기존의 pod도 영향 받음

### Add taint

```bash
$ kubectl label nodes <storage-node-name> node-role.kubernetes.io/storage=''

# kubectl taint nodes {NODE_NAME} {KEY}={VALUE}:(NoSchedule|PreferNoSchedule|NoExecute)
$ kubectl taint nodes node-10 node-role.kubernetes.io/storage='':PreferNoSchedule
```

```yaml
# CephCluster 에서 아래 내용을 꼭 추가해줘야 한다!
placement:
	osd:
		tolerations:
		- key: node-type.cluster.label/storage
			operator: Exists
```

### Delete taint

```bash
# kubectl taint nodes {NODE_NAME}:(NoSchedule|PreferNoSchedule|NoExecute)-
$ kubectl taint nodes node-10 node-role.kubernetes.io/storage='':PreferNoSchedule-
```

## Toleration

> Taint를 무시(예외)하여 가용하도록 한다

### Add toleration

```yaml
tolerations:
- key: "node-role.kubernetes.io/storage"
  operator: "Exists"
  effect: "PreferNoSchedule"
```

## UID

### Print UID

```bash
# pod의 UID 표기
$ kubectl get pods --all-namespaces -o custom-columns=PodName:.metadata.name,PodUID:.metadata.uid
```

## EVENTS

### Print events

```bash
# kubectl get (events|ev) -n [NAMESPACE] --sort-by=.metadata.creationTimestamp
$ kubectl get ev-n kube-system --sort-by=.metadata.creationTimestamp
```