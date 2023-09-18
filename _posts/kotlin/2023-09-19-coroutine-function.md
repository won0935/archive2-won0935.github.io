---
title: "코루틴 빌더와 Job"
date: '2023-09-19'
categories: [ Kotlin, Coroutine ]
---

# 코루틴 빌더와 Job

## `runBlocking()`

- 새로운 코루틴을 만들고, 루틴 세계와 코루틴 세계를 이어준다.
- runBlocking으로 인해 만들어진 코루틴과 그 안의 모든 코루틴이 완료될 때 까지 스레드를 blocking한다.

```kotlin
fun main() = runBlocking {
}
```

## `launch()`

- 반환값이 **없는** 코드를 실행
- 코루틴을 제어할 수 있는 객체 `Job`을 반환 받는다.

```kotlin
fun main() = runBlocking {
  val job = launch {
    pirntWithThread("World!")
  }
}
```

### `Job` 객체

- `start()` : 시작 신호
- `cancel()` : 취소 신호
- `join()` : 코루틴이 완료될 때까지 대기

#### 동시에 두 코루틴을 활용하기

![image](https://github.com/won0935/won0935.github.io/assets/55419159/3c257fdf-a12b-47f4-901f-7e53763525e5)

#### `join()`을 활용하면

![image](https://github.com/won0935/won0935.github.io/assets/55419159/7bd9600b-c471-4a6d-a774-bc4fcb2e1919)

코루틴 1이 끝날 때까지 대기했다가 코루틴 2가 실행된다.

## `async()`

- 반환값이 **있는** 코드를 실행
- `Deferred<T>` 객체를 반환 받는다.

```kotlin
fun main() = runBlocking {
  val deferred = async {
    3 + 5
  }
}
```

### `Deferred<T>` 객체

- `Job`을 상속 받아 구현함 (`Job`의 모든 기능을 사용할 수 있음)
- `await()` : 코루틴이 완료될 때까지 대기하며, 결과값을 반환

### `async()` 활용 시 이점

- 여러 API를 **동시에 호출**하여 소요시간을 최소화할 수 있다.
- callback을 이용하지 않고 **동기 방식**으로 코드를 작성할 수 있다.

### `async()` 주의 사항

- `CoroutineStart.LAZY` 옵션을 사용하면, `start()` 함수를 호출해야 한다.

#### `start()` 미사용

- `await()` 함수를 호출했을 때 계산 결과를 계속 기다린다.(블로킹)

```kotlin
fun main(): Unit = runBlocking {
  val time = measureTimeMillis {
    val job1 = async(start = CoroutineStart.LAZY) { apiCall1() } //CoroutineStart.LAZY 옵션 사용 시
    val job2 = async(start = CoroutineStart.LAZY) { apiCall2() }
    printWithThread(job1.await() + job2.await())
  }
  printWithThread("소요시간 : $time ms")
}
```

```shell
// [main @coroutine#1] 3
// [main @coroutine#1] 소요시간 : 2018 ms
```

#### `start()` 사용

- `start()` 함수를 호출해주어야 한다.(논블로킹)

```kotlin
fun main(): Unit = runBlocking {
  val time = measureTimeMillis {
    val job1 = async(start = CoroutineStart.LAZY) { apiCall1() } //CoroutineStart.LAZY 옵션 사용 시
    val job2 = async(start = CoroutineStart.LAZY) { apiCall2() }
    job1.start() //논블로킹하게 동작하기 위해 start() 함수를 호출해준다.
    job2.start()
    printWithThread(job1.await() + job2.await())
  }
  printWithThread("소요시간 : $time ms")
}
```

```shell
// [main @coroutine#1] 3
// [main @coroutine#1] 소요시간 : 1018 ms
```
