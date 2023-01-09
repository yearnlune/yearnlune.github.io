---
tags:
- spring-boot
- spring-graphql
- InvalidPropertyException
- DataBinder
- kotlin
title: spring-graphql에서 InvalidPropertyException 문제
date: 2023/01/09
author: 김동환
description: spring-graphql의 InvalidPropertyException 원인 및 해결
disabled: false
categories:
- spring
---

# 환경

- JDK 11
- spring-boot 2.7.0
- spring-graphql:1.0.0

# 문제 발생

자산 관리 솔루션에서 자산 관련 API를 제공해주는 서비스에서 아래와 같은 실물 자산의 목록을 가져오는 API를 호출하는 과정에서 다음과 같은 문제가 발생하였다.

```kotlin
@QueryMapping("tangibleAssetsGet")
fun findTangibleAssets(@Argument page: PageInput)
```

```graphql
# mongodb-search.graphqls
scalar Long

input PageInput {
    pageNumber: Long!
    pageSize: Long!
    sort: [SortInput]! = []
    searches: [SearchInput]! = []
}

input SearchInput {
    by: String!
    type: PropertyType!
    operator: SearchOperatorType!
    value: [String!]!
}
```

```
org.springframework.beans.InvalidPropertyException: Invalid property 'searches[0].value[256]' of bean class [io.github.yearnlune.search.graphql.PageInput]: Invalid list index in property path 'value[256]'; nested exception is java.lang.IndexOutOfBoundsException: Index 256 out of bounds for length 256
```

위의 오류에서 확인할 수 있는 것은 `IndexOutOfBoundsException`로 할당된 컬렉션 크기가 `256`이지만, 이를 초과하여 문제가 발생하였다. 실제로, `SearchInput.value` 값들이 1,000여 개 정도로 확인되었다. 하지만 `SearchInput.value`의 최대 크기를 256으로 Validation을 설정한 적이 없어 어떠한 문제가 있는지 확인하기로 하였다.

# 문제 원인

원인의 발생은 spring-graphql에서 `@Argument`를 바인딩해주는 `GraphQlArgumentBinder`에서 시작되었다. 객체로 바인딩하기 위해 사용된 `DataBinder`에서 기본 컬렉션의 크기인 `DEFAULT_AUTO_GROW_COLLECTION_LIMIT`의 값 **256**을 그대로 사용하였기 때문에 생긴 문제였다.

```java
// GraphQlArgumentBinder
public Object bind(
  DataFetchingEnvironment environment, @Nullable String argumentName, ResolvableType targetType
) throws BindException {

    Object rawValue = (argumentName != null ?
        environment.getArgument(argumentName) : environment.getArguments());

    if (rawValue == null) {
        return wrapAsOptionalIfNecessary(null, targetType);
    }

    Class<?> targetClass = targetType.resolve();
    Assert.notNull(targetClass, "Could not determine target type from " + targetType);

    DataBinder binder = new DataBinder(null, argumentName != null ? argumentName : "arguments");
    BindingResult bindingResult = binder.getBindingResult();
    Stack<String> segments = new Stack<>();

    try {
        // From Map
        if (rawValue instanceof Map) {
            Object target = createValue((Map<String, Object>) rawValue, targetClass, bindingResult, segments);
            return wrapAsOptionalIfNecessary(target, targetType);
        }
    }
    finally {
        checkBindingResult(bindingResult);
    }
  }
```

```java
public class DataBinder implements PropertyEditorRegistry, TypeConverter {

    public static final int DEFAULT_AUTO_GROW_COLLECTION_LIMIT = 256;
}
```

# 문제 해결

## spring-graphql : ~1.0.1

`spring-graphql:1.0.1` 버전부터 기본 컬렉션 크기를 `1024`로 적용하였으며, 해당 크기를 변경할 수 있게 변경되었다. 해당 버전이 적용된 `spring-boot-starter-graphql:2.7.2` 이후 버전을 적용하여 **최대 크기를 변경 및 적용**할 수 있다.

```java
// ~1.0.1
public class GraphQlArgumentBinder {
  
    private static final int DEFAULT_AUTO_GROW_COLLECTION_LIMIT = 1024;
}
```

```kotlin
@Configuration
class GraphqlConfig {

    @Bean
    fun annotatedControllerConfigurer(): AnnotatedControllerConfigurer {
        return AnnotatedControllerConfigurer()
            .apply { setDataBinderInitializer { it.autoGrowCollectionLimit = 4096 } }
    }
}
```

## spring-graphql : ^1.1.0

`spring-graphql:1.1.0` 버전부터 spring-context의 `DataBinder`를 사용하지 않고, **BeanUtils**를 활용하도록 변경되었다. 그리하여, 해당 버전이 적용된 `spring-boot-starter-graphql:3.0.0`부터는 위와 같은 문제가 발생하지 않는다. 하지만 **Spring Boot** **3.0.0 버전은 JDK 17부터 지원**하기 때문에 잘 고려하여 적용해야 한다.

```java
@Nullable
private Object bindMapToObjectViaConstructor(
  Map<String, Object> rawMap, Constructor<?> constructor, ResolvableType ownerType, 
  ArgumentsBindingResult bindingResult) {

    String[] paramNames = BeanUtils.getParameterNames(constructor);
    Class<?>[] paramTypes = constructor.getParameterTypes();
    Object[] constructorArguments = new Object[paramTypes.length];

    for (int i = 0; i < paramNames.length; i++) {
        String name = paramNames[i];

        ResolvableType targetType = ResolvableType.forType(
            ResolvableType.forConstructorParameter(constructor, i).getType(), ownerType);

        constructorArguments[i] = bindRawValue(
            name, rawMap.get(name), !rawMap.containsKey(name), targetType, paramTypes[i], bindingResult);
    }

    try {
        return BeanUtils.instantiateClass(constructor, constructorArguments);
    }
    catch (BeanInstantiationException ex) {
        // Ignore, if we had binding errors to begin with
        if (bindingResult.hasErrors()) {
            return null;
        }
        throw ex;
    }
}
```