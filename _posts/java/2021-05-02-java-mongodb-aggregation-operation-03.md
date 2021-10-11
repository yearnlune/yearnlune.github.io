---
tags: 
    - java
    - mongodb
    - mongo
    - spring data
    - aggregation
title: Spring Data MongoDB - Aggregation Operation - 03
date: 2021/05/02
author: 김동환
description: Aggregation Operation - 03 ($bucket, $bucketAuto, $facet, $set, $unset)
disabled: false
categories:
  - java
---
　
# BucketOperation

> Document들을 일정 범위로 특정 필드를 기준으로 그룹화하는 `$bucket`를 지원해준다.

```java
{ "_id" : 1, "last_name" : "Bernard", "first_name" : "Emil", "year_born" : 1868, "year_died" : 1941, "nationality" : "France" },
{ "_id" : 2, "last_name" : "Rippl-Ronai", "first_name" : "Joszef", "year_born" : 1861, "year_died" : 1927, "nationality" : "Hungary" },
{ "_id" : 3, "last_name" : "Ostroumova", "first_name" : "Anna", "year_born" : 1871, "year_died" : 1955, "nationality" : "Russia" },
{ "_id" : 4, "last_name" : "Van Gogh", "first_name" : "Vincent", "year_born" : 1853, "year_died" : 1890, "nationality" : "Holland" },
{ "_id" : 5, "last_name" : "Maurer", "first_name" : "Alfred", "year_born" : 1868, "year_died" : 1932, "nationality" : "USA" },
{ "_id" : 6, "last_name" : "Munch", "first_name" : "Edvard", "year_born" : 1863, "year_died" : 1944, "nationality" : "Norway" },
{ "_id" : 7, "last_name" : "Redon", "first_name" : "Odilon", "year_born" : 1840, "year_died" : 1916, "nationality" : "France" },
{ "_id" : 8, "last_name" : "Diriks", "first_name" : "Edvard", "year_born" : 1855, "year_died" : 1930, "nationality" : "Norway" }
```

```java
/*
    db.artists.aggregate( [
      {
         $bucket: {
           groupBy: "$year_born",
           boundaries: [ 1840, 1850, 1860, 1870, 1880 ],
           default: "Other",
           output: {
             "count": { $sum: 1 },
             "artists" : {
               $push: {
                 $concat: [ "$first_name", " ", "$last_name"]
               }
             }
           }
        }
      }
    ] )
*/
@Test
public void 연대별_아티스트() {
    AggregationExpression concatExpression = StringOperators.Concat
        .valueOf("first_name")
        .concat(" ")
        .concatValueOf("last_name");

    BucketOperation bucketOperation = Aggregation.bucket("year_born")
        .withBoundaries(1840, 1850, 1860, 1870, 1880)
        .withDefaultBucket("Other")
        .andOutputCount().as("count")
        .andOutput(concatExpression).push().as("artists");

    Aggregation aggregation = Aggregation.newAggregation(
        bucketOperation
    );

    mongoTemplate.aggregate(aggregation, "artists", ArtistDTO.Bucket.class);
}

/* Results
{ "_id" : 1840, "count" : 1, "artists" : ["Odilon Redon" ] }
{ "_id" : 1850, "count" : 2, "artists" : ["Vincent Van Gogh", "Edvard Diriks"] }
{ "_id" : 1860, "count" : 4, "artists" : ["Emil Bernard", "Joszef Rippl-Ronai", "Alfred Maurer", "Edvard Munch"] }
{ "_id" : 1870, "count" : 1, "artists" : ["Anna Ostroumova"] }
*/
```

　
# BucketAutoOperation

> 자동으로 Document들을 일정 범위로 특정 필드를 기준으로 그룹화하는 `$bucketAuto`를 지원해준다.

```java
{ _id: 1 }
{ _id: 2 }
...
{ _id: 100 }
```

```java
/*
    db.things.aggregate( [
      {
        $bucketAuto: {
          groupBy: "$_id",
          buckets: 5,
          granularity: <granularity>
        }
      }
    ] )
*/
// 
@Test
public void 일부터_백까지_그룹화() {
	BucketAutoOperation bucketAutoOperation = Aggregation.bucketAuto("_id", 5)
		.withGranularity(BucketAutoOperation.Granularities.R20);

	Aggregation aggregation = Aggregation.newAggregation(
		bucketAutoOperation
	);

	mongoTemplate.aggregate(aggregation, "things", ThingDTO.Bucket.class);
}

/* Results
{ "_id" : { "min" : 0, "max" : 20 }, "count" : 20 }
{ "_id" : { "min" : 20, "max" : 40 }, "count" : 20 }
{ "_id" : { "min" : 40, "max" : 63 }, "count" : 23 }
{ "_id" : { "min" : 63, "max" : 90 }, "count" : 27 }
{ "_id" : { "min" : 90, "max" : 100 }, "count" : 10 }
*/
```

　
# FacetOperation

