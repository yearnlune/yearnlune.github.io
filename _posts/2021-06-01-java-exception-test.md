---
tags: 
    - java
    - exception
    - test
    - junit
title: Java Exception Test
date: 2021/06/01
author: 김동환
description: 자바 Exception 테스트
disabled: false
categories:
  - java
---

　
# Exception

> 일련의 과정을 테스트하는 경우에 일부 예외를 던져 처리하는 경우가 있다. 또한, 정상적으로 수행되는 경우 외에도 비정상적으로 수행되어 예외가 발생하는 것을 테스트해야 한다.

　
## 기본 구조

　
### ExpectedException 활용

`org.junit.rules.ExpectedException`를 활용하여 Exception 테스트를 진행한다.

```java
public class ExceptionTest {

    @Rule
    public ExpectedException expectedException = ExpectedException.none();
    
    @Test
    public void readConfigFile_notExistsConfigPath_shouldBeThrowFileNotFoundException() throws Exception {
        /* THEN */
        expectedException.expect(FileNotFoundException.class);
        expectedException.expectMessage("does not exist");
    
        /* GIVEN */
        File config = new File("/abc/rsyslog.conf");
    
        /* WHEN */
        FileUtils.readFileToString(config, StandardCharsets.UTF_8);
    }
}
```

　
`expect(Class<? extends Throwable> type)`

: 예외로 던져진 클래스와 `type`을 비교 검증한다.

`expectMessage(String substring)`

: 예외로 던져진 메시지와 `containsString()`를 통해 비교 검증한다.

`expectMessage(Matcher<String> matcher)`

: 예외로 던져진 메시지와 `is()`, `startsWith()`등을 `Matcher`를 활용하여 비교 검증한다.

　
### @Test.expected 활용

예상되는 Exception 클래스를 선언하여 테스트를 진행한다.

```java
public class ExceptionTest {

    @Test(expected = FileNotFoundException.class)
    public void readConfigFile_notExistsConfigPath_shouldBeThrowFileNotFoundException() throws Exception {
        /* GIVEN */
        File config = new File("/abc/rsyslog.conf");
    
        /* WHEN */
        FileUtils.readFileToString(config, StandardCharsets.UTF_8);
    
        /* THEN */
        // FileNotFoundException
    }
}
```

　
### assertThrows 활용

JUnit5 jupiter에서 `assertThrows` 함수를 지원해준다. 해당 함수를 통해서 Exception을 람다형식와 같이 처리할 수 있다.

```java
public static <T extends Throwable> T assertThrows(Class<T> expectedType, Executable executable) {
    return AssertThrows.assertThrows(expectedType, executable);
}
```

```java
public class ExceptionTest {

    @Test
    public void readConfigFile_notExistsConfigPath_shouldBeThrowFileNotFoundException() throws Exception {
        /* GIVEN */
        File config = new File("/abc/rsyslog.conf");
        
        /* THEN -> EXPECTED EXCEPTION */
        Exception exception = assertThrows(FileNotFoundException.class, () -> {
                
            /* WHEN */
            FileUtils.readFileToString(config, StandardCharsets.UTF_8);
        });
    
        /* THEN -> EXPECTED EXCEPTION MESSAGE */  
        assertThat(exception.getMessage(), containsString("does not exist"));
    }
}
```