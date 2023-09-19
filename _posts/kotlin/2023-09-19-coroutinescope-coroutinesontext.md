---
title: "CoroutineScope과 CoroutineContext"
date: '2023-09-19'
categories: [ Kotlin, Coroutine ]
---

# CoroutineScope

사실 `launch`, `async`는 `CoroutineScope`의 확장 함수이다.

```kotlin
//launch
public fun CoroutineScope.launch(
  context: CoroutineContext = EmptyCoroutineContext,
  start: CoroutineStart = CoroutineStart.DEFAULT,
  block: suspend CoroutineScope.() -> Unit
): Job

//async
public fun <T> CoroutineScope.async(
  context: CoroutineContext = EmptyCoroutineContext,
  start: CoroutineStart = CoroutineStart.DEFAULT,
  block: suspend CoroutineScope.() -> T
): Deferred<T>
```

직접 `CoroutineScope`을 만들면 `runBlocking`이 필요하지 않다.

## CoroutineScope의 역할

CoroutineScope의 주요 역할은 **`CoroutineContext` 라는 데이터를 보관하는 것**이다.

```kotlin
public interface CoroutineScope {
  public val coroutineContext: CoroutineContext
}
```

### CoroutineContext가 가질 수 있는 것들

- 코루틴 이름
- 코루틴 그 자체
- `CoroutineDispatcher` -> 코루틴이 **어떤 스레드에 배정될지**를 관리하는 역할을 한다.
- `CoroutineExceptionHandler`

# CoroutineContext

- `CoroutineContext`는 Map + Set을 합쳐놓은 형태

```kotlin
// + 기호를 활용한 Element 합성
CoroutineName("재원 코루틴") + SupervisorJob()
// context에 Element를 추가
coroutineContext + CoroutineName("재원 코루틴")
```

- 부모 자식 관계의 코루틴에서 `CoroutineContext`의 데이터를 공유할 수 있다.

![image](https://github.com/won0935/won0935.github.io/assets/55419159/7e98ae0f-ae42-471b-8501-e5853b877925)

## Dispatcher 종류

### `Dispatchers.Default`

가장 기본적인 디스패처, CPU 자원을 많이 쓸 때 권장 별다른 설정이 없으면 이 디스패처가 사용된다.

### `Dispatchers.IO`

I/O에 최적화된 디스패처, 네트워크나 디스크 작업에 적합하다. 기본적으로 최대 64개의 스레드를 사용한다.

### `Dispatchers.Main`

메인 스레드에서 동작하는 디스패처, 안드로이드에서만 사용 가능하다.

### `asCoroutineDispatcher()`

Java 의 쓰레드 풀인 ExecutorService를 Dispatcher로 변환할 수 있다.
