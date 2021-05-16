---
tags: 
    - k8s
    - kubernetes
    - cronjob
    - schedule
title: Cron Job
date: 2021/05/16
author: 김동환
description:  k8s cron job
disabled: true
categories:
  - kubernetes
---

　
# 크론잡이란?

> 일정한 기간마다 잡을 생성하여 수행하는 것으로 정기적이고 반복적인 작업을 만드는데 사용된다.

　
# 기본 Schema

> 크론(Cron)형식을 사용한다.

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
```