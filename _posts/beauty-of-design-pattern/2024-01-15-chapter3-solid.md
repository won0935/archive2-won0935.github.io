---
title: "[3장] 설계원칙 - SOLID"
date: '2024-01-15'
categories: [ 설계, 디자인 패턴의 아름다움 ]
tags: [ 다자인패턴, SOLID ]
image:
  path: https://github.com/won0935/won0935.github.io/assets/55419159/5b3c1887-0957-4817-965c-d4fa6e7330c6
  alt: 디자인 패턴의 아름다움
---

# SOLID

## SRP (단일 책임 원칙)

- 하나의 클래스나 모듈은 하나의 책임만 가져야 한다.
- 클래스와 모듈은 서로 의미가 동일하다.

### 클래스를 분리해야하는 기준

- 클래스에 코드, 함수 또는 속성이 너무 많아 코드의 가독성과 유지 보수성에 영향을 미치는 경우
- 클래스가 너무 과하게 다른 클래스에 의존하는 경우
- 클래스에 `private` 메소드가 너무 많은 경우
- 클래스의 이름을 명확하게 지정하기 어려운 경우(`Manager`, `Context`.. 뜨끔😅)
- 클래스의 많은 메소드나 항목이 일부 로직에서만 작동하는 경우

### 생각해보기

- 클래스 외에 단일 책임 원칙을 적용할 수 있는 설계에는 어떤 것들이 있을까

## OCP (개방 폐쇠 원칙)

- 확장에는 열려있고, 변경에는 닫혀있어야 한다.
- 가장 어렵지만 유용한 원칙

이해하기 어려운 이유
> - 코드를 변경할 때 확장으로 보아야 하는지 수정으로 보아야 하는지 판단하기 어렵기 떄문

유용한 이유
> - 객체지향에서는 **확장성이 코드 품질의 중요한 척도**이기 때문
> - 디자인 패턴의 대부분은 **확장성 문제**를 해결하기 위함임

### 코드를 수정하는 것은 개방 폐쇠 원칙을 위반하는 것일까?

- 결론만 말하자면 기존 코드를 전혀 수정하지 않는 것은 불가능함
- **단위테스트를 깨트리지 않는 한** 코드를 수정하는 것은 개방 폐쇠 원칙를 위반하는 것이 아님

### 개방 폐쇠 원칙을 달성하는 방법

- 코드를 작성할 때 앞으로 요구사항이 추가될 가능성이 있는지 고려해야 함(다형성)
- 코드의 변경가능한 부분과 변경할 수 없는 부분을 잘 식별해야 함(캡슐화)
- 코드의 **확장성**은 코드 품질을 판단하는 중요한 기준임

### 실무에 적용하기

- 아래와 같은 경우 확장 포인트를 미리 준비
  - 단기간에 진행할 수 있는 확장
  - 코드 구조 변경에 미치는 영향이 큰 확장
  - 비용이 많이 들지않는 확장
- 아래와 같은 경우 필요할 때 리펙토링
  - 후에 지원해야 하는 여부가 확실하지 않은 경우
  - 확장이 코드개발에 부하를 주는 경우

### 생각해보기

- 개방 폐쇠 원칙이 생겨난 이유가 무엇일까

## LSP (리스코프 치환 원칙)

- 자식 클래스는 부모 클래스에서 가능한 행위를 수행할 수 있어야 한다.

### 리스코프 치환 원칙과 다형성의 차이점

- 보기에는 비슷하나 완전히 다른 의미임
- 리스코프 치환 원칙은 상위 클래스를 대체할 때 프로그램의 **원래 논리적 동작이 변경되지 않고 프로그램의 정확성이 손상되지 않도록 해야한다**는 의미임(비즈니스 로직의 통일성이 중요)

### 리스코프 치환 원칙을 위반하는 경우

- 계약에 따른 설계
  - 하위 클래스를 설계할 때는 상위 클래스의 **동작 규칙**을 따라야함
  - 하위 클래스는 내부 구현 논리를 변경할 수 있지만 **동작 규칙**은 변경할 수 없음
  - **동작 규칙**에 포함되는 것들
    - 함수의 입출력
    - 예외에 대한 규칙
    - 주석에 나열된 특수 지침

#### 선언한 기능을 위반하는 경우

- ex> 메소드를 오버라이드해서 완전히 다른 기능을 구현한 경우

#### 입력, 출력 및 예외에 대한 계약을 위반하는 경우

- ex> 상위 클래스의 메소드가 장애 시 `CustomException`을 발생시킨다는 계약을 가지고 있지만 하위 클래스에서는 다른 예외를 발생하는 경우

#### 주석에 나열된 특별 지침을 위반하는 경우

