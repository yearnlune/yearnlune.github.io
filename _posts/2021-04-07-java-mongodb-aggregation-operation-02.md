---
tags: 
    - java
    - mongodb
    - mongo
    - spring data
    - aggregation
title: Spring Data MongoDB - Aggregation Operation - 02
date: 2021/04/07
author: 김동환
description: Aggregation Operation - 02 ($unwind, $sample, $group, $lookup, $count)
disabled: false
categories:
  - java
---
　
# UnwindOperation

> Document내 array field의 각각의 요소들을 풀어 Document로 변경해주는 MongoDB의 `$unwind`를 지원해준다.

```java
{ "_id" : 1, "item" : "ABC1", sizes: [ "S", "M", "L"] }
```

```java
/*
	db.inventory.aggregate(
		[ { $unwind : "$sizes" } ]
	)
*/
@Test
public void size를_Document로_풀기() {
	UnwindOperation unwindOperation = Aggregation.unwind("sizes");

	Aggregation aggregation = Aggregation.newAggregation(
		unwindOperation
	);

	mongoTemplate.aggregate(aggregation, "inventory", Inventory.class);
}

/* Results
{ "_id" : 1, "item" : "ABC1", "sizes" : "S" }
{ "_id" : 1, "item" : "ABC1", "sizes" : "M" }
{ "_id" : 1, "item" : "ABC1", "sizes" : "L" }
*/
```

　
# SampleOperation

> 샘플링을 지원해주는 MongoDB의 `$sample`을 지원해준다.

```java
{ "_id" : 1, "name" : "dave123", "q1" : true, "q2" : true }
{ "_id" : 2, "name" : "dave2", "q1" : false, "q2" : false  }
{ "_id" : 3, "name" : "ahn", "q1" : true, "q2" : true  }
{ "_id" : 4, "name" : "li", "q1" : true, "q2" : false  }
{ "_id" : 5, "name" : "annT", "q1" : false, "q2" : true  }
{ "_id" : 6, "name" : "li", "q1" : true, "q2" : true  }
{ "_id" : 7, "name" : "ty", "q1" : false, "q2" : true  }
```



```java
/*
	db.users.aggregate(
   		[ { $sample: { size: 3 } } ]
	)
*/
@Test
public void 최대_3개로_샘플링() {
	SampleOperation sampleOperation = Aggregation.sample(3L);

	Aggregation aggregation = Aggregation.newAggregation(
		sampleOperation
	);

	mongoTemplate.aggregate(aggregation, "users", User.class);
}

/* Results (Random하게 sampling하여 Results가 다를 수 있음)
{ "_id" : 2, "name" : "dave2", "q1" : false, "q2" : false  }
{ "_id" : 4, "name" : "li", "q1" : true, "q2" : false  }
{ "_id" : 7, "name" : "ty", "q1" : false, "q2" : true  }
*/
```

　
# GroupOperation

> 특정 field를 기준으로 그룹화하는 MongoDB의 `$group`을 지원해준다.

```java
{ "_id" : 1, "item" : "abc", "price" : NumberDecimal("10"), "quantity" : NumberInt("2"), "date" : ISODate("2014-03-01T08:00:00Z") }
{ "_id" : 2, "item" : "jkl", "price" : NumberDecimal("20"), "quantity" : NumberInt("1"), "date" : ISODate("2014-03-01T09:00:00Z") }
{ "_id" : 3, "item" : "xyz", "price" : NumberDecimal("5"), "quantity" : NumberInt( "10"), "date" : ISODate("2014-03-15T09:00:00Z") }
{ "_id" : 4, "item" : "xyz", "price" : NumberDecimal("5"), "quantity" :  NumberInt("20") , "date" : ISODate("2014-04-04T11:21:39.736Z") }
{ "_id" : 5, "item" : "abc", "price" : NumberDecimal("10"), "quantity" : NumberInt("10") , "date" : ISODate("2014-04-04T21:23:13.331Z") }
{ "_id" : 6, "item" : "def", "price" : NumberDecimal("7.5"), "quantity": NumberInt("5" ) , "date" : ISODate("2015-06-04T05:08:13Z") }
{ "_id" : 7, "item" : "def", "price" : NumberDecimal("7.5"), "quantity": NumberInt("10") , "date" : ISODate("2015-09-10T08:43:00Z") }
{ "_id" : 8, "item" : "abc", "price" : NumberDecimal("10"), "quantity" : NumberInt("5" ) , "date" : ISODate("2016-02-06T20:20:13Z") }
```

