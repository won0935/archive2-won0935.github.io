---
title: "Structured Concurrency"
date: '2023-09-19'
categories: [ Kotlin, Coroutine ]
---

# 코루틴의 Life cycle

![image](https://github.com/won0935/won0935.github.io/assets/55419159/bb564f3b-4d28-4109-b947-2e369e23d11f)

## Completing 단계가 필요한 이유

- 자식 코루틴이 있을 경우 응답을 기다려야 하기 때문
- 부모 자식 코루틴에서 처리 중 에러 발생 시 취소 요청을 전파해야하기 떄문

```kotlin
fun main(): Unit = runBlocking {
  launch {
    delay(600L)
    printWithThread("A") //출력되지 않음. 에러가 전파되었기 때문
  }
  launch {
    delay(500L)
    throw IllegalArgumentException("코루틴 실패 !")
  }
}

// Exception in thread "main" java.lang.IllegalArgumentException: 코루틴 실패 !
```

## Structured Concurrency
- 부모 자식 관계의 코루틴이 한 몸 처럼 움직이는 것

### Structured Concurrency의 장점
- 수많은 코루틴이 유실되거나 누수되지 않도록 보장한다.
- 코드 내의 에러가 유실되지 않고 적절히 보고될 수 있도록 보장한다.
