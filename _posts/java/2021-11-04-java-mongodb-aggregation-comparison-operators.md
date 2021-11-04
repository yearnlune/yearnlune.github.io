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
  { "item" : "abc2", "width" : 200, "cmpTo250" : -1 "cmpToHeight" : -1 }
  { "item" : "xyz1", "width" : 250, "cmpTo250" : 0 "cmpToHeight" : 0 }
  { "item" : "VWZ1", "width" : 300, "cmpTo250" : 1 "cmpToHeight" : 1 }
  { "item" : "VWZ2", "width" : 180, "cmpTo250" : -1 "cmpToHeight" : -1 }
*/
```