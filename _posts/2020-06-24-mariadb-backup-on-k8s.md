---
tags: 
    - mariadb
    - backup
    - k8s
    - kubernetes
title: mariadb 백업(on k8s)
date: 2020/01/27
author: 김동환
description:  mariadb 백업(on k8s)
disabled: false
categories:
  - dream-factory
---

## mariadb pod 찾기
```shell script
# kubectl get [pod | po] [--all-namespaces | -n {NAMESPACE}] | grep {keyword}
$ kubectl get po -n database | grep maria
```

## pod execute
```shell script
# kubectl exec [po | pod] -n {NAMESPACE} -it {POD_NAME} /bin/bash
$ kubectl exec po -n database mariadb-master-0

# Kubectl exec -n {NAMESPACE} -it {POD_NAME} -- /bin/bash
$ kubectl exec po -n database -it mariadb-master-0 -- /bin/bash
```

## Backup mariaDB in pod
```shell script
# mysqldump -u{USER_NAME} -p -A > {FILE_PATH(NAME)}
$ mysqldump -uroot -p -A > /tmp/dump.sql
```

## Transfer node(local)
```shell script
# kubectl cp {NAMESPACE}/{POD_NAME}:{SOURCE_FILE_PATH} {TARGET_FILE_PATH}
$ kubectl cp database/mariadb-master-0:/tmp/dump.sql ~/mariadb-master-dump.sql
```

## Compression
```shell script
# tar zcvf {FILE_NAME}.tar.gz [ * | {FILE(DIR)_NAME}]
$ tar zcvf mariadb-master-dump.tar.gz ~/mariadb-master-dump.sql
```