> 현 스테이지에서 여러 Aggregate를 하는 `$facet`을 지원해준다.

```java
{ "_id" : 1, "title" : "The Pillars of Society", "artist" : "Grosz", "year" : 1926,
  "price" : NumberDecimal("199.99"),
  "tags" : [ "painting", "satire", "Expressionism", "caricature" ] }
{ "_id" : 2, "title" : "Melancholy III", "artist" : "Munch", "year" : 1902,
  "price" : NumberDecimal("280.00"),
  "tags" : [ "woodcut", "Expressionism" ] }
{ "_id" : 3, "title" : "Dancer", "artist" : "Miro", "year" : 1925,
  "price" : NumberDecimal("76.04"),
  "tags" : [ "oil", "Surrealism", "painting" ] }
{ "_id" : 4, "title" : "The Great Wave off Kanagawa", "artist" : "Hokusai",
  "price" : NumberDecimal("167.30"),
  "tags" : [ "woodblock", "ukiyo-e" ] }
{ "_id" : 5, "title" : "The Persistence of Memory", "artist" : "Dali", "year" : 1931,
  "price" : NumberDecimal("483.00"),
  "tags" : [ "Surrealism", "painting", "oil" ] }
{ "_id" : 6, "title" : "Composition VII", "artist" : "Kandinsky", "year" : 1913,
  "price" : NumberDecimal("385.00"),
  "tags" : [ "oil", "painting", "abstract" ] }
{ "_id" : 7, "title" : "The Scream", "artist" : "Munch", "year" : 1893,
  "tags" : [ "Expressionism", "painting", "oil" ] }
{ "_id" : 8, "title" : "Blue Flower", "artist" : "O'Keefe", "year" : 1918,
  "price" : NumberDecimal("118.42"),
  "tags" : [ "abstract", "painting" ] }
```

```java
/*
	db.artwork.aggregate( [
	  {
	    $facet: {
	      "categorizedByTags": [
	        { $unwind: "$tags" },
	        { $sortByCount: "$tags" }
	      ],
	      "categorizedByPrice": [
	        { $match: { price: { $exists: 1 } } },
	        {
	          $bucket: {
	            groupBy: "$price",
	            boundaries: [  0, 150, 200, 300, 400 ],
	            default: "Other",
	            output: {
	              "count": { $sum: 1 },
	              "titles": { $push: "$title" }
	            }
	          }
	        }
	      ],
	      "categorizedByYears(Auto)": [
	        {
	          $bucketAuto: {
	            groupBy: "$year",
	            buckets: 4
	          }
	        }
	      ]
	    }
	  }
	])
*/

@Test
public void 카테고리화() {
	FacetOperation facetOperation = Aggregation.facet()
		.and(Aggregation.unwind("tags"),
			Aggregation.sortByCount("tags")).as("categorizedByTags")
		.and(Aggregation.match(Criteria.where("price").exists(true)),
			Aggregation.bucket("price")
				.withBoundaries(0, 150, 200, 300, 400)
				.withDefaultBucket("Other")
				.andOutputCount()
				.as("count")
				.andOutput("title")
				.push()
				.as("titles")).as("categorizedByPrice")
		.and(Aggregation.bucketAuto("year", 4)).as("categorizedByYears(Auto)");

	Aggregation aggregation = Aggregation.newAggregation(
		facetOperation
	);

	mongoTemplate.aggregate(aggregation, "artwork", ArtworkDTO.Categorized.class);
}

/* Results
{
  "categorizedByYears(Auto)" : [
    // First bucket includes the document without a year, e.g., _id: 4
    { "_id" : { "min" : null, "max" : 1902 }, "count" : 2 },
    { "_id" : { "min" : 1902, "max" : 1918 }, "count" : 2 },
    { "_id" : { "min" : 1918, "max" : 1926 }, "count" : 2 },
    { "_id" : { "min" : 1926, "max" : 1931 }, "count" : 2 }
  ],
  "categorizedByPrice" : [
    {
      "_id" : 0,
      "count" : 2,
      "titles" : [
        "Dancer",
        "Blue Flower"
      ]
    },
    {
      "_id" : 150,
      "count" : 2,
      "titles" : [
        "The Pillars of Society",
        "The Great Wave off Kanagawa"
      ]
    },
    {
      "_id" : 200,
      "count" : 1,
      "titles" : [
        "Melancholy III"
      ]
    },
    {
      "_id" : 300,
      "count" : 1,
      "titles" : [
        "Composition VII"
      ]
    },
    {
      // Includes document price outside of bucket boundaries, e.g., _id: 5
      "_id" : "Other",
      "count" : 1,
      "titles" : [
        "The Persistence of Memory"
      ]
    }
  ],
  "categorizedByTags" : [
    { "_id" : "painting", "count" : 6 },
    { "_id" : "oil", "count" : 4 },
    { "_id" : "Expressionism", "count" : 3 },
    { "_id" : "Surrealism", "count" : 2 },
    { "_id" : "abstract", "count" : 2 },
    { "_id" : "woodblock", "count" : 1 },
    { "_id" : "woodcut", "count" : 1 },
    { "_id" : "ukiyo-e", "count" : 1 },
    { "_id" : "satire", "count" : 1 },
    { "_id" : "caricature", "count" : 1 }
  ]
}
*/
```

　
# SetOperation

