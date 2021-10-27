---
tags: 
    - java
    - mongodb
    - mongo
    - spring data
    - aggregation
title: Spring Data MongoDB - StringOperators
date: 2021/10/27
author: 김동환
description: StringOperators ($concat, $substr, $toLower, $toUpper, $strcasecmp, $split, $strlenBytes, $strlenCP, $trim)
disabled: true
categories:
  - java
---
　
> $concat, $substr, $toLower, $toUpper, $strcasecmp, $split, $strlenBytes, $strlenCP, $trim 등 String과 연관된 operator를 사용할 수 있다.
>

# $concat

```java
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "product 1" }
{ "_id" : 2, "item" : "ABC2", quarter: "13Q4", "description" : "product 2" }
{ "_id" : 3, "item" : "XYZ1", quarter: "14Q2", "description" : null }
```

```java
/*
  db.inventory.aggregate(
    [
        { $project: { itemDescription: { $concat: [ "$item", " - ", "$description" ] } } }
    ]
  )
*/
@Test
public void concat() {
  ProjectionOperation projectionOperation = Aggregation.project()
    .and(StringOperators.valueOf("item")
      .concat("-")
      .concatValueOf("description")).as("itemDescription");

  Aggregation aggregation = Aggregation.newAggregation(
    projectionOperation
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "itemDescription" : "ABC1 - product 1" }
	{ "_id" : 2, "itemDescription" : "ABC2 - product 2" }
	{ "_id" : 3, "itemDescription" : null }
*/
```

# $substr

```java
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "product 1" }
{ "_id" : 2, "item" : "ABC2", quarter: "13Q4", "description" : "product 2" }
{ "_id" : 3, "item" : "XYZ1", quarter: "14Q2", "description" : null }
```

```java
/*
  db.inventory.aggregate(
    [
      {
        $project:
            {
              item: 1,
              yearSubstring: { $substr: [ "$quarter", 0, 2 ] },
              quarterSubtring: { $substr: [ "$quarter", 2, -1 ] }
            }
      }
    ]
  )
*/
@Test
public void substr() {
  ProjectionOperation projectionOperation = Aggregation.project()
    .andInclude("item")
    .and(StringOperators.valueOf("quarter")
      .substring(0, 2)).as("yearSubstring")
    .and(StringOperators.valueOf("quarter")
      .substring(2, -1)).as("quarterSubtring");

  Aggregation aggregation = Aggregation.newAggregation(
    projectionOperation
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "item" : "ABC1", "yearSubstring" : "13", "quarterSubtring" : "Q1" }
	{ "_id" : 2, "item" : "ABC2", "yearSubstring" : "13", "quarterSubtring" : "Q4" }
	{ "_id" : 3, "item" : "XYZ1", "yearSubstring" : "14", "quarterSubtring" : "Q2" }
*/
```

# $toLower

```java
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "PRODUCT 1" }
{ "_id" : 2, "item" : "abc2", quarter: "13Q4", "description" : "Product 2" }
{ "_id" : 3, "item" : "xyz1", quarter: "14Q2", "description" : null }
```

```java
/*
  db.inventory.aggregate(
    [
      {
        $project:
          {
            item: { $toLower: "$item" },
            description: { $toLower: "$description" }
          }
      }
    ]
  )
*/
@Test
public void toLower() {
  ProjectionOperation projectionOperation = Aggregation.project()
    .and(StringOperators.valueOf("item").toLower()).as("item")
    .and(StringOperators.valueOf("description").toLower()).as("description");

  Aggregation aggregation = Aggregation.newAggregation(
    projectionOperation
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "item" : "abc1", "description" : "product 1" }
	{ "_id" : 2, "item" : "abc2", "description" : "product 2" }
	{ "_id" : 3, "item" : "xyz1", "description" : "" }
*/
```

# $toUpper

```java
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "PRODUCT 1" }
{ "_id" : 2, "item" : "abc2", quarter: "13Q4", "description" : "Product 2" }
{ "_id" : 3, "item" : "xyz1", quarter: "14Q2", "description" : null }
```

```java
/*
  db.inventory.aggregate(
    [
      {
        $project:
          {
            item: { $toUpper: "$item" },
            description: { $toUpper: "$description" }
          }
      }
    ]
  )
*/
@Test
public void toLower() {
  ProjectionOperation projectionOperation = Aggregation.project()
    .and(StringOperators.valueOf("item").toUpper()).as("item")
    .and(StringOperators.valueOf("description").toUpper()).as("description");

  Aggregation aggregation = Aggregation.newAggregation(
    projectionOperation
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "item" : "ABC1", "description" : "PRODUCT 1" }
	{ "_id" : 2, "item" : "ABC2", "description" : "PRODUCT 2" }
	{ "_id" : 3, "item" : "XYZ1", "description" : "" }
*/
```