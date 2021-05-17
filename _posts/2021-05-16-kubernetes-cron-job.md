---
tags: 
    - k8s
    - kubernetes
    - cronjob
    - schedule
    - 크론잡
title: CronJob
date: 2021/05/16
author: 김동환
description:  k8s CronJob(크론잡)
disabled: false
categories:
  - kubernetes
---

　
# 크론잡이란?

> 일정한 기간마다 잡을 생성하여 수행하는 것으로 정기적이고 반복적인 작업을 만드는데 사용된다. schedule은 linux의 cron의 스키마를 활용하며, 해당 작업의 실패 시의 처리(restartPolicy) 등을 설정할 수 있다.

```bash
$ kubectl get cronjob -A

NAMESPACE          NAME                                           SCHEDULE       SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cert-manager       cert-manager-webhook-ca-sync                   @weekly        False     0        35h             2y110d
zenko              zenko-zenko-reporting-count-items              @hourly        False     0        52m             408d
tester             mysql-backup-cronjob                           0 0 * * *      False     0        11h             69d

$ kubectl get job -n tester | grep mysql-backup-cronjob

NAME                                         COMPLETIONS   DURATION   AGE
mysql-backup-cronjob-1621036800              1/1           92m        2d12h
mysql-backup-cronjob-1621123200              1/1           109m       36h
mysql-backup-cronjob-1621209600              1/1           84m        12h

$ kubectl get pod -n tester | grep mysql-backup-cronjob

NAME                                                          READY   STATUS             RESTARTS   AGE
mysql-backup-cronjob-1621036800-6jkcq                         0/1     Completed          0          2d12h
mysql-backup-cronjob-1621123200-vvc8g                         0/1     Completed          0          36h
mysql-backup-cronjob-1621209600-b4qph                         0/1     Completed          0          12h
```

- CronJob은 Job을 생성 및 관리하며, 생성된 Job은 Pod를 생성 및 관리한다.

　
# 기본 Schema

## cron

```bash
# ┌───────────── 분 (0 - 59)
# │ ┌───────────── 시 (0 - 23)
# │ │ ┌───────────── 일 (1 - 31)
# │ │ │ ┌───────────── 월 (1 - 12)
# │ │ │ │ ┌───────────── 요일 (0 - 6) (일요일부터 토요일까지;
# │ │ │ │ │                            특정 시스템에서는 7도 일요일)
# │ │ │ │ │
# │ │ │ │ │
# * * * * *

# * -- 어떠한 수든 상관없이
# , -- ~와
# - -- ~사이에
# / -- ~마다

# 0 0 1 1 * -- 매년 1월 1일 자정에 실행 @yearly
# 0 0 1 * * -- 매월 1일 자정에 실행 @monthly
# 0 0 * * 0 -- 매주 일요일 자정에 실행 @weekly
# 0 0 * * * -- 매일 자정에 실행 @daily
# 0 * * * * -- 매시 0분에 시작 @hourly

# */30 * * * * -- 매번 30분 마다
# 0 2 */1 * * -- 매번 1일 02시 마다
```

　
## cronjob.yaml

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
    concurrencyPolicy: Allow
    suspend: false
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

`spec.schedule`
: 스케쥴 정책을 cron의 형태로 설정한다.

`spec.concurrencyPolicy` 
: 해당 작업(Job)을 진행 할 시 동시 실행의 여부를 설정한다. 기본 값은 `Allow`이며, `Forbid` 를 통해서 동시 실행을 거부할 수 있다. `Replace`의 경우 기존의 진행하고 있는 작업을 새로 교체한다.

`spec.suspend`
: 기본값은 false이며, true일 경우 향후에 실행될 작업(Job)들을 연기(중지)시킨다.

`spec.startingDeadlineSeconds`
: 어떠한 이유로 스케쥴대로 실행하지 못했을 경우 deadline을 설정할 수 있다. 기본 값은 nil여서 deadline이 없다.

　
# 따라해보기

> node를 활용하여 간단한 job 생성하기

## STEP 1. 작업 생성하기

해당 작업을 실패할 경우 `process.exit(1);` 을 통해서 실패를 알린다.

```tsx
//index.ts
import {schedule} from "./schedule";

schedule.start()
    .then(() => {
        process.exit(0);
    })
    .catch((e) => {
        console.error(`FAILED ${e}`);
        process.exit(1);
    });

///////////////////////////////////////////

//schedule.ts
export class Schedule {
    public start(): Promise<any> {
        return mongodbHandler.tryConnect()
            .then(this.createLatestReports)
            .then(results => this.createEventReports(results))
            .then(results => this.saveReports(results));
    }
	
	private createLatestReports(): Promise<any> {...}
	
	...
}
```

　
## STEP 2. cronjob 생성하기

`spec.schedule` 에 스케쥴러를 설정하며,  `spec.concurrencyPolicy`, `spec.suspend` 등을 설정하여 적용한다.

```yaml
kind: CronJob
apiVersion: batch/v1beta1
metadata:
  name: report-cronjob
spec:
  schedule: '*/30 * * * *'
  concurrencyPolicy: Allow
  suspend: false
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: report-cronjob
              image: >-
                local.docker.repository.com/report-cronjob:release-0.0.1
              imagePullPolicy: IfNotPresent
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
```

　
# 참고자료

[k8s cron-jobs](https://kubernetes.io/ko/docs/concepts/workloads/controllers/cron-jobs/)

[cron wiki](https://en.wikipedia.org/wiki/Cron)