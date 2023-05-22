---
tags: 
    - mongodb
    - facet
    - offset pagination
title: 오프셋 페이지네이션 $facet으로 쉽고 간편하게
date: 2023/05/22
author: 김동환
description: mongodb $facet
disabled: false
categories:
  - general
---

# Offset Pagination

오프셋 페이지네이션을 통해 서비스를 제공할 때 백엔드 서비스에선 **오프셋, 사이즈, 검색 및 정렬**과 같은 정보를 통해 쿼리를 진행하게 된다.

```tsx
interface PaginationRequest {
  offset: number;
  limit: number;
  sort: { columnKey: string; direction: 'asc' | 'desc' };
  search: { keyword: string; columnKey?: string };
}
```

이후 쿼리를 진행하고 나온 **항목들**과 페이지 계산에 필요한 **총 항목 수**를 제공한다.

```tsx
interface PaginationResponse {
  items: Item[];
  total: number;
}
```

# AS-IS

일반적으로 서버에서 위의 응답을 제공해주기 위해선 두가지의 일을 해줘야 한다. 

* 페이지 메타정보가 적용된 항목들

Request로 제공된 **limit**, **offset**, **search**, **sort** 정보를 토대로 쿼리를 작성하고 이에 대한 산출물을 제공한다.

```jsx
db.getCollection('users')
  .find({ name: { $regex: '김' } })
  .sort({ 'name': 1 })
  .skip(20)
  .limit(10)
```

* 총 항목 수

이와 동시에 Request로 작성된 쿼리를 기반으로 총 항목 수를 제공한다.

```jsx
db.getCollection('users').find({ name: { $regex: '김' } }).count()
```

총 두 개의 쿼리를 실행하고 이 산출물들을 제공한다.

# TO-BE

mongodb의 `$facet` 을 적극 활용하여 위 두 가지 일을 한번에 처리할 수 있다.

## $facet

facet은 하나의 쿼리를 여러 방면으로 집계하여 한번에 제공해주는 MongoDB의 오퍼레이터다.

![MONGODB_FACET](/assets/images/mongodb/facet.png)

$facet을 적용하여 아래와 같이 하나의 쿼리(파이프라인)로 여러 개의 aggregation을 처리할 수 있다.

```jsx
db.getCollection('users').aggregate([
  { $match: { name: { $regex: '김' } } },  // Query
  { $facet: {
      items: [  // Aggregation #1
        { $sort: { 'name': 1 } },
        { $skip: 20 },
        { $limit: 10 },
      ],
      total: [  // Aggregation #2
        { $count: 'count' },
      ]
  } },
  { $project: { total: { $arrayElemAt: ['$total.count', 0] } } }
])
```

