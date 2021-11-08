---
tags: 
    - java
    - mongodb
    - mongo
    - spring data
    - aggregation
title: Spring Data MongoDB - ComparisonOperators
date: 2021/11/04
author: 김동환
description: ComparisonOperators ($cmp, $eq, $gt, $gte, $lt, $lte, $ne)
disabled: false
categories:
  - java
---

> $cmp, $eq, $gt, $gte, $lt, $lte, $ne 등 비교연산자와 관련된 연산자를 사용할 수 있다.
>

# $cmp

> `compareToValue` , `compareTo` 를 통해서 $cmp를 사용할 수 있다.
>

```java
{ "_id" : 1, "item" : "abc1", description: "product 1", width: 300, height: 250 }
{ "_id" : 2, "item" : "abc2", description: "product 2", width: 200, height: 250 }
{ "_id" : 3, "item" : "xyz1", description: "product 3", width: 250, height: 250 }
{ "_id" : 4, "item" : "VWZ1", description: "product 4", width: 300, height: 250 }
{ "_id" : 5, "item" : "VWZ2", description: "product 5", width: 180, height: 250 }
```

```java
/*
  db.inventory.aggregate(
    [
      {
      $project:
        {
	        item: 1,
	        width: 1,
					cmpTo250: { $cmp: [ "$width", 250 ] },
	        cmpToHeight: { $cmp: [ "$width", "$height" ] },
	        _id: 0
        }
      }
    ]
  )
*/
@Test
public void cmp() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .andInclude("width")
      .and(ComparisonOperators.valueOf("width").compareToValue(250)).as("cmpTo250")
      .and(ComparisonOperators.valueOf("width").compareTo("height")).as("cmpToHeight")
      .andExclude("_id")
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
  { "item" : "abc1", "width" : 300, "cmpTo250" : 1, "cmpToHeight" : 1 }
  { "item" : "abc2", "width" : 200, "cmpTo250" : -1, "cmpToHeight" : -1 }
  { "item" : "xyz1", "width" : 250, "cmpTo250" : 0, "cmpToHeight" : 0 }
  { "item" : "VWZ1", "width" : 300, "cmpTo250" : 1, "cmpToHeight" : 1 }
  { "item" : "VWZ2", "width" : 180, "cmpTo250" : -1, "cmpToHeight" : -1 }
*/
```

# $eq

> `equalToValue`, `equalTo`를 통해서 $eq를 사용할 수 있다.
>

```java
{ "_id" : 1, "item" : "abc1", description: "product 1", width: 300, height: 250 }
{ "_id" : 2, "item" : "abc2", description: "product 2", width: 200, height: 250 }
{ "_id" : 3, "item" : "xyz1", description: "product 3", width: 250, height: 250 }
{ "_id" : 4, "item" : "VWZ1", description: "product 4", width: 300, height: 250 }
{ "_id" : 5, "item" : "VWZ2", description: "product 5", width: 180, height: 250 }
```

```java
/*
  db.inventory.aggregate(
    [
      {
        $project:
            {
              item: 1,
              width: 1,
              widthEq250: { $eq: [ "$width", 250 ] },
              widthEqHeight: { $eq: [ "$width", "$height" ] },
              _id: 0
            }
      }
    ]
  )
*/
@Test
public void eq() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .andInclude("width")
      .and(ComparisonOperators.valueOf("width").equalToValue(250)).as("widthEq250")
      .and(ComparisonOperators.valueOf("width").equalTo("height")).as("widthEqHeight")
      .andExclude("_id")
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}
/*
  { "item" : "abc1", "width" : 300, "widthEq250" : false, "widthEqHeight" :false }
  { "item" : "abc2", "width" : 200, "widthEq250" : false, "widthEqHeight" : false }
  { "item" : "xyz1", "width" : 250, "widthEq250" : true, "widthEqHeight" : true }
  { "item" : "VWZ1", "width" : 300, "widthEq250" : false, "widthEqHeight" : false }
  { "item" : "VWZ2", "width" : 180, "widthEq250" : false, "widthEqHeight" : false }
*/
```

# $gt, $gte

```java
{ "_id" : 1, "item" : "abc1", description: "product 1", width: 300, height: 250 }
{ "_id" : 2, "item" : "abc2", description: "product 2", width: 200, height: 250 }
{ "_id" : 3, "item" : "xyz1", description: "product 3", width: 250, height: 250 }
{ "_id" : 4, "item" : "VWZ1", description: "product 4", width: 300, height: 250 }
{ "_id" : 5, "item" : "VWZ2", description: "product 5", width: 180, height: 250 }
```

