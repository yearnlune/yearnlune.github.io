---
tags: 
    - java
    - rest
    - exception
title: REST API Exception 처리
date: 2021/03/18
author: 김동환
description: REST API Exception 처리
disabled: false
categories:
  - java
---
　
# REST API Exception

> 적절한 `HttpStatus`로 응답이 필요하다.

　
## ResponseStatusException

> `ResponseStatusException` 를 활용하여, 적절한 `HttpStatus` 와 message를 함께 Throw 시킨다.

```java
	
@PostMapping("/account")
public ResponseEntity<AccountDTO.CommonResponse> registerAccount(
    @RequestBody AccountDTO.RegisterRequest registerRequest) {
    
    // ...
    
    // registerRequest.id가 없거나, 길이가 1이하 일 경우 BAD_REQUEST
    if (registerRequest.getId() == null || registerRequest.getId().length() < 1) {
        throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "id 값이 없습니다.");
    }
    
    // ...
    
    AccountDTO.CommonResponse account = accountService.saveAccountIfNotExist(registerRequest);
    
    return new ResponseEntity<>(account, HttpStatus.CREATED);
}

/*
ResponseBody
{
  "timestamp": "2020-12-17T08:19:41.919+0000",
  "status": 400,
  "error": "Bad Request",
  "message": "id 값이 없습니다.",
  "path": "/accounts"
}
*/
```

　
## ExceptionHandler

> `@ExceptionHandler(Exception.class)` 를 활용하여, 해당 `@Controller`, `@RestController` 내 에서 발생한 해당 Exception을 감지하여 처리한다.

```java
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class ExceptionResponse {
    private Timestamp timestamp;
    
    private Integer status;
    
    private String error;
    
    private String message;
    
    private String path;
    
    public ExceptionResponse(HttpStatus httpStatus, BindingResult bindingResult, String path) {
        this.timestamp = new Timestamp(System.currentTimeMillis());
        this.status = httpStatus.value();
        this.error = httpStatus.name();
        this.message = createErrorMessage(bindingResult);
        this.path = path;
    }
    
    public ExceptionResponse(HttpStatus httpStatus, String reason, String path) {
        this.timestamp = new Timestamp(System.currentTimeMillis());
        this.status = httpStatus.value();
        this.error = httpStatus.name();
        this.message = reason;
        this.path = path;
    }
    
    private static String createErrorMessage(BindingResult bindingResult) {
        StringBuilder stringBuilder = new StringBuilder();
        boolean isFirst = true;
    
        for (FieldError fieldError : bindingResult.getFieldErrors()) {
            if (!isFirst) {
                stringBuilder.append(", ");
            } else {
                isFirst = false;
            }
    
            stringBuilder.append("[");
            stringBuilder.append(fieldError.getField());
            stringBuilder.append("] ");
            stringBuilder.append(fieldError.getDefaultMessage());
            stringBuilder.append(" ");
        }
    
        return stringBuilder.toString();
    }
}
```

　
```java
@PostMapping("/account")
public ResponseEntity<AccountDTO.CommonResponse> registerAccount(
    @RequestBody @Valid AccountDTO.RegisterRequest registerRequest) {
    AccountDTO.CommonResponse account = accountService.saveAccountIfNotExist(registerRequest);
    return new ResponseEntity<>(account, HttpStatus.CREATED);
}

// registerAccount 메소드에서 AccountDTO.RegisterRequest를 @Vaild하는 과정에서 이슈가 생긴 경우
@ExceptionHandler(MethodArgumentNotValidException.class)
protected ResponseEntity<ExceptionResponse> handleMethodArgumentNotValid(MethodArgumentNotValidException exception,
    HttpServletRequest request) {
    // ExceptionResponse 클래스
    return new ResponseEntity<>(
        new ExceptionResponse(HttpStatus.BAD_REQUEST, exception.getBindingResult(), request.getRequestURI()),
        HttpStatus.BAD_REQUEST);
}

/*
ResponseBody
{
  "timestamp": "2020-12-21T05:09:50.020+0000",
  "status": 400,
  "error": "BAD_REQUEST",
  "message": "[password] 반드시 값이 존재하고 길이 혹은 크기가 0보다 커야 합니다. , [id] 반드시 값이 존재하고 길이 혹은 크기가 0보다 커야 합니다. ",
  "path": "/accounts"
}
*/
```

　
## ControllerAdvice

> `@ControllerAdvice` 와 `@ExceptionHandler`를 활용하여, `@Controller`, `@RestController` 에서 발생한 Exception을 관리 할 수 있다.

```java
@ControllerAdvice
public class ResponseExceptionHandler {

    // @Controller, @RestController에서 MethodArgumentNotValidException 발생 할 경우 처리
    @ExceptionHandler(MethodArgumentNotValidException.class)
    protected ResponseEntity<ExceptionResponse> handleMethodArgumentNotValid(MethodArgumentNotValidException exception,
        HttpServletRequest request) {
        return new ResponseEntity<>(
            new ExceptionResponse(HttpStatus.BAD_REQUEST, exception.getBindingResult(), request.getRequestURI()),
            HttpStatus.BAD_REQUEST);
    }

}

/*
ResponseBody
{
  "timestamp": "2020-12-21T05:09:50.020+0000",
  "status": 400,
  "error": "BAD_REQUEST",
  "message": "[password] 반드시 값이 존재하고 길이 혹은 크기가 0보다 커야 합니다. , [id] 반드시 값이 존재하고 길이 혹은 크기가 0보다 커야 합니다. ",
  "path": "/accounts"
}
*/
```