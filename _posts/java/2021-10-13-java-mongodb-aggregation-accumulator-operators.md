---
tags: 
    - java
    - mongodb
    - mongo
    - spring data
    - aggregation
title: Spring Data MongoDB - AccumulatorOperators
date: 2021/10/13
author: 김동환
description: AccumulatorOperators ($sum, $avg, $max, $min, $stdDevPop, $stdDevSamp)
disabled: false
categories:
  - java
---
　
# $sum

> 총합
>

```java
{ "_id": 1, "quizzes": [ 10, 6, 7 ], "labs": [ 5, 8 ], "final": 80, "midterm": 75 }
{ "_id": 2, "quizzes": [ 9, 10 ], "labs": [ 8, 8 ], "final": 95, "midterm": 80 }
{ "_id": 3, "quizzes": [ 4, 5, 5 ], "labs": [ 6, 5 ], "final": 78, "midterm": 70 }
```

```java
/*
    db.students.aggregate([
      {
        $project: {
          quizTotal: { $sum: "$quizzes"},
          labTotal: { $sum: "$labs" },
          examTotal: { $sum: [ "$final", "$midterm" ] }
        }
      }
    ])
*/
@Test
public void 각_점수_총합_구하기() {
    ProjectionOperation projectionOperation = Aggregation.project()
        .and(AccumulatorOperators.valueOf("quizzes").sum()).as("quizTotal")
        .and(AccumulatorOperators.valueOf("labs").sum()).as("labTotal")
        .and(AccumulatorOperators.valueOf("final").sum().and("midterm")).as("examTotal");
    
    Aggregation aggregation = Aggregation.newAggregation(
        projectionOperation
    );
    
    mongoTemplate.aggregate(aggregation, "students", Student.class);
}

/*
    { "_id" : 1, "quizTotal" : 23, "labTotal" : 13, "examTotal" : 155 }
    { "_id" : 2, "quizTotal" : 19, "labTotal" : 16, "examTotal" : 175 }
    { "_id" : 3, "quizTotal" : 14, "labTotal" : 11, "examTotal" : 148 }
*/
```

# $avg

> 평균
>

```java
{ "_id": 1, "quizzes": [ 10, 6, 7 ], "labs": [ 5, 8 ], "final": 80, "midterm": 75 }
{ "_id": 2, "quizzes": [ 9, 10 ], "labs": [ 8, 8 ], "final": 95, "midterm": 80 }
{ "_id": 3, "quizzes": [ 4, 5, 5 ], "labs": [ 6, 5 ], "final": 78, "midterm": 70 }
```

```java
/*
    db.students.aggregate([
      {
        $project: {
          quizAvg: { $avg: "$quizzes"},
          labAvg: { $avg: "$labs" },
          examAvg: { $avg: [ "$final", "$midterm" ] }
        }
      }
    ])
*/
@Test
public void 각_점수_평균_구하기() {
    ProjectionOperation projectionOperation = Aggregation.project()
        .and(AccumulatorOperators.valueOf("quizzes").avg()).as("quizAvg")
        .and(AccumulatorOperators.valueOf("labs").avg()).as("labAvg")
        .and(AccumulatorOperators.valueOf("final").avg().and("midterm")).as("examAvg");

    Aggregation aggregation = Aggregation.newAggregation(
        projectionOperation
    );
    
    mongoTemplate.aggregate(aggregation, "students", Student.class);
}

/*
    { "_id" : 1, "quizAvg" : 7.666666666666667, "labAvg" : 6.5, "examAvg" : 77.5 }
    { "_id" : 2, "quizAvg" : 9.5, "labAvg" : 8, "examAvg" : 87.5 }
    { "_id" : 3, "quizAvg" : 4.666666666666667, "labAvg" : 5.5, "examAvg" : 74 }
*/
```

# $max

> 최대값
>

```java
{ "_id": 1, "quizzes": [ 10, 6, 7 ], "labs": [ 5, 8 ], "final": 80, "midterm": 75 }
{ "_id": 2, "quizzes": [ 9, 10 ], "labs": [ 8, 8 ], "final": 95, "midterm": 80 }
{ "_id": 3, "quizzes": [ 4, 5, 5 ], "labs": [ 6, 5 ], "final": 78, "midterm": 70 }
```