```java
/*
  db.inventory.aggregate(
    [
      {
        $project:
            {
              item: 1,
              width: 1,
              widthGt250: { $gt: [ "$width", 250 ] },
              widthGtHeight: { $gt: [ "$width", "$height" ] },
              widthGte250: { $gte: [ "$width", 250 ] },
              widthGteHeight: { $gte: [ "$width", "$height" ] },
              _id: 0
            }
      }
    ]
  )
*/
@Test
public void gte() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .andInclude("width")
      .and(ComparisonOperators.valueOf("width").greaterThanValue(250)).as("widthGt250")
      .and(ComparisonOperators.valueOf("width").greaterThan("height")).as("widthGtHeight")
      .and(ComparisonOperators.valueOf("width").greaterThanEqualToValue(250)).as("widthGte250")
      .and(ComparisonOperators.valueOf("width").greaterThanEqualTo("height")).as("widthGteHeight")
      .andExclude("_id")
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}
/*
  { "item" : "abc1", "width" : 300, "widthGt250" : true, "widthGtHeight" :true, "widthGte250" :true, "widthGteHeight" :true }
  { "item" : "abc2", "width" : 200, "widthGt250" : false, "widthGtHeight" :false, "widthGte250" :false, "widthGteHeight" :false }
  { "item" : "xyz1", "width" : 250, "widthGt250" : false, "widthGtHeight" :false, "widthGte250" :true, "widthGteHeight" :true }
  { "item" : "VWZ1", "width" : 300, "widthGt250" : true, "widthGtHeight" :true, "widthGte250" :true, "widthGteHeight" :true }
  { "item" : "VWZ2", "width" : 180, "widthGt250" : false, "widthGtHeight" :false, "widthGte250" :false, "widthGteHeight" :false }
*/
```

# $lt, $lte

```java
{ "_id" : 1, "item" : "abc1", description: "product 1", width: 300, height: 250 }
{ "_id" : 2, "item" : "abc2", description: "product 2", width: 200, height: 250 }
{ "_id" : 3, "item" : "xyz1", description: "product 3", width: 250, height: 250 }
{ "_id" : 4, "item" : "VWZ1", description: "product 4", width: 300, height: 250 }
{ "_id" : 5, "item" : "VWZ2", description: "product 5", width: 180, height: 250 }
```

```java
/*
  db.inventory.aggregate(
    [
      {
        $project:
            {
              item: 1,
              width: 1,
              widthLt250: { $lt: [ "$width", 250 ] },
              widthLtHeight: { $lt: [ "$width", "$height" ] },
              widthLte250: { $lte: [ "$width", 250 ] },
              widthLteHeight: { $lte: [ "$width", "$height" ] },
              _id: 0
            }
      }
    ]
  )
*/
@Test
public void lte() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .andInclude("width")
      .and(ComparisonOperators.valueOf("width").lessThanValue(250)).as("widthLt250")
      .and(ComparisonOperators.valueOf("width").lessThan("height")).as("widthLtHeight")
      .and(ComparisonOperators.valueOf("width").lessThanEqualToValue(250)).as("widthLte250")
      .and(ComparisonOperators.valueOf("width").lessThanEqualTo("height")).as("widthLteHeight")
      .andExclude("_id")
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}
/*
  { "item" : "abc1", "width" : 300, "widthLt250" : false, "widthLtHeight" :false, "widthLte250" :false, "widthLteHeight" :false }
  { "item" : "abc2", "width" : 200, "widthLt250" : true, "widthLtHeight" :true, "widthLte250" :true, "widthLteHeight" :true }
  { "item" : "xyz1", "width" : 250, "widthLt250" : false, "widthLtHeight" :false, "widthLte250" :true, "widthLteHeight" :true }
  { "item" : "VWZ1", "width" : 300, "widthLt250" : false, "widthLtHeight" :false, "widthLte250" :false, "widthLteHeight" :false }
  { "item" : "VWZ2", "width" : 180, "widthLt250" : true, "widthLtHeight" :true, "widthLte250" :true, "widthLteHeight" :true }
*/
```

# $ne

```java
{ "_id" : 1, "item" : "abc1", description: "product 1", width: 300, height: 250 }
{ "_id" : 2, "item" : "abc2", description: "product 2", width: 200, height: 250 }
{ "_id" : 3, "item" : "xyz1", description: "product 3", width: 250, height: 250 }
{ "_id" : 4, "item" : "VWZ1", description: "product 4", width: 300, height: 250 }
{ "_id" : 5, "item" : "VWZ2", description: "product 5", width: 180, height: 250 }
```

```java
/*
  db.inventory.aggregate(
    [
      {
        $project:
            {
              item: 1,
              width: 1,
              widthNe250: { $ne: [ "$width", 250 ] },
              widthNeHeight: { $ne: [ "$width", "$height" ] },
              _id: 0
            }
      }
    ]
  )
*/
@Test
public void ne() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .andInclude("width")
      .and(ComparisonOperators.valueOf("width").notEqualToValue(250)).as("widthNe250")
      .and(ComparisonOperators.valueOf("width").notEqualTo("height")).as("widthNeHeight")
      .andExclude("_id")
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}
/*
  { "item" : "abc1", "width" : 300, "widthNe250" : true, "widthNeHeight" :true }
  { "item" : "abc2", "width" : 200, "widthNe250" : true, "widthNeHeight" : true }
  { "item" : "xyz1", "width" : 250, "widthNe250" : false, "widthNeHeight" : false }
  { "item" : "VWZ1", "width" : 300, "widthNe250" : true, "widthNeHeight" : true }
  { "item" : "VWZ2", "width" : 180, "widthNe250" : true, "widthNeHeight" : true }
*/
```