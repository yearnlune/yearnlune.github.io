---
tags: 
    - java
    - gradle
    - gradlew
    - gradle wrapper
    - change version
title: gradle wrapper version 변경
date: 2021/07/21
author: 김동환
description: gradle 프로젝트에서 gradlew version 변경하기
disabled: false
categories:
  - java
---

　
# gradlew version 확인

## Command

프로젝트에서 `gradlew --version` 커맨드로 확인할 수 있다.

```powershell
> gradlew --version

------------------------------------------------------------
Gradle 6.6.1
------------------------------------------------------------

Build time:   2020-08-25 16:29:12 UTC
Revision:     f2d1fb54a951d8b11d25748e4711bec8d128d7e3

Kotlin:       1.3.72
Groovy:       2.5.12
Ant:          Apache Ant(TM) version 1.10.8 compiled on May 10 2020
JVM:          1.8.0_211 (Oracle Corporation 25.211-b12)
OS:           Windows 10 10.0 amd64
```

## gradle-wrapper.properties

프로젝트 내에 `./gradle/wrapper/gradle-wrapper.properties` 파일에서 `distributionUrl` 에 정의된 gradle 버전을 통해서 확인할 수 있다.

```ruby
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-6.6.1-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

　
# gradlew version 변경

## Command
프로젝트에서 `gradlew wrapper --gradle-version ${GRADLE_VERSION}` 커맨드로 변경할 수 있다.

```powershell
# gradlew wrapper --gradle-version ${GRADLE_VERSION}
> gradlew wrapper --gradle-version 6.7.1
```

## build.gradle

build.gradle에 gradle 버전을 정의하고 이를 실행(`gradlew wrapper`)하여 적용한다.

```groovy
wrapper {
    gradleVersion = '6.7.1'
}
```

## gradle-wrapper.properties

프로젝트 내에 `./gradle/wrapper/gradle-wrapper.properties` 파일에서 `distributionUrl` 에 정의된 gradle 버전을 변경하여 적용할 수 있다.

```ruby
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-6.7.1-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```