> Document에 새로운 필드를 추가하는 `$addFields` 와 동일한 `$set` 을 지원해준다.

```java
{ _id: 1, student: "Maya", homework: [ 10, 5, 10 ], quiz: [ 10, 8 ], extraCredit: 0 },
{ _id: 2, student: "Ryan", homework: [ 5, 6, 5 ], quiz: [ 8, 8 ], extraCredit: 8 }
```

```java
/*
db.scores.aggregate( [
   {
     $set: {
        totalHomework: { $sum: "$homework" },
        totalQuiz: { $sum: "$quiz" }
     }
   },
   {
     $set: {
        totalScore: { $add: [ "$totalHomework", "$totalQuiz", "$extraCredit" ] } }
   }
] )
*/

@Test
public void 종합_점수_구하기() {
	SetOperation setHomeworkAndQuizOperation = SetOperation.builder()
		.set("totalHomework").toValueOf(ArithmeticOperators.valueOf("homework").sum())
		.set("totalQuiz").toValueOf(ArithmeticOperators.valueOf("quiz").sum());

	SetOperation setScoreOperation = SetOperation.builder()
		.set("totalScore")
		.toValueOf(ArithmeticOperators.valueOf("totalHomework").add("totalQuiz").add("extraCredit"));

	Aggregation aggregation = Aggregation.newAggregation(
		setHomeworkAndQuizOperation,
		setScoreOperation
	);

	mongoTemplate.aggregate(aggregation, "scores", ScoreDTO.TotalScore.class);
}

/* Results
{
  "_id" : 1,
  "student" : "Maya",
  "homework" : [ 10, 5, 10 ],
  "quiz" : [ 10, 8 ],
  "extraCredit" : 0,
  "totalHomework" : 25,
  "totalQuiz" : 18,
  "totalScore" : 43
}
{
  "_id" : 2,
  "student" : "Ryan",
  "homework" : [ 5, 6, 5 ],
  "quiz" : [ 8, 8 ],
  "extraCredit" : 8,
  "totalHomework" : 16,
  "totalQuiz" : 16,
  "totalScore" : 40
}
*/
```

　
# UnsetOperation

> Document의 일부 Field를 지우는 `$unset`을 지원해준다.

```java
{ "_id" : 1, title: "Antelope Antics", isbn: "0001122223334", author: { last:"An", first: "Auntie" }, copies: [ { warehouse: "A", qty: 5 }, { warehouse: "B", qty: 15 } ] },
{ "_id" : 2, title: "Bees Babble", isbn: "999999999333", author: { last:"Bumble", first: "Bee" }, copies: [ { warehouse: "A", qty: 2 }, { warehouse: "B", qty: 5 } ] }
```

```java
/*
    db.books.aggregate([ { $unset: "copies" } ])
*/

@Test
public void copies_필드_지우기() {
	UnsetOperation unsetOperation = UnsetOperation.unset("copies");

	Aggregation aggregation = Aggregation.newAggregation(
		unsetOperation
	);

	mongoTemplate.aggregate(aggregation, "books", Book.class);
}

/* Results
{ "_id" : 1, "title" : "Antelope Antics", "isbn" : "0001122223334", "author" : { "last" : "An", "first" : "Auntie" } }
{ "_id" : 2, "title" : "Bees Babble", "isbn" : "999999999333", "author" : { "last" : "Bumble", "first" : "Bee" } }
*/
```