- ex> 상위 클래스에서 선언한 비즈니스로직을 하위 클래스에서 재정의하는 경우
- 이 경우 상위 클래스의 주석을 수정하는 편이 나음

### 리스코프 치환 원칙을 확인하는 꿀팁

- 상위 클래스의 **단위 테스트**로 하위 클래스의 코드를 확인

### 생각해보기

- 리스코프 치환 원칙이 중요한 이유

## ISP (인터페이스 분리 원칙)

> 클라이언트는 **필요하지 않은** 인터페이스에 의존하도록 강요받지 않아야 한다.
>
> _by 로버트 마틴_

### API나 기능의 집합에서의 인터페이스

설계할 때 인터페이스 또는 기능의 일부가 호출자 중 일부에만 사용되거나 전혀 사용되지 않는다면 불필요한 항목을 **강요**하는 대신,
인터페이스나 기능에서 해당 부분을 분리하여 해당 호출자에게 별도로 제공해야 하며, 사용하지 않는 인터페이스나 기능에는 접근하지 못하게 해야함

#### 예제코드

```kotlin
// 일반 사용자는 회원가입, 로그인, 로그아웃만 가능
interface UserService {
  fun register(user: User)
  fun login(user: User)
  fun logout(user: User)
}

// 삭제는 관리자만 가능
interface AdminService {
  fun delete(user: User)
}
```

### 단일 API나 기능에서의 인터페이스

- API나 기능은 가능한 한 단순해야 하며 하나의 기능에 여러 다른 기능 논리를 구현하지 않아야 함
- 인터페이스 분리 원칙은 단일 책임 원칙과 유사함
  - 호출자가 인터페이스 일부만 사용하는 경우 단일 책임 원칙을 위반하는 것

#### 예제코드

```kotlin
data class Statistics(
  val count: Int,
  val sum: Int,
  val average: Int
)

fun count(dataSet: Collection<Long>): Statistics {
  val statistics = Statistics()
  // 로직 생략...
  return statistics
}
```

- 위 코드에서는 `count` 함수가 `sum`과 `average`를 계산하는 로직을 포함하고 있음
- 분할 후 코드는 아래와 같음

```kotlin
fun count(dataSet: Collection<Long>): Int {
  // 로직 생략...
}

fun sum(dataSet: Collection<Long>): Int {
  // 로직 생략...
}

fun average(dataSet: Collection<Long>): Int {
  // 로직 생략...
}
```

### 객체지향 프로그래밍에서의 인터페이스

#### 인터페이스 분리 원칙을 지킨 경우

- 시스템이 `Redis`, `Kafka`, `MySql` 시스템에 연계되어있다고 가정
- `Config` 클래스를 구현해야함

```kotlin
class RedisConfig(
  val configSource: ConfigSource,
  val host: String,
  val port: Int,
  val timeout: Int
) {

  fun getAddress(): String {
    return "$host:$port"
  }

  fun update() {
    // configSource를 통해 Redis의 설정을 업데이트
  }
}

class KafkaConfig(...)

class MySqlConfig(...)
```

##### 요구사항 : 실시간으로 설정정보를 업데이트 해야함 (Redis, Kafka)

```kotlin
interface Updater {
  fun update()
}

class RedisConfig : Updater {
  // 생략...
  override fun update() {
    // configSource를 통해 Redis의 설정을 업데이트
  }
}

class KafkaConfig : Updater {
  // 생략...
  override fun update() {
    // configSource를 통해 Kafka의 설정을 업데이트
  }
}

class MySqlConfig(...)

class ScheduledUpdater(
  val configs: List<Updater>
) {
  @Scheduled(fixedDelay = 60 * 1000)
  fun run() {
    for (config in configs) {
      config.update()
    }
  }
}
```

##### 요구사항 추가 : 모니터링을 해야함 (Redis, MySql)

```kotlin
interface Viewer {
  fun outputInPlainText()
}

class RedisConfig : Updater, Viewer {
  // 생략...
  override fun update() {
    // configSource를 통해 Redis의 설정을 업데이트
  }

  override fun outputInPlainText() {
    // Redis의 설정을 텍스트로 출력
  }
}

class KafkaConfig : Updater {
  // 생략...
  override fun update() {
    // configSource를 통해 Kafka의 설정을 업데이트
  }
}

class MySqlConfig : Viewer {
  // 생략...
  override fun outputInPlainText() {
    // MySql의 설정을 텍스트로 출력
  }
}

class ScheduledUpdater(...)

class ViewerMonitor(
  val host: String,
  val port: Int,
  val viewers: List<Viewer> = mutableListOf()
) {

  fun addViewer(viewer: Viewer) {
    viewers.add(viewer)
  }

  fun run() {
    // 생략...
  }
}

class Application(
  val redisConfig: RedisConfig,
  val kafkaConfig: KafkaConfig,
  val mySqlConfig: MySqlConfig,
) {

  fun main(args: Array<String>) {
    val scheduledUpdater = ScheduledUpdater(listOf(redisConfig, kafkaConfig))
    val viewerMonitor = ViewerMonitor("localhost", 8080, listOf(redisConfig, mySqlConfig))
    viewerMonitor.addViewer(kafkaConfig)
    viewerMonitor.addViewer(mySqlConfig)
    viewerMonitor.run()
  }
}
```

