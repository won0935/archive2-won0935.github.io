---
title: "suspending function"
date: '2023-09-20'
categories: [ Kotlin, Coroutine ]
---

# suspend fun 이란

- 코루틴이 **중지** 되었다가 **재개** 될 수 있는 지점
- 다른 **suspend를 붙은 함수**를 호출할 수 있다.

```kotlin
public fun CoroutineScope.launch(
  context: CoroutineContext = EmptyCoroutineContext,
  start: CoroutineStart = CoroutineStart.DEFAULT,
  block: suspend CoroutineScope.() -> Unit //suspend가 붙은 함수를 호출할 수 있다.
): Job
```

# suspend fun 함수의 활용

여러 비동기 라이브러리를 사용할 수 있도록 도와준다.

```kotlin
fun main(): Unit = runBlocking {
  val result1 = apiCall1()
  val result2 = apiCall2(result1)
  printWithThread(result2)
}
suspend fun apiCall1(): Int {
  return CoroutineScope(Dispatchers.Default).async {
    Thread.sleep(1_000L)
    100
  }.await()
}
suspend fun apiCall2(num: Int): Int {
  return CompletableFuture.supplyAsync { //자바의 CompletableFuture를 사용할 수 있다.
    Thread.sleep(1_000L)
    100
  }.await()
}
```

# 추가적인 suspend 함수들

## coroutineScope

추가적인 코루틴을 만들고, 주어진 함수 블록이 바로 실행된다.
만들어진 코루틴이 모두 완료되면 다음 코드로 넘어간다.

```kotlin
fun main(): Unit = runBlocking {
  printWithThread("START")
  printWithThread(calculateResult())
  printWithThread("END")
}
suspend fun calculateResult(): Int = coroutineScope {
  val num1 = async {
    delay(1_000L)
    10
  }
  val num2 = async {
    delay(1_000L)
    20
  }
  num1.await() + num2.await()
}
// START
// 30
// END
```

## withContext

coroutineScope과 기본적으로 유사하다.
context에 변화를 주는 기능이 추가적으로 있다.

```kotlin
fun main(): Unit = runBlocking {
  printWithThread("START")
  printWithThread(calculateResult())
  printWithThread("END")
}
suspend fun calculateResult(): Int = withContext(Dispatchers.Default) {
  val num1 = async {
    delay(1_000L)
    10
  }
  val num2 = async {
    delay(1_000L)
    20
  }
  num1.await() + num2.await()
}
// START
// 30
// END
```

## withTimeout & withTimeoutOrNull

coroutineScope과 기본적으로 유사하다.
주어진 시간 안에 새로 생긴 코루틴이 완료되어야 한다.

```kotlin
fun main(): Unit = runBlocking {
  withTimeout(1_000L) {
    delay(1_500L)
    10 + 20
  }
}
// Exception in thread "main" kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 1000 ms
```