```java
/*
	db.sales.aggregate( [
		{ $group : { _id : "$item", totalSum: { $sum: "$price" } } }
	] )
*/
@Test
public void item으로_그룹화한_가격_총합() {
	GroupOperation groupOperation = Aggregation.group("item")
		.sum("price").as("totalSum");

	Aggregation aggregation = Aggregation.newAggregation(
		groupOperation
	);

	mongoTemplate.aggregate(aggregation, "sales", Sale.class);
}

/*
{ "_id" : "abc", "totalSum" : 30 }
{ "_id" : "jkl", "totalSum" : 20 }
{ "_id" : "def", "totalSum" : 15 }
{ "_id" : "xyz", "totalSum" : 10 }
*/
```

　
# LookupOperation

> Left Outer Join과 같이 콜렉션과 Join하는 MongoDB의 `$lookup`을 지원해준다.

```java
// `orders` collection
{ "_id" : 1, "item" : "almonds", "price" : 12, "quantity" : 2 },
{ "_id" : 2, "item" : "pecans", "price" : 20, "quantity" : 1 },
{ "_id" : 3  }

// `inventory` collection
{ "_id" : 1, "sku" : "almonds", "description": "product 1", "instock" : 120 },
{ "_id" : 2, "sku" : "bread", "description": "product 2", "instock" : 80 },
{ "_id" : 3, "sku" : "cashews", "description": "product 3", "instock" : 60 },
{ "_id" : 4, "sku" : "pecans", "description": "product 4", "instock" : 70 },
{ "_id" : 5, "sku": null, "description": "Incomplete" },
{ "_id" : 6 }
```

```java
/*
    db.orders.aggregate([
      {
         $lookup:
           {
             from: "inventory",
             localField: "item",
             foreignField: "sku",
             as: "inventory_docs"
           }
      }
    ])
*/
@Test
public void 각_주문의_재고정보() {
	LookupOperation lookupOperation = Aggregation.lookup("inventory", "item", "sku", "inventory_docs");

	Aggregation aggregation = Aggregation.newAggregation(
		lookupOperation
	);

	mongoTemplate.aggregate(aggregation, "orders", Order.class);
}

/*
{
   "_id" : 1,
   "item" : "almonds",
   "price" : 12,
   "quantity" : 2,
   "inventory_docs" : [
      { "_id" : 1, "sku" : "almonds", "description" : "product 1", "instock" : 120 }
   ]
}
{
   "_id" : 2,
   "item" : "pecans",
   "price" : 20,
   "quantity" : 1,
   "inventory_docs" : [
      { "_id" : 4, "sku" : "pecans", "description" : "product 4", "instock" : 70 }
   ]
}
{
   "_id" : 3,
   "inventory_docs" : [
      { "_id" : 5, "sku" : null, "description" : "Incomplete" },
      { "_id" : 6 }
   ]
}
*/
```

　
# CountOperation

> Document들의 개수를 나타내주는 MongoDB의 `$count`를 지원해준다.

```java
{ "_id" : 1, "subject" : "History", "score" : 88 }
{ "_id" : 2, "subject" : "History", "score" : 92 }
{ "_id" : 3, "subject" : "History", "score" : 97 }
{ "_id" : 4, "subject" : "History", "score" : 71 }
{ "_id" : 5, "subject" : "History", "score" : 79 }
{ "_id" : 6, "subject" : "History", "score" : 83 }
```

```java
/*
  db.scores.aggregate(
    [
      {
        $match: {
          score: {
            $gt: 80
          }
        }
      },
      {
        $count: "passing_scores"
      }
    ]
  )
*/
@Test
public void 점수가_80초과인_총_개수() {
	MatchOperation matchOperation = Aggregation.match(Criteria.where("score").gt(80));

	CountOperation countOperation = Aggregation.count().as("passing_scores");

	Aggregation aggregation = Aggregation.newAggregation(
		matchOperation,
		countOperation
	);

	mongoTemplate.aggregate(aggregation, "scores", Score.class);
}
```