#### 인터페이스 분리 원칙을 지키지 않는 경우

```kotlin
interface Config {
  fun update()
  fun outputInPlainText()
}

class RedisConfig : Config {
  // 생략...
  override fun update() {
    // configSource를 통해 Redis의 설정을 업데이트
  }

  override fun outputInPlainText() {
    // Redis의 설정을 텍스트로 출력
  }
}

class KafkaConfig : Config {
  // 생략...
  override fun update() {
    // configSource를 통해 Kafka의 설정을 업데이트
  }

  override fun outputInPlainText() {
    // Kafka의 설정을 텍스트로 출력
  }
}

class MySqlConfig : Config {
  // 생략...
  override fun update() {
    // configSource를 통해 MySql의 설정을 업데이트
  }

  override fun outputInPlainText() {
    // MySql의 설정을 텍스트로 출력
  }
}
```

#### 두 코드의 비교

- 첫 번째 설계는 더 유연하기 때문에 확장성이 높고 재사용이 쉬움
  - `ScheduledUpdater`는 `Updater` 인터페이스를 구현하는 객체만 받음
  - 요구사항이 추가되었을 때에도 대응이 쉬움
- 두 번째 설계는 코드에서 쓸모없는 작업을 수행함
  - `KafkaConfig` 객체는 `outputInPlainText` 기능이 필요없음
  - 인터페이스의 밀도가 작은 경우 변경에 따라 수정해야하는 클래스도 줄어듬

### 생각해보기

- `AtomicInteger` 클래스의 `getAndIncremnet()` 메소드는 인터페이스 분리 원칙을 위반하는가?

## DIP (의존 역전 원칙)

의존 역전 원칙은 사용하기는 쉬운 반면 이해하기는 어려움

- **의존 역전**이 뜻하는 것은 어떤 대상 사이의 역전인가? 어떤 의존이 역전되는 것인가? 역전은 무엇을 의미하는가?
- **제어 반전**과 **의존성 주입**이라는 두 가지 다른 개념을 접할 수 있는데, 이 개념은 의존 역전과 같은 개념에 속하는가? 아니라면 어떤 차이가 있는가?
- `Spring` 의 IoC는 앞에서 언급한 3가지 개념과 어떤 관계가 있는가?

### 제어 반전

- 아래 코드는 제어 반전의 일반적인 형태임
- 프레임워크는 전체 실행흐름을 관리하기 위한 확장 가능한 **코드 골격**을 제공
- 프로그래머가 프레임워크를 사용할 때는 제공되는 확장 포인트에 비즈니스 코드를 작성하는 것만으로 프로그램을 완성할 수 있음 

#### 예제 코드

```kotlin
abstract class TestCase {
  abstract fun doTest()
}

class TestExecutor(
  val testCases: List<TestCase>
) {
  fun execute() {
    for (testCase in testCases) { // 제어의 흐름을 역전시킴
      testCase.doTest()
    }
  }
}
```

- 제어 : 프로그램의 실행 흐름을 제어하는 것
- 역전 : 프로그램의 실행 흐름을 제어하는 권한을 프레임워크에 넘김

### 의존성 주입

#### 예제 코드

```kotlin
class Notification(
  val messageSender: MessageSender // 의존성 주입
)
```

### 의존성 주입 프레임워크

- 객체 생성 주입은 비즈니스 논리에 속하지 않기 떄문에 프레임워크에 의해 자동으로 완성되는 프레임워크를 **의존성 주입 프레임워크**라고 함
- 의존성 주입 프레임워크는 프레임워크가 자동으로 객체를 생성하고, 객체의 라이프사이클을 관리하고, 의존성 주입을 할 수 있음

### 의존 역전 원칙

- 상위 모듈은 하위 모듈에 의존하지 않아야 하며, 추상화에 의존하여야 함
- 또한 추상화가 세부사항에 의존하는 것이 아니라, 세부사항이 추상화에 의존해야 함

### 생각해보기

- 인터페이스 기반과 의존성 주입은 서로 유사한데, 두 개념의 차이점은 무엇일까? 
