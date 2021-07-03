---
tags:
    - git
    - config
    - local
    - email
    - username
title: 프로젝트 별 계정 설정
date: 2021/07/03
author: 김동환
description: 프로젝트 별 계정 설정 (name, email)
disabled: false
categories:
- git
---


# Git Bash

```bash
git config --local user.name "DREAM_FACTORY"
git config --local user.email "DREAM_FACTORY@example.com"
```

# File

**프로젝트** 내에 `.git/config`를 수정하여 적용한다.

```
[user]
	email = DREAM_FACTORY@example.com
	name = DREAM_FACTORY
```