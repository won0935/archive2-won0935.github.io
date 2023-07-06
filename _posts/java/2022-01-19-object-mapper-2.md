---
title: "ObjectMapper MismatchedInputException 발생시 해결방법"
date: '2022-01-19'
categories: [ Java, Util ]
---

## 상황

`String` 값으로 들어온 json String을 `Object`로 변환하려함

```java
public void convertTest(String jsonStr){
  TestReqeust obj=new ObjectMapper().readValue(jsonStr,new TypeReference<TestReqeust>(){
  });
  }
```

## 오류

```shell
Caused by: 
        com.fasterxml.jackson.databind.exc.MismatchedInputException:
        Cannot construct instance of myInstance(although at least one Creator exists): 
        cannot deserialize from Object value (no delegate- or property-based Creator)
```

## 원인

TestRequest에 파라미터가 없는 생성자가 없어서 나는 오류

## 해결방법

**파라미터가 없는 생성자**를 추가한다.

1. Lombok `@NoArgsConstructor`
2. `public TestRequest(){}`

