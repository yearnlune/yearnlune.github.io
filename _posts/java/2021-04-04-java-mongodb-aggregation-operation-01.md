---
tags: 
    - java
    - mongodb
    - mongo
    - spring data
    - aggregation
title: Spring Data MongoDB - Aggregation Operation - 01
date: 2021/04/04
author: 김동환
description: Aggregation Operation - 01 ($match, $addFields, $project, $sort, $skip, $limit)
disabled: false
categories:
  - java
---
　
> MongoDB의 Aggregation을 지원하는 Java 인터페이스로 `$match` , `$project` , `$sort` 등을 지원해준다.

　
# MatchOperation

> 지정된 조건과 일치하는 Document를 찾아주는 MongoDB에서의 `$match`를 지원해준다.

```java
{ "_id" : ObjectId("512bc95fe835e68f199c8686"), "author" : "dave", "score" : 80, "views" : 100 }
{ "_id" : ObjectId("512bc962e835e68f199c8687"), "author" : "dave", "score" : 85, "views" : 521 }
{ "_id" : ObjectId("55f5a192d4bede9ac365b257"), "author" : "ahn", "score" : 60, "views" : 1000 }
{ "_id" : ObjectId("55f5a192d4bede9ac365b258"), "author" : "li", "score" : 55, "views" : 5000 }
{ "_id" : ObjectId("55f5a1d3d4bede9ac365b259"), "author" : "annT", "score" : 60, "views" : 50 }
{ "_id" : ObjectId("55f5a1d3d4bede9ac365b25a"), "author" : "li", "score" : 94, "views" : 999 }
{ "_id" : ObjectId("55f5a1d3d4bede9ac365b25b"), "author" : "ty", "score" : 95, "views" : 1000 }
```

```java

/*
 	db.articles.aggregate(
    [ { $match : { author : "dave" } } ]
	)
*/
@Test
public void 저자가_dave인_사용자찾기() {
	MatchOperation matchOperation = Aggregation.match(
		Criteria.where("author").is("dave")
	);

	Aggregation aggregation = Aggregation.newAggregation(
		matchOperation
	);

	mongoTemplate.aggregate(aggregation, "articles", Article.class);
}

/* Results
{ "_id" : ObjectId("512bc95fe835e68f199c8686"), "author" : "dave", "score" : 80, "views" : 100 }
{ "_id" : ObjectId("512bc962e835e68f199c8687"), "author" : "dave", "score" : 85, "views" : 521 }
*/
```

　
# AddFieldsOperation

> Document에 새로운 필드를 추가하는 MongoDB에서의 `$addFields`를 지원해준다.

```java
{
  _id: 1,
  student: "Maya",
  homework: [ 10, 5, 10 ],
  quiz: [ 10, 8 ],
  extraCredit: 0
}
```

```java
/*
	db.scores.aggregate( [
	{
		 $addFields: {
		   totalHomework: { $sum: "$homework" }
		 }
  } ] )
*/
@Test
public void 숙제점수_총합_필드추가() {
	AggregationExpression homeworkSumExpression = ArithmeticOperators.valueOf("homework").sum();

	AddFieldsOperation addFieldsOperation = Aggregation.addFields()
		.addField("totalHomework").withValueOf(homeworkSumExpression)
		.build();

	Aggregation aggregation = Aggregation.newAggregation(
		addFieldsOperation
	);

	mongoTemplate.aggregate(aggregation, "scores", Score.class);
}

/* Results
{
  _id: 1,
  student: "Maya",
  homework: [ 10, 5, 10 ],
  quiz: [ 10, 8 ],
  extraCredit: 0,
  totalHomework: 25
}
*/

```

　
# ProjectionOperation

> 원하는 필드를 제외하거나 포함하는 MongoDB에서의 `$project`를 지원해준다.

```java
{
  "_id" : 1,
  title: "abc123",
  isbn: "0001122223334",
  author: { last: "zzz", first: "aaa" },
  copies: 5
}
```

```java
/*
	db.books.aggregate(
	 [ { $project : { title : 1 , author : 1 } } ]
	)
*/
@Test
public void 제목과_저자만_출력하기() {
	ProjectionOperation projectionOperation = Aggregation.project()
		.and("title").as("title")
		.and("author").as("author");

	Aggregation aggregation = Aggregation.newAggregation(
		projectionOperation
	);

	mongoTemplate.aggregate(aggregation, "books", Book.class);
}

/* Results
{
  "_id" : 1,
  title: "abc123",
  author: { last: "zzz", first: "aaa" }
}
*/
```

　
# SortOperation

