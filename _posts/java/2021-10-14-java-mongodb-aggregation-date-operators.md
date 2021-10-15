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
disabled: false
categories:
  - java
---

> $year, $month, $week, $hour, $minute, $second, $millisecond, $dayOfYear, $dayOfMonth, $dayOfWeek 등 날짜와 관련된 operator를 사용할 수 있다.
>

# Value

> 날짜의 값을 `java.util.Date` , `java.util.Calendar` , `java.time.Instant` , `java.time.ZonedDateTime` , `Long` , `Field` , `AggregationExpression` 등으로 받아  `dateValue()` 를 통하여 Date Operators를 활용할 수 있다.
>

```java
{
  "_id" : 1,
  "item" : "abc",
  "price" : 10,
  "quantity" : 2,
  "date" : ISODate("2014-01-02T08:15:39.736Z")
}
```

```java
/*
    db.sales.aggregate(
      [
        {
          "$project": {
            "today": {"$year": {"$date": Date()}}, 
            "targetDate": {"$dayOfMonth": "$date"}, 
            "aDayBeforeOfTargetDate": {"$dayOfMonth": {"$subtract": ["$date", 86400000]}}
          }
        }
      ]
    )
*/
@Test
public void dateValue() {
    ProjectionOperation projectionOperation = Aggregation.project()
        .and(DateOperators.dateValue(new Date()).year()).as("today")
        .and(DateOperators.dateValue(Fields.field("date")).dayOfMonth()).as("targetDate")
        .and(DateOperators.dateValue(
            ArithmeticOperators.Subtract.valueOf("date").subtract(TimeUnit.DAYS.toMillis(1))
        ).dayOfMonth()).as("aDayBeforeOfTargetDate");
    
    Aggregation aggregation = Aggregation.newAggregation(
        projectionOperation
    );
    
    mongoTemplate.aggregate(aggregation, "sales", Sale.class);
}

/*
    {
        "_id" : 1,
        "today" : 2021,
        "targetDate" : 2,
        "aDayBeforeOfTargetDate" : 1
    }
*/
```


# Field

> `dateOf()` 를 통해 필드를 활용하여, 다양한 Date Operators를 사용할 수 있다.
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
    
    Aggregation aggregation = Aggregation.newAggregation(
        projectionOperation
    );
    
    mongoTemplate.aggregate(aggregation, "sales", Sale.class);
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


# Date string

> Date String을 value, field로 받아 Date 타입으로 변환할 수 있다
>

## Value

> `dateFromString()` 를 통해 DateString을 값으로 받아 Date 타입으로 바꿀 수있다. 또한 `withFormat()` 를 활용하여 값으로 사용된 Date Format(`%Y-%m-%dT%H:%M:%S.%LZ`)을 알려줄 수 있다.
>

```java
/*
    db.tests.aggregate(
      [
        {
          "$project": {
            "dateFromString": {
              "$dateFromString": {"dateString": "15-10-2021", "format": "%d-%m-%Y"}
            },
            "dateFromStringWithUTC": {
              "$dateFromString":{"dateString": "15-10-2021", "format": "%d-%m-%Y", "timezone": "+09"}
            },
            "dateFromStringWithOlson": {
              "$dateFromString": {"dateString": "15-10-2021", "format": "%d-%m-%Y", "timezone": "Asia/Seoul"}
            }
          }
        }
      ]
    )
*/
@Test
public void dateFromString() {
    ProjectionOperation projectionOperation = Aggregation.project()
        .and(
            DateOperators.dateFromString("15-10-2021").withFormat("%d-%m-%Y")
        ).as("dateFromString")
        .and(
            DateOperators.dateFromString("15-10-2021").withFormat("%d-%m-%Y").withTimezone(DateOperators.Timezone.valueOf("+09"))
        ).as("dateFromStringWithUTC")
        .and(
            DateOperators.dateFromString("15-10-2021").withFormat("%d-%m-%Y").withTimezone(DateOperators.Timezone.valueOf("Asia/Seoul"))
        ).as("dateFromStringWithOlson");
    
    Aggregation aggregation = Aggregation.newAggregation(
        projectionOperation
    );
    
    mongoTemplate.aggregate(aggregation, "tests", Map.class);
}

/*
    {
        "_id": 1,
        "dateFromString": ISODate("2021-10-15T00:00:00.000Z"),
        "dateFromString": ISODate("2021-10-14T15:00:00.000Z"),
        "dateFromString": ISODate("2021-10-14T15:00:00.000Z")
    }
*/
```

## Field

> `DateFromString.fromStringOf` 를 활용하여 field의 DateString을 활용할 수 있다.
>

```java
{
  "_id" : 1,
  "item" : "abc",
  "price" : 10,
  "quantity" : 2,
  "date" : 2021-10-15 11:00:00
}
```

```java
/*
    db.sales.aggregate(
      [
        {
          "$project": {
            "dateFromString": {
              "$dateFromString": {"dateString" : "$date", "format" : "%Y-%m-%d %H:%M:%S"}
            }
          }
        }
      ]
    )
*/
@Test
public void fromStringOf() {
    ProjectionOperation projectionOperation = Aggregation.project()
    .and(
      DateOperators.DateFromString.fromStringOf("date").withFormat("%Y-%m-%d %H:%M:%S")
    ).as("dateFromString");
    
    Aggregation aggregation = Aggregation.newAggregation(
      projectionOperation
    );
    
    mongoTemplate.aggregate(aggregation, "sales", Map.class);
}

/*
    {
        "_id": 1,
        "dateFromString": ISODate("2021-10-15T11:00:00.000Z"),
    }
*/
```