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
disabled: false
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
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .and(StringOperators.valueOf("item")
        .concat("-")
        .concatValueOf("description")).as("itemDescription")
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
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .and(StringOperators.valueOf("quarter")
        .substring(0, 2)).as("yearSubstring")
      .and(StringOperators.valueOf("quarter")
        .substring(2, -1)).as("quarterSubtring")
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
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .and(StringOperators.valueOf("item").toLower()).as("item")
      .and(StringOperators.valueOf("description").toLower()).as("description")
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
public void toUpper() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .and(StringOperators.valueOf("item").toUpper()).as("item")
      .and(StringOperators.valueOf("description").toUpper()).as("description")
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "item" : "ABC1", "description" : "PRODUCT 1" }
	{ "_id" : 2, "item" : "ABC2", "description" : "PRODUCT 2" }
	{ "_id" : 3, "item" : "XYZ1", "description" : "" }
*/
```

# $strcasecmp

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
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .and(StringOperators.valueOf("quarter").strCaseCmp("13q4")).as("comparisonResult")
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
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .and(StringOperators.valueOf("item").indexOf("foo")).as("byteLocation")
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
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .and(StringOperators.valueOf("item").indexOfCP("foo")).as("cpLocation")
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

# $strLenBytes

```java
{ "_id" : 1, "name" : "apple" }
{ "_id" : 2, "name" : "banana" }
{ "_id" : 3, "name" : "éclair" }
{ "_id" : 4, "name" : "hamburger" }
{ "_id" : 5, "name" : "jalapeño" }
{ "_id" : 6, "name" : "pizza" }
{ "_id" : 7, "name" : "tacos" }
{ "_id" : 8, "name" : "寿司" }
```

```java
/*
  db.food.aggregate(
    [
      {
        $project: {
          "name": 1,
          "length": { $strLenBytes: "$name" }
        }
      }
    ]
  )
*/
@Test
public void strLenBytes() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("name")
      .and(StringOperators.StrLenBytes.stringLengthOf("name")).as("length")
  );

  mongoTemplate.aggregate(aggregation, "food", Food.class);
}

/*
	{ "_id" : 1, "name" : "apple", "length" : 5 }
	{ "_id" : 2, "name" : "banana", "length" : 6 }
	{ "_id" : 3, "name" : "éclair", "length" : 7 }
	{ "_id" : 4, "name" : "hamburger", "length" : 9 }
	{ "_id" : 5, "name" : "jalapeño", "length" : 9 }
	{ "_id" : 6, "name" : "pizza", "length" : 5 }
	{ "_id" : 7, "name" : "tacos", "length" : 5 }
	{ "_id" : 8, "name" : "寿司", "length" : 6 }
*/
```

# $strLenCP

```java
{ "_id" : 1, "name" : "apple" }
{ "_id" : 2, "name" : "banana" }
{ "_id" : 3, "name" : "éclair" }
{ "_id" : 4, "name" : "hamburger" }
{ "_id" : 5, "name" : "jalapeño" }
{ "_id" : 6, "name" : "pizza" }
{ "_id" : 7, "name" : "tacos" }
{ "_id" : 8, "name" : "寿司" }
```

```java
/*
  db.food.aggregate(
  [
    {
    $project: {
      "name": 1,
      "length": { $strLenCP: "$name" }
    }
    }
  ]
  )
*/
@Test
public void strLenCP() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("name")
      .and(StringOperators.StrLenCP.stringLengthOfCP("name")).as("length")
  );

  mongoTemplate.aggregate(aggregation, "food", Food.class);
}

/*
	{ "_id" : 1, "name" : "apple", "length" : 5 }
	{ "_id" : 2, "name" : "banana", "length" : 6 }
	{ "_id" : 3, "name" : "éclair", "length" : 6 }
	{ "_id" : 4, "name" : "hamburger", "length" : 9 }
	{ "_id" : 5, "name" : "jalapeño", "length" : 8 }
	{ "_id" : 6, "name" : "pizza", "length" : 5 }
	{ "_id" : 7, "name" : "tacos", "length" : 5 }
	{ "_id" : 8, "name" : "寿司", "length" : 2 }
*/
```

# $substrCP

## Value

```java
{ "_id" : 1, "name" : "apple" }
{ "_id" : 2, "name" : "banana" }
{ "_id" : 3, "name" : "éclair" }
{ "_id" : 4, "name" : "hamburger" }
{ "_id" : 5, "name" : "jalapeño" }
{ "_id" : 6, "name" : "pizza" }
{ "_id" : 7, "name" : "tacos" }
{ "_id" : 8, "name" : "寿司sushi" }
```

```java
/*
  db.food.aggregate(
    [
      {
        $project: {
            "name": 1,
            "menuCode": { $substrCP: [ "$name", 0, 3 ] }
        }
      }
    ]
  )
*/
@Test
public void substrCPWithValue() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("name")
      .and(StringOperators.valueOf("name").substringCP(0, 3)).as("menuCode")
  );

  mongoTemplate.aggregate(aggregation, "food", Food.class);
}