```java
/*
    db.students.aggregate([
      {
        $project: {
          quizMax: { $max: "$quizzes"},
          labMax: { $max: "$labs" },
          examMax: { $max: [ "$final", "$midterm" ] }
        }
      }
    ])
*/
@Test
public void 각_점수_최대값_구하기() {
    ProjectionOperation projectionOperation = Aggregation.project()
        .and(AccumulatorOperators.valueOf("quizzes").max()).as("quizMax")
        .and(AccumulatorOperators.valueOf("labs").max()).as("labMax")
        .and(AccumulatorOperators.valueOf("final").max().and("midterm")).as("examMax");
    
    Aggregation aggregation = Aggregation.newAggregation(
        projectionOperation
    );
    
    mongoTemplate.aggregate(aggregation, "students", Student.class);
}

/*
    { "_id" : 1, "quizMax" : 10, "labMax" : 8, "examMax" : 80 }
    { "_id" : 2, "quizMax" : 10, "labMax" : 8, "examMax" : 95 }
    { "_id" : 3, "quizMax" : 5, "labMax" : 6, "examMax" : 78 }
*/
```

# $min

> 최소값
>

```java
{ "_id": 1, "quizzes": [ 10, 6, 7 ], "labs": [ 5, 8 ], "final": 80, "midterm": 75 }
{ "_id": 2, "quizzes": [ 9, 10 ], "labs": [ 8, 8 ], "final": 95, "midterm": 80 }
{ "_id": 3, "quizzes": [ 4, 5, 5 ], "labs": [ 6, 5 ], "final": 78, "midterm": 70 }
```

```java
/*
    db.students.aggregate([
      {
        $project: {
          quizMin: { $min: "$quizzes"},
          labMin: { $min: "$labs" },
          examMin: { $min: [ "$final", "$midterm" ] }
        }
      }
    ])
*/
@Test
public void 각_점수_최소값_구하기() {
    ProjectionOperation projectionOperation = Aggregation.project()
        .and(AccumulatorOperators.valueOf("quizzes").min()).as("quizMin")
        .and(AccumulatorOperators.valueOf("labs").min()).as("labMin")
        .and(AccumulatorOperators.valueOf("final").min().and("midterm")).as("examMin");
    
    Aggregation aggregation = Aggregation.newAggregation(
        projectionOperation
    );
    
    mongoTemplate.aggregate(aggregation, "students", Student.class);
}

/*
    { "_id" : 1, "quizMin" : 6, "labMin" : 5, "examMin" : 75 }
    { "_id" : 2, "quizMin" : 9, "labMin" : 8, "examMin" : 80 }
    { "_id" : 3, "quizMin" : 4, "labMin" : 5, "examMin" : 70 }
*/
```

# $stdDevPop

> 모집단 표준편차
>

```java
{ 
  "_id" : 1,
  "scores" : [
    { "name" : "dave123", "score" : 85 },
    { "name" : "dave2", "score" : 90 },
    { "name" : "ahn", "score" : 71 }
  ]
}, {
  "_id" : 2,
  "scores" : [
    { "name" : "li", "quiz" : 2, "score" : 96 },
    { "name" : "annT", "score" : 77 },
    { "name" : "ty", "score" : 82 }
  ]
}
```

```java
/*
    db.quizzes.aggregate([
      { 
        $project: { 
          stdDev: { $stdDevPop: "$scores.score" } 
        } 
      }
    ])
*/
@Test
public void 퀴즈_점수_표준편차_구하기() {
    ProjectionOperation projectionOperation = Aggregation.project()
        .and(AccumulatorOperators.valueOf("scores.score").stdDevPop()).as("stdDev");
    
    Aggregation aggregation = Aggregation.newAggregation(
        projectionOperation
    );
    
    mongoTemplate.aggregate(aggregation, "quizzes", Quiz.class);
}

/*
    { "_id" : 1, "stdDev" : 8.04155872120988 }
    { "_id" : 2, "stdDev" : 8.04155872120988 }
*/
```

# $stdDevSamp

> 표본 표준편차
>

```java
{ 
  "_id" : 1,
  "scores" : [
    { "name" : "dave123", "score" : 85 },
    { "name" : "dave2", "score" : 90 },
    { "name" : "ahn", "score" : 71 }
  ]
}, {
  "_id" : 2,
  "scores" : [
    { "name" : "li", "quiz" : 2, "score" : 96 },
    { "name" : "annT", "score" : 77 },
    { "name" : "ty", "score" : 82 }
  ]
}
```

```java
/*
    db.quizzes.aggregate([
      { 
        $project: { 
          stdDevSamp: { $stdDevSamp: "$scores.score" } 
        } 
      }
    ])
*/
@Test
public void 퀴즈_점수_표본_표준편차_구하기() {
    ProjectionOperation projectionOperation = Aggregation.project()
        .and(AccumulatorOperators.valueOf("scores.score").stdDevSamp()).as("stdDevSamp");
    
    Aggregation aggregation = Aggregation.newAggregation(
        projectionOperation
    );
    
    mongoTemplate.aggregate(aggregation, "quizzes", Quiz.class);
}

/*
    { "_id" : 1, "stdDev" : 9.848857801796104 }
    { "_id" : 2, "stdDev" : 9.848857801796104 }
*/
```