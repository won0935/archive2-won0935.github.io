---
title: "부모자식 & 루트 코루틴과 예외 처리"
date: '2023-09-19'
categories: [ Kotlin, Coroutine ]
---

# 부모 자식 & 루트 코루틴

## 부모 자식 코루틴

아래의 코드를 다이어그램으로 나타내면...

```kotlin
fun main(): Unit = runBlocking {
  val job1 = launch {
    delay(1_000L)
    printWithThread("Job 1")
  }
  val job2 = launch {
    delay(1_000L)
    printWithThread("Job 2")
  }
}
```

![image](https://github.com/won0935/won0935.github.io/assets/55419159/04bbfc2e-3eed-40fb-a0f3-f638b5760ad8)

## 루트 코루틴

`CoroutineScope` 를 사용함으로써 새로운 루트 코루틴을 생성할 수 있다.

```kotlin
fun main(): Unit = runBlocking {
  val job1 = CoroutineScope(Dispatchers.Default).launch {
    delay(1_000L)
    printWithThread("Job 1")
  }
  val job2 = CoroutineScope(Dispatchers.Default).launch {
    delay(1_000L)
    printWithThread("Job 2")
  }
}
```

![image](https://github.com/won0935/won0935.github.io/assets/55419159/16d2589c-5f08-47fe-8675-551b79a06ef4)

# launch, async 시에 부모자식 & 루트 코루틴

## 루트 코루틴일 경우

### launch

`Exception`이 발생하자마자 예외를 출력하고 코루틴이 종료된다.

```kotlin
fun main(): Unit = runBlocking {
  val job = CoroutineScope(Dispatchers.Default).launch {
    throw IllegalArgumentException()
  }
  delay(1_000L)
}
// Exception in thread "DefaultDispatcher-worker-1 @coroutine#2" java.lang.IllegalArgumentException
```

### async

`Exception`이 발생하더라도 예외를 출력하지 않는다.
`await()` 를 호출해야 예외가 발생한다.

```kotlin
fun main(): Unit = runBlocking {
  val job = CoroutineScope(Dispatchers.Default).async {
    throw IllegalArgumentException()
  }
  delay(1_000L)
  job.await() //이 때 예외가 발생한다.
}
// Exception in thread "DefaultDispatcher-worker-1 @coroutine#2" java.lang.IllegalArgumentException
```

## 부모자식 코루틴일 경우

바로 에러가 발생한다.

```kotlin
fun main(): Unit = runBlocking {
  val job = launch { // async도 동일
    throw IllegalArgumentException()
  }
  delay(1_000L)
}

// Exception in thread "main" java.lang.IllegalArgumentException
```

자식 코루틴에서 발생한 예외가 부모 코루틴으로 전파된다는 것을 알 수 있다.
