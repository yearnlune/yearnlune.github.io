---
tags: 
    - java
    - Garbage Collection
    - gc
title: Garbage Collection
date: 2022/08/18
author: 김동환
description: JVM 구조 및 GC의 종류
disabled: false
categories:
  - java
---

# JVM (Java Virtual Machine)

> Write once, run anywhere
>

JVM은 자바 바이트코드를 실행하고, 런타임 환경을 제공해준다. 플랫폼에 상관없이 명령어를 실행하여, 플랫폼에 독립적인 특징을 가지고 있다. 주된 JVM은 Sun사의 Hotspot JVM, Oracle사의 JRockit JVM, IBM의 J9 JVM 등이 있다.

## Component

JVM은 크게 `Class Loader`, `Execution Engine`, `Runtime Data Areas`로 구성되어 있다.

![key-hotspot-jvm-components](/assets/images/java/garbage-collection/key-hotspot-jvm-components.png)

### Class Loader

자바 바이트 코드를 **로딩하고 런타임 데이터를 생성**한다. 생성된 런타임 데이터는 JVM의 런타임 데이터 영역 중 Method 영역에 적재한다.

### Execution Engine

**바이트 코드를 실행**한다. 실행하는 방식에는 크게 두가지가 있다. 하나씩 바이트 코드를 해석하며 실행하는 **Interpreter 방식**과 Interpreter의 느린 단점을 보완하기 위해 바이트 코드 전체를 바이너리 코드로 변경하고 이를 실행하는 **JIT(Just-In-Time) Compile 방식**이 있다.

### Runtime Data Areas

해당 영역에는 크게 Method, Stack, Heap, PC Register, Native method stack이 존재한다.

1. Method area

   클래스, 필드, 메서드 등을 저장하는 공간이다.
2. Stack area

   Primitive type의 값이나, Heap 영역에 생성된 객체의 데이터 참조 주소를 저장하는 공간이다.
3. Heap area

   객체, 배열 등 동적으로 할당된 데이터가 저장되는 공간이다.
4. PC register area

   스택의 Operand를 가져와 저장하는 공간으로, 쓰레드가 어떠한 명령을 수행해야 하는지를 저장한다.
5. Native method stack area

   JNI(Java Native Interface)를 통해 수행하는 명령을 저장한다.

# **GC (Garbage Collection)**

GC는 위에서 살펴본 `Heap area`에서 사용 중인 객체(라이브 객체)와 아닌 객체를 구별하여 사용되지 않는 객체를 삭제하는 행동을 의미한다.

## Hotspot JVM heap

![hotspot-heap-structure](/assets/images/java/garbage-collection/hotspot-heap-structure.png)

### young generation

> young generation은 Eden, Survivor0, Survivor1의 공간이 지니며, 새로운 객체가 생성되면 처음으로 배치되는 공간이다.
>

새로운 객체가 생성된다면 `Eden` 공간에 배치가 된다. 해당 공간이 가득 차게 되면 GC(Minor GC)가 발생하며, 해당 GC에서 살아남은 객체는 하나의 Survivor 공간에 배치된다. 이후에 Survivor 공간이 가득차게 되면 GC가 발생하며, 여기서 살아남은 객체는 다른 Survivor 공간에 배치된다. 반복된 이러한 과정에서 살아남은 객체들은 old generation으로 이동하게 된다.

### old generation

> old generation은 young generation에서 넘어온 객체를 배치하는 공간이다.
>

young generation에서 넘어온 객체를 배치하게 되는데, 이 과정에서 객체들이 일정수준 쌓이게 되면 GC(Major GC)가 발생하게 된다. 해당 GC가 발생할 때 STW(Stop-the-world)가 발생하는데 이 과정에서 Application의 스레드가 멈추게 된다.

### permanent generation

> permanent generation는 Class에 대한 메타 정보나 JVM이 생성한 내부 객체들이 배치되는 공간이다.
>

자바 8 이후로는 heap area의 permanent generation이 삭제되고 Native memory 영역의 **metaspace**로 변경되었다.

