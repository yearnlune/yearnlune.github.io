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
              comparisonResult: { $strcasecmp: [ "$quarter", "13q4" ] }
            }
        }
    ]
  )
*/
@Test
public void strcasecmp() {
  ProjectionOperation projectionOperation = Aggregation.project()
    .andInclude("item")
    .and(StringOperators.valueOf("quarter").strCaseCmp("13q4")).as("comparisonResult");

  Aggregation aggregation = Aggregation.newAggregation(
    projectionOperation
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "item" : "ABC1", "comparisonResult" : -1 }
	{ "_id" : 2, "item" : "ABC2", "comparisonResult" : 0 }
	{ "_id" : 3, "item" : "XYZ1", "comparisonResult" : 1 }
*/
```

# $indexOfBytes

```java
{ "_id" : 1, "item" : "foo" }
{ "_id" : 2, "item" : "fóofoo" }
{ "_id" : 3, "item" : "the foo bar" }
{ "_id" : 4, "item" : "hello world fóo" }
{ "_id" : 5, "item" : null }
{ "_id" : 6, "amount" : 3 }
```

```java
/*
  db.inventory.aggregate(
    [
      {
        $project:
            {
              byteLocation: { $indexOfBytes: [ "$item", "foo" ] },
            }
        }
    ]
  )
*/
@Test
public void indexOfBytes() {
  ProjectionOperation projectionOperation = Aggregation.project()
    .and(StringOperators.valueOf("item").indexOf("foo")).as("byteLocation");

  Aggregation aggregation = Aggregation.newAggregation(
    projectionOperation
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "byteLocation" : "0" }
	{ "_id" : 2, "byteLocation" : "4" }
	{ "_id" : 3, "byteLocation" : "4" }
	{ "_id" : 4, "byteLocation" : "-1" }
	{ "_id" : 5, "byteLocation" : null }
	{ "_id" : 6, "byteLocation" : null }
*/

```

# $indexOfCP

```java
{ "_id" : 1, "item" : "foo" }
{ "_id" : 2, "item" : "fóofoo" }
{ "_id" : 3, "item" : "the foo bar" }
{ "_id" : 4, "item" : "hello world fóo" }
{ "_id" : 5, "item" : null }
{ "_id" : 6, "amount" : 3 }
```

```java
/*
  db.inventory.aggregate(
    [
      {
        $project:
            {
              cpLocation: { $indexOfCP: [ "$item", "foo" ] },
            }
        }
    ]
  )
*/
@Test
public void indexOfCP() {
  ProjectionOperation projectionOperation = Aggregation.project()
    .and(StringOperators.valueOf("item").indexOfCP("foo")).as("cpLocation");

  Aggregation aggregation = Aggregation.newAggregation(
    projectionOperation
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "cpLocation" : "0" }
	{ "_id" : 2, "cpLocation" : "3" }
	{ "_id" : 3, "cpLocation" : "4" }
	{ "_id" : 4, "cpLocation" : "-1" }
	{ "_id" : 5, "cpLocation" : null }
	{ "_id" : 6, "cpLocation" : null }
*/
```

# $split

```java
{ "_id" : 1, "country": "NA", "city" : "Berkeley, CA", "qty" : 648 }
{ "_id" : 2, "country": "NA", "city" : "Bend, OR", "qty" : 491 }
{ "_id" : 3, "country": "NA", "city" : "Kensington, CA", "qty" : 233 }
{ "_id" : 4, "country": "NA", "city" : "Eugene, OR", "qty" : 842 }
{ "_id" : 5, "country": "NA", "city" : "Reno, NV", "qty" : 655 }
{ "_id" : 6, "country": "NA", "city" : "Portland, OR", "qty" : 408 }
{ "_id" : 7, "country": "NA", "city" : "Sacramento, CA", "qty" : 574 }
```

```java
/*
  db.deliveries.aggregate([
    { $project : { country: 1, city_state : { $split: ["$city", ", "] }, qty : 1 } },
    { $unwind : "$city_state" },
    { $match : { city_state : /[A-Z]{2}/ } },
    { $group : { _id: { "country" : "$country", "state" : "$city_state" }, total_qty : { "$sum" : "$qty" } } },
    { $sort : { total_qty : -1 } }
  ]);
*/
@Test
public void split() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("country")
      .and(StringOperators.valueOf("city").split(", ")).as("city_state")
      .andInclude("qty"),
    Aggregation.unwind("city_state"),
    Aggregation.match(Criteria.where("city").regex("[A-Z]{2}")),
    Aggregation.group(Fields.fields()
        .and("country")
        .and("state", "city_state"))
      .sum("qty").as("total_qty"),
    Aggregation.sort(Sort.Direction.DESC, "total_qty")
  );

  mongoTemplate.aggregate(aggregation, "deliveries", Delivery.class);
}

/*
	{ "_id" : { "country": "NA", "state" : "OR" }, "total_qty" : 1741 }
	{ "_id" : { "country": "NA", "state" : "CA" }, "total_qty" : 1455 }
	{ "_id" : { "country": "NA", "state" : "NV" }, "total_qty" : 655 }
*/
```