> 원하는 필드로 정렬을 하는 MongoDB에서의 `$sort`를 지원해준다.

```java
{ "_id" : 1, "name" : "Central Park Cafe", "borough" : "Manhattan"}
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "borough" : "Queens"}
{ "_id" : 3, "name" : "Empire State Pub", "borough" : "Brooklyn"}
{ "_id" : 4, "name" : "Stan's Pizzaria", "borough" : "Manhattan"}
{ "_id" : 5, "name" : "Jane's Deli", "borough" : "Brooklyn"}
```

```java
/*
	db.restaurants.aggregate(
	 [ { $sort : { borough : 1, _id: -1 } } ]
  )
*/
@Test
public void 도시를_기준으로_오름차순_정렬() {
	SortOperation sortOperation = Aggregation.sort(Sort.Direction.ASC, "borough")
			.and(Sort.Direction.DESC, "_id");

	Aggregation aggregation = Aggregation.newAggregation(
		sortOperation
	);

	mongoTemplate.aggregate(aggregation, "restaurants", Restaurant.class);
}

/* Results
{ "_id" : 5, "name" : "Jane's Deli", "borough" : "Brooklyn" }
{ "_id" : 3, "name" : "Empire State Pub", "borough" : "Brooklyn" }
{ "_id" : 4, "name" : "Stan's Pizzaria", "borough" : "Manhattan" }
{ "_id" : 1, "name" : "Central Park Cafe", "borough" : "Manhattan" }
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "borough" : "Queens" }
*/
```

　
# SkipOperation

> 원하는 양 이후의 Document를 가져오는 MongoDB의 `$skip`을 지원해준다.

```java
{ "_id" : 1, "name" : "Central Park Cafe", "borough" : "Manhattan"}
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "borough" : "Queens"}
{ "_id" : 3, "name" : "Empire State Pub", "borough" : "Brooklyn"}
{ "_id" : 4, "name" : "Stan's Pizzaria", "borough" : "Manhattan"}
{ "_id" : 5, "name" : "Jane's Deli", "borough" : "Brooklyn"}
```

```java
/*
	db.article.aggregate([
		{ $skip : 2 }
	]);
*/
@Test
public void 두개_이후의_레스토랑() {
	SkipOperation skipOperation = Aggregation.skip(2L);

	Aggregation aggregation = Aggregation.newAggregation(
		skipOperation
	);

	mongoTemplate.aggregate(aggregation, "restaurants", Restaurant.class);
}

/* Results
{ "_id" : 3, "name" : "Empire State Pub", "borough" : "Brooklyn"}
{ "_id" : 4, "name" : "Stan's Pizzaria", "borough" : "Manhattan"}
{ "_id" : 5, "name" : "Jane's Deli", "borough" : "Brooklyn"}
*/
```

　
# LimitOperation

> 원하는 양만큼 Document를 가져오는 MongoDB의 `$limit`을 지원해준다.

```java
{ "_id" : 1, "name" : "Central Park Cafe", "borough" : "Manhattan"}
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "borough" : "Queens"}
{ "_id" : 3, "name" : "Empire State Pub", "borough" : "Brooklyn"}
{ "_id" : 4, "name" : "Stan's Pizzaria", "borough" : "Manhattan"}
{ "_id" : 5, "name" : "Jane's Deli", "borough" : "Brooklyn"}
```

```java
/*
	db.article.aggregate([
	   { $limit : 3 }
	]);
*/
@Test
public void 최대_세개까지의_레스토랑() {
	LimitOperation limitOperation = Aggregation.limit(3L);

	Aggregation aggregation = Aggregation.newAggregation(
		limitOperation
	);

	mongoTemplate.aggregate(aggregation, "restaurants", Restaurant.class);
}

/* Results
{ "_id" : 1, "name" : "Central Park Cafe", "borough" : "Manhattan"}
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "borough" : "Queens"}
{ "_id" : 3, "name" : "Empire State Pub", "borough" : "Brooklyn"}
*/
```