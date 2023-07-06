---
title: "apply, run, with, let, also 정리"
date: '2022-05-07'
categories: [ Kotlin ]
---

# 범위지정함수(scope function)란?

범위지정함수는 **특정 객체에 대한 작업을 블록 안에 넣어 실행할 수 있도록 하는 함수**이다.
**블록은 특정 객체에 대해 할 작업의 범위가 되며, 따라서 범위지정함수**라 부른다.
특정 객체에 대한 작업을 블록안에 넣게 되면 가독성이 증가하여 유지 보수가 쉬워진다.
코틀린에서는 `apply`, `run`, `with`, `let`, `also` 총 5가지 기본적인 범위지정함수를 지원한다.

---

# 함수별 정리

`apply`, `run`, `with`, `let`, `also` 에 대한 분류를 하면 다음과 같다.

![image](https://user-images.githubusercontent.com/55419159/167235461-ba8c84d1-9886-4f94-9a23-43109b0178ea.png)

아래의 data 클래스를 활용해 예시를 들어본다.

```kotlin
data class Person(
  var name: String = "",
  var age: Int = 0,
  var temperature: Float = 36.5f
)
```

## apply

**`apply`는 수신객체 내부 프로퍼티를 변경한 다음 수신 객체 자체를 반환하기 위해 사용되는 함수**이다.
따라서 객체 생성 시에 다양한 프로퍼티를 설정해야 하는 경우 자주 사용된다.

`apply`에서의 블록은 람다식의 수신 객체로 `apply`의 수신객체(`T`)를 지정하기 때문에 람다식 내부에서 수신객체에 대한 명시를 하지 않고 함수를 호출할 수 있게 된다.

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T 
```

`apply`를 활용하면 다음의 방법으로 수신객체의 프로퍼티 지정이 가능하다.
람다식의 수신객체가 `apply`의 수신객체이기 때문에 수신객체에 대한 명시를 생략하는 것이 가능하다.

```kotlin
val person = Person().apply {
  name = "Song"
  age = 30
  temperature = 36.3f
}
```

프로퍼티 설정 시마다 person을 쓰지 않아도 되어 가독성이 좋다.
자바의 builder 타입과 유사하다.

## run

**`run`은 `apply`와 똑같이 동작하지만 수신 객체를 return 하지 않고, `run` 블록의 마지막 라인을 return** 하는 범위지정 함수이다.
이는 **수신객체에 대해 특정한 동작을 수행한 후 결과값을 리턴 받아야 할 경우** 사용한다.

```kotlin
public inline fun <T, R> T.run(block: T.() -> R): R 
```

예를 들어 위 `Person` 객체의 체온을 체크해서 정상인지를 확인하다고 해보자.
만약 사람의 체온이 37.5도 이상이면 비정상으로 다음과 같이 마지막줄을 return 받을 수 있다.

```kotlin
Person(
  name = "",
  age = 0,
  temperature = 36.5f
) {
  fun isSick(): Boolean = temperature > 37.5f
}
```

```kotlin
fun main() {
  val person = Person(name = "Song", age = 30, temperature = 36.3f)
  val isPersonSick = person.run {
    temperature = 37.2f
    isSick() //return 값
  }

  print("PersonIsSick : $isPersonSick")
}
```

![image](https://user-images.githubusercontent.com/55419159/167236105-84dd1876-2d86-44e2-89cc-0ac443a4816d.png)

`run`은 수신객체 없이도 동작할 수 있다.
다만 수신객체 없이 `run`을 사용하면 내부에 수신객체를 명시해줘야 한다.

```kotlin
val person = Person(name = "Song", age = 30, temperature = 36.3f)
val isPersonSick = run {
  person.temperature = 37.2f
  person.isSick()
}
```

## with

**`with`은 수신객체에 대한 작업 후 마지막 라인을 return 한다.
`run`과 완전히 똑같이 동작한다.
다른 점은 `run`은 확장 함수로 사용되지만 `with`은 수신객체를 파라미터로 받아 사용한다는 점이다.**
`run`을 사용하는 것이 깔끔하므로 실제로는 거의 사용하지 않는다.

```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R 
```

위의 예시에 with을 사용하면 다음과 같다.

```kotlin
fun main() {
  val person = Person(name = "Song", age = 30, temperature = 36.3f)
  val isPersonSick = with(person) {
    temperature = 37.2f
    isSick() //return 값
  }

  print("PersonIsSick : $isPersonSick")
}
```

## let

**`let`은 수신객체를 이용해 작업을 한 후 마지막 줄을 return할 때 사용한다.
`run`이나 `with`과는 수신객체를 접근할 때 it을 사용해야 한다는 점만 다르고 나머지 동작은 같다.**
하지만 실제 사용에서는 조금 차이가 있다.

```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R 
```

`let`은 다음과 같은 경우 사용된다.

- **null check 이후 코드를 실행**해야 하는 경우
- **nullable한 수신객체를 다른 타입의 변수로 변환**해야 하는 경우

**요약하면 nullable한 값을 처리해야 할 때는 let를 사용해야 한다.**

`let`을 이용해 null check를 하려면 아래와 같이 null check 연산잔인 `?`와 함께 사용해야 한다.
`?.let`을 사용하게 되면 `let`의 블록은 수신객체가 null이 아닐때만 수행된다.
따라서 `let` 블록에서의 `it` 타입은 nullable 하지 않은 타입이 된다.

예를들어 사람이 null이 아닐 때만 영화를 예매해야 한다고 해보자.

```kotlin
fun main() {
  val person: Person? = null
  val isReserved = person?.let { it: Person ->
    reserveMovie(it)
  }
}
```

person은 nullable한 객체 (Person?)였는데, `?.let`을 사용하면 `let` 블록 내부에서는 더이상 nullable하지 않은 it : Person 이 된다.
즉, null check 후 코드가 실행된 것을 확인할 수 있다.
또한 person을 사용해 영화를 예매하고 결과값을 return받았으므로 Person객체가 다른 타입의 변수로 변환된 것 또한 확인할 수 있다.

_물론 `let`은 nullable하지 않은 대상에 대해서도 사용할 수 있지만, 실무에서는 nullable한 값이 아닐 경우에는 `run`을 사용하는 것이 일반적이다._

## also

`also`는 `apply`와 마찬가지로 수신객체 자신을 반환한다.
`apply`가 프로퍼티를 세팅 후 객체 자체를 반환 하는데만 사용된다면, `also`는 프로퍼티 세팅 뿐만아니라 객체에 대한 추가적인 작업(로깅, 유효성 검증)을 한 후 객체를 반환할 때 사용된다.

`also`에서의 블록은 람다식의 입력 파라미터로 `also`의 수신객체를(`T`)를 지정하기 때문에 내부에서 수신객체를 사용하기 위해서는 `it`을 사용해야 한다.

```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T 
```

`also`가 사용되는 예는 다음과 같다.

number을 반환받는 함수를 만든 후 해당 number의 숫자를 올리고 싶을 때 다음과 같이 number을 return한 다음 number의 값을 올린다.

```kotlin
var number = 3

fun getAndIncreaseNumber() = number.also {
  number++
}

fun main() {
  println("first number ${getAndIncreaseNumber()}")
  println("second number ${getAndIncreaseNumber()}")
}
```

![image](https://user-images.githubusercontent.com/55419159/167237080-8fff8aa1-e7aa-4fc4-baf0-3933ef0aa810.png)

**주의할 점은 객체를 사용할 때는 객체의 주소값을 return하는 것이기 때문에 객체의 프로퍼티가 바뀌면 `also`에서 return하는 객체의 프로퍼티 또한 바뀐다는 점이다**.
따라서 객체의 프로퍼티를 다음과 같이 바꾸어 버릴 경우, 바뀐 프로퍼티가 객체의 값이 되어버린다.
분명 age가 29에서 시작됐는데 30으로 나오는 것을 볼 수 있다.

```kotlin
val person = Person(name = "Song", age = 30, temperature = 36.3f)

fun getAndIncreaseNumber() = person.also {
  person.age = it.age + 1
}

fun main() {
  println("first number ${getAndIncreaseNumber()}")
  println("second number ${getAndIncreaseNumber()}")
}
```

![image](https://user-images.githubusercontent.com/55419159/167237215-9d922e45-ca96-4d75-9785-0742407b8a43.png)

따라서 보통 객체에 대해 같은 용도로 사용하고자 할 때는 `copy`를 사용해야 한다. 그래야 바뀌지 않은 객체가 return됨을 보장할 수 있다.

```kotlin
val person = Person(name = "Song", age = 30, temperature = 36.3f)

fun getAndIncreaseNumber() = person.also {
  person.age = person.copy(age = it.age + 1)
}

fun main() {
  println("first number ${getAndIncreaseNumber()}")
  println("second number ${getAndIncreaseNumber()}")
}
```

![image](https://user-images.githubusercontent.com/55419159/167237241-9f3343ac-d2af-4252-b37e-8bff1f6a0f3f.png)

이러한 문제 때문에 `also`는 거의 사용되지 않고, 사용할 때는 프로퍼티를 바꾸지 않고 동작을 추가적으로 해야하는 경우(로깅, 검증)등에서 사용된다.
