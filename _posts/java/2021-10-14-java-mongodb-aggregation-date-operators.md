---
tags: 
    - java
    - mongodb
    - mongo
    - spring data
    - aggregation
title: Spring Data MongoDB - DateOperators
date: 2021/10/14
author: 김동환
description: DateOperators ($year, $month, $week, $hour, $minute, $second, $millisecond, $dayOfYear, $dayOfMonth, $dayOfWeek)
disabled: true
categories:
  - java
---

> $year, $month, $week, $hour, $minute, $second, $millisecond, $dayOfYear, $dayOfMonth, $dayOfWeek 등 날짜와 관련된 operator를 사용할 수 있다.
>

# Field

> 필드를 활용하여 다양한 Date Operators를 사용할 수 있다.
>

```java
{
  "_id" : 1,
  "item" : "abc",
  "price" : 10,
  "quantity" : 2,
  "date" : ISODate("2014-01-01T08:15:39.736Z")
}
```

```java
/*
	db.sales.aggregate([
	 {
	   $project:
		 {
		   year: { $year: "$date" },
		   month: { $month: "$date" },
		   day: { $dayOfMonth: "$date" },
		   hour: { $hour: "$date" },
		   minutes: { $minute: "$date" },
		   seconds: { $second: "$date" },
		   milliseconds: { $millisecond: "$date" },
		   dayOfYear: { $dayOfYear: "$date" },
		   dayOfWeek: { $dayOfWeek: "$date" },
		   week: { $week: "$date" }
		 }
	 }
 ])
*/
@Test
public void dateOf() {
	ProjectionOperation projectionOperation = Aggregation.project()
		.and(DateOperators.dateOf("date").year()).as("year")
		.and(DateOperators.dateOf("date").month()).as("month")
		.and(DateOperators.dateOf("date").dayOfMonth()).as("day")
		.and(DateOperators.dateOf("date").hour()).as("hour")
		.and(DateOperators.dateOf("date").minute()).as("minutes")
		.and(DateOperators.dateOf("date").second()).as("seconds")
		.and(DateOperators.dateOf("date").millisecond()).as("milliseconds")
		.and(DateOperators.dateOf("date").dayOfYear()).as("dayOfYear")
		.and(DateOperators.dateOf("date").dayOfWeek()).as("dayOfWeek")
		.and(DateOperators.dateOf("date").week()).as("week");

	Aggregation aggregation	= Aggregation.newAggregation(
		projectionOperation
	);

	mongoTemplate.aggregate(aggregation, "students", Student.class);
}

/*
	{
	  "_id" : 1,
	  "year" : 2014,
	  "month" : 1,
	  "day" : 1,
	  "hour" : 8,
	  "minutes" : 15,
	  "seconds" : 39,
	  "milliseconds" : 736,
	  "dayOfYear" : 1,
	  "dayOfWeek" : 4,
	  "week" : 0
	}
*/
```