---
tags: 
    - java
    - mongodb
    - mongo
    - spring data
    - aggregation
title: Spring Data MongoDB - ConditionalOperators
date: 2021/08/20
author: 김동환
description: ConditionalOperators ($cond, $switch, $ifNull)
disabled: false
categories:
  - java
---
　
# $cond

## Value

```java
{ "_id" : 1, "item" : "abc1", qty: 300 }
{ "_id" : 2, "item" : "abc2", qty: 200 }
{ "_id" : 3, "item" : "xyz1", qty: 250 }
```

```java
/*
    db.inventory.aggregate(
       [
          {
             $project:
               {
                 item: 1,
                 discount:
                   {
                     $cond: { if: { $gte: [ "$qty", 250 ] }, then: 30, else: 20 }
                   }
               }
          }
       ]
    )
*/
@Test
public void 물품_수량에_맞춰_할인율_설정하기() {
    Criteria condition = Criteria.where("qty").gte(250);
    AggregationExpression conditionExpression = ConditionalOperators.when(condition)
        .then(30)
        .otherwise(20);
    
    ProjectionOperation projectionOperation = Aggregation.project("item")
        .and(conditionExpression).as("discount");
    
    Aggregation aggregation = Aggregation.newAggregation(projectionOperation);
    
    mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/* Results
    { "_id" : 1, "item" : "abc1", "discount" : 30 }
    { "_id" : 2, "item" : "abc2", "discount" : 20 }
    { "_id" : 3, "item" : "xyz1", "discount" : 30 }
*/
```

## Field

| `thenValueOf` , `otherwiseValueOf` 를 통해서 필드를 삽입할 수 있다.

```java
{ "_id" : 1, "item" : "abc1", "expiration_date": ISODate("2021-06-12T13:21:43Z")}
{ "_id" : 2, "item" : "abc2", "expiration_date": ISODate("2021-06-25T18:38:12Z")}
{ "_id" : 3, "item" : "xyz1", "expiration_date**"**: ISODate("2021-07-21T08:20:28Z")}
```

```java
/*
    db.inventory.aggregate(
       [
          {
             $project:
               {
                 item: 1,
                 expired:
                   {
                     $cond: { if: { $gte: [ "$expiration_date", "$$NOW" ] }, then: "$expiration_date", else: "EXPIRED" }
                   }
               }
          }
       ]
    )
*/
@Test
public void 물품_유통기한_상태_필드_표기하기() {
    Criteria condition = Criteria.where("expiration_date").gte(new Date());
    AggregationExpression conditionExpression = ConditionalOperators.when(condition)
        .thenValueOf("expiration_date")
        .otherwise("EXPIRED");
    
    ProjectionOperation projectionOperation = Aggregation.project("item")
        .and(conditionExpression).as("expired");
    
    Aggregation aggregation = Aggregation.newAggregation(projectionOperation);
    
    mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/* Results
    { "_id" : 1, "item" : "abc1", "expired" : "EXPIRED" }
    { "_id" : 2, "item" : "abc2", "expired" : "EXPIRED" }
    { "_id" : 3, "item" : "xyz1", "expired" : ISODate("2021-07-21T08:20:28Z") }
*/
```

## Aggregation Expression

| `thenValueOf` , `otherwiseValueOf` 를 통해서 표현식을 삽입할 수 있다.

```java
{ "_id" : 1, "item" : "abc1", qty: 500 }
{ "_id" : 2, "item" : "abc2", qty: 2000 }
{ "_id" : 3, "item" : "abc3", qty: 95 }
{ "_id" : 1, "item" : "abc4", qty: 0 }
{ "_id" : 2, "item" : "abc5", qty: 200 }
```