/*
	{ "_id" : 1, "name" : "apple", "menuCode" : "app" }
	{ "_id" : 2, "name" : "banana", "menuCode" : "ban" }
	{ "_id" : 3, "name" : "éclair", "menuCode" : "écl" }
	{ "_id" : 4, "name" : "hamburger", "menuCode" : "ham" }
	{ "_id" : 5, "name" : "jalapeño", "menuCode" : "jal" }
	{ "_id" : 6, "name" : "pizza", "menuCode" : "piz" }
	{ "_id" : 7, "name" : "tacos", "menuCode" : "tac" }
	{ "_id" : 8, "name" : "寿司sushi", "menuCode" : "寿司s" }
*/
```

## Referenece

> 최신버전(`spring-data-mongodb:3.3.0-RC1`)에서 `StringOperator.SubstrCP` 클래스는 value(int) 외에 지원하지 않음
>

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
        $project: {
          item: 1,
          yearSubstring: { $substrCP: [ "$quarter", 0, 2 ] },
          quarterSubtring: {
            $substrCP: [
              "$quarter", 2, { $subtract: [ { $strLenCP: "$quarter" }, 2 ] }
            ]
          }
        }
      }
    ]
  )
*/
@Test
public void substrCPWithReference() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .and(StringOperators.valueOf("quarter").substringCP(0, 2)).as("yearSubstring")
      .andExpression("substrCP(quarter, 2, subtract(strLenCP(quarter),2))").as("quarterSubtring")
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "item" : "ABC1", "yearSubstring" : "13", "quarterSubtring" : "Q1" }
	{ "_id" : 2, "item" : "ABC2", "yearSubstring" : "13", "quarterSubtring" : "Q4" }
	{ "_id" : 3, "item" : "XYZ1", "yearSubstring" : "14", "quarterSubtring" : "Q2" }
*/
```

# $trim

```java
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : " product 1" }
{ "_id" : 2, "item" : "ABC2", quarter: "13Q4", "description" : "product 2 \n The product is in stock.  \n\n  " }
{ "_id" : 3, "item" : "XYZ1", quarter: "14Q2", "description" : null }
```

```java
/*
  db.inventory.aggregate(
    [
      { 
        $project: { 
          item: 1, 
          description: { $trim: { input: "$description" } } 
        }
      }
    ]
  )
*/
@Test
public void trim() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .and(StringOperators.valueOf("description").trim()).as("description")
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "item" : "ABC1", "description" : "product 1" }
	{ "_id" : 3, "item" : "XYZ1", "description" : null }
	{ "_id" : 2, "item" : "ABC2", "description" : "product 2 \n The product is in stock." }
*/
```

# $ltrim

```java
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : " product 1" }
{ "_id" : 2, "item" : "ABC2", quarter: "13Q4", "description" : "product 2 \n The product is in stock.  \n\n  " }
{ "_id" : 3, "item" : "XYZ1", quarter: "14Q2", "description" : null }
```

```java
/*
  db.inventory.aggregate(
    [
      { 
        $project: { 
          item: 1, 
          description: { $ltrim: { input: "$description" } } 
        }
      }
    ]
  )
*/
@Test
public void ltrim() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .and(StringOperators.valueOf("description").ltrim()).as("description")
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "item" : "ABC1", "description" : "product 1" }
	{ "_id" : 2, "item" : "ABC2", "description" : "product 2 \n The product is in stock.  \n\n  " }
	{ "_id" : 3, "item" : "XYZ1", "description" : null }
*/
```

# $rtrim

```java
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : " product 1" }
{ "_id" : 2, "item" : "ABC2", quarter: "13Q4", "description" : "product 2 \n The product is in stock.  \n\n  " }
{ "_id" : 3, "item" : "XYZ1", quarter: "14Q2", "description" : null }
```

```java
/*
  db.inventory.aggregate(
    [
      { 
        $project: { 
          item: 1, 
          description: { $rtrim: { input: "$description" } } 
        }
      }
    ]
  )
*/
@Test
public void rtrim() {
  Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.project()
      .andInclude("item")
      .and(StringOperators.valueOf("description").rtrim()).as("description")
  );

  mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/*
	{ "_id" : 1, "item" : "ABC1", "description" : " product 1" }
	{ "_id" : 2, "item" : "ABC2", "description" : "product 2 \n The product is in stock." }
	{ "_id" : 3, "item" : "XYZ1", "description" : null }
*/
```