## GC type

### Serial GC

싱글 스레드로 동작하는 GC이다. 그리하여 STW가 길다. CPU의 코어가 한 개인 경우만 사용해야 한다. 접근 가능한 객체를 가려내고(Mark) 나머지를 삭제한뒤(Sweep), 가려낸 객체를 모은다.(Compact)

### Parallel GC

Minor GC를 멀티 스레드로 병렬 동작하는 GC이다. **JAVA 8**에서 기본 GC로 사용되고 있다. Major GC 또한 멀티 스레드로 동작하기 위해서는 `Parallel Old GC` 를 사용해야 한다.

### CMS GC

접근 가능한 객체를 가려내는 행위를 한번에 하는 것이 아닌 나눠서하는 GC이다. GC Root가 가장 가까운 객체만을 최초로 마킹하며(STW), 이후 객체의 참조를 따라가며 마킹을 한다. 이후에 다시 한번 마킹한 객체를 다시 확인하며(STW), 참조되지 않은 객체는 삭제한다.

STW의 시간을 줄이는데는 효과적이지만, 위에서 살펴본 GC에 비해 많은 CPU, Memory를 사용하게 된다. 또한 Compaction 작업이 없어 Memory Fragmentation이 발생하게 되고 임계점에 다다르면 Compaction을 진행하게되고, 이 시간이 STW보다 더 오래 걸릴 수 있다.

### G1 GC

위에서 살펴본 heap 영역과는 달리 generation을 두지 않고 1~32MB 크기의 2000여개 region으로 관리한다. 해당 region은 eden, survivor, old의 영역이 될 수 있다. 추가적으로 region 크기의 50%가 넘는 큰 객체를 저장할 수 있는 Humongous 영역이 추가되었다. 기본적인 GC는 remember set에 저장된 region의 참조 정보를 토대로 진행한다. 전체 영역에 GC를 실행하는 것이 아닌 region 단위로 실행 할 수 있기에 이전 GC들보다 STW 시간이 짧다. **JAVA 9**부터 기본 GC로 사용되고 있다.

![g1-heap-allocation](/assets/images/java/garbage-collection/g1-heap-allocation.png)

G1GC는 eden space와 survivor space의 사이즈의 계산을 통해 **Young GC**를 수행한다. 애플리케이션의 스레드를 멈추고(STW), 멀티스레드를 통해 라이브 객체를 eden에서 survivor 혹은 survivor에서 old generation으로 복사 및 이동하며, 그 외 미사용 객체의 region은 available region으로 만든다.

old region 점유율 임계값(IHOP)을 도달하게 되면 **Old GC**를 수행한다. old gc의 경우 Initial Mark, Concurrent Marking, Remark, Cleanup, Copy 크게 5가지의 단계를 지닌다.

1. Initial Mark(STW)

   old region에 라이브 객체들이 참조하는 survivor region을 마킹한다.
2. Concurrent Marking

   전체 heap에서 라이브 객체 및 빈 객체를 찾아 마킹하며, region의 라이브 객체의 비율을 계산한다.
3. Remark(STW)

   SATB(Snapshot At The Beginning) 알고리즘을 통해 라이브 객체를 최종 판단하여 마킹한다. Concurrent Marking에서 빈 객체로 마킹한 region을 제거한다.
4. Cleanup(STW)

   라이브 객체의 비율을 참고하여 접근 가능 객체가 가정 적은 Region에 대한 미사용 객체를 처리한다.
5. Copy(STW)

   라이브 객체들을 available region에 copy 및 compaction을 수행한다.

# 참고자료

[https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)

[https://www.oracle.com/technical-resources/articles/java/g1gc.html](https://www.oracle.com/technical-resources/articles/java/g1gc.html)

[https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-ED3AB6D3-FD9B-4447-9EDF-983ED2F7A573](https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-ED3AB6D3-FD9B-4447-9EDF-983ED2F7A573)

[https://d2.naver.com/helloworld/1329](https://d2.naver.com/helloworld/1329)