```java
/*
    db.inventory.aggregate(
       [
          {
             "$project":
                {
                   "item": 1,
                   "state":
                      {
                         "$cond": {"if": {"$gte": ["$qty", 100]},
                            "then": {"$cond": {"if": {"$gte": ["$qty", 1000]},
                               "then": "GOOD",
                               "else": "WARNING"}},
                            "else": {"$cond": {"if": {"$eq": ["$qty", 0]},
                               "then": "CRITICAL",
                               "else": "EMPTY"}}}
                      }
                }
          }
       ]
    )
*/
@Test
public void 물품_수량_상태_표기하기() {
    AggregationExpression conditionExpression = ConditionalOperators.when(Criteria.where("qty").gte(100))
        .thenValueOf(ConditionalOperators.when(Criteria.where("qty").gte(1000))
            .then("GOOD")
            .otherwise("WARNING"))
        .otherwiseValueOf(ConditionalOperators.when(Criteria.where("qty").is(0))
            .then("EMPTY")
            .otherwise("CRITICAL"));
    
    ProjectionOperation projectionOperation = Aggregation.project("item")
        .and(conditionExpression).as("state");
    
    Aggregation aggregation = Aggregation.newAggregation(projectionOperation);
    
    mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/* Results
    { "_id" : 1, "item" : "abc1", "state": "WARNING"}
    { "_id" : 2, "item" : "abc2", "state": "GOOD"}
    { "_id" : 3, "item" : "abc3", "state": "CRITICAL"}
    { "_id" : 1, "item" : "abc4", "state": "EMPTY"}
    { "_id" : 2, "item" : "abc5", "state": "WARNING"}
*/
```

# $switch

| `then` 를 통해서 `value, field, aggregation expression` 를 삽입할 수 있다.

```java
{ "_id" : 1, "item" : "abc1", qty: 500 }
{ "_id" : 2, "item" : "abc2", qty: 2000 }
{ "_id" : 3, "item" : "abc3", qty: 95 }
{ "_id" : 1, "item" : "abc4", qty: 0 }
{ "_id" : 2, "item" : "abc5", qty: 200 }
```

```java
/*
    db.inventory.aggregate(
       [
          {
             "$project":
                {
                   "item": 1,
                   "state":
                      {
                         "$switch": {"branches":
                            [
                               {"case": {"$gte": ["$qty", 1000]}, "then": "GOOD"},
                               {"case": {"$gte": ["$qty", 100]}, "then": "WARNING"},
                               {"case": {"$eq": ["$qty", 0]}, "then": "EMPTY"}
                            ], 
                                                    "default": "CRITICAL"
                      }
                }
          }
       ]
    )
*/
@Test
public void 물품_수량_상태_표기하기_USING_SWITCH_CASE() {
    AggregationExpression conditionExpression = ConditionalOperators.switchCases(
        ConditionalOperators.Switch.CaseOperator.when(ComparisonOperators.valueOf("qty").greaterThanEqualToValue(1000)).then("GOOD"),
        ConditionalOperators.Switch.CaseOperator.when(ComparisonOperators.valueOf("qty").greaterThanEqualToValue(100)).then("WARNING"),
        ConditionalOperators.Switch.CaseOperator.when(ComparisonOperators.valueOf("qty").equalToValue(0)).then("EMPTY")).defaultTo("CRITICAL");

    ProjectionOperation projectionOperation = Aggregation.project("item")
        .and(conditionExpression).as("state");

    Aggregation aggregation = Aggregation.newAggregation(projectionOperation);
    
    mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/* Results
    { "_id" : 1, "item" : "abc1", "state": "WARNING"}
    { "_id" : 2, "item" : "abc2", "state": "GOOD"}
    { "_id" : 3, "item" : "abc3", "state": "CRITICAL"}
    { "_id" : 1, "item" : "abc4", "state": "EMPTY"}
    { "_id" : 2, "item" : "abc5", "state": "WARNING"}
*/
```

# $IfNull

| `thenValueOf` 를 통해서 필드 및 표현식을 삽입할 수 있다.

```java
{ "_id" : 1, "item" : "buggy", description: "toy car", "quantity" : 300 },
{ "_id" : 2, "item" : "bicycle", description: null, "quantity" : 200 
{ "_id" : 3, "item" : "flag" }
```

```java
/*
    db.inventory.aggregate(
       [
          {
             $project: {
                item: 1,
                description: { $ifNull: [ "$description", "Unspecified" ] }
             }
          }
       ]
    )
*/
@Test
public void Null일_경우_문자열_삽입 () {
    AggregationExpression conditionExpression = ConditionalOperators.ifNull("description").then("Unspecified");

    ProjectionOperation projectionOperation = Aggregation.project("item")
        .and(conditionExpression).as("description");

    Aggregation aggregation = Aggregation.newAggregation(projectionOperation);

    mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/* Results
    { "_id" : 1, "item" : "buggy", "description" : "toy car" }
    { "_id" : 2, "item" : "bicycle", "description" : "Unspecified" }
    { "_id" : 3, "item" : "flag", "description" : "Unspecified" }
*/
```