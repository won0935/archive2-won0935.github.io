---
title: "[3장] 설계원칙 - LoD"
date: '2024-01-16'
categories: [ 설계, 디자인 패턴의 아름다움 ]
tags: [ 다자인패턴, LoD ]
image:
  path: https://github.com/won0935/won0935.github.io/assets/55419159/5b3c1887-0957-4817-965c-d4fa6e7330c6
  alt: 디자인 패턴의 아름다움
---

# LoD (Law of Demeter)

- 최소 지식의 원칙(The Least Knowledge Principle) 이라고도 함
- 모든 유닛이 최소한의 지식만을 갖도록 설계해야 한다는 원칙
- LoD를 통해 높은 응집도와 낮은 결합도를 유지할 수 있도록 유지할 수 있음

## LoD를 위반하는 코드

웹 페이지를 크롤링하는 코드를 작성한다고 가정해보자.

- `NetworkTransporter` 클래스는 `HTTP` 요청을 보내는 역할을 한다.
- `HtmlDownloader` 클래스는 `HTTP` 요청을 보내고 응답을 받아서 `HTML`을 다운로드하는 역할을 한다.
- `Document` 클래스는 웹페이지 문서를 표시하고, 문서의 내용을 분석하여 추출하는 역할을 한다.

```kotlin
class NetworkTransporter {
  fun send(htmlRequest: HtmlRequest): Array<Byte> {
    // ...
  }
}

class HtmlDownloader(
  private val networkTransporter: NetworkTransporter
) {

  fun downloadHtml(url: String): Html {
    val rawHtml = networkTransporter.send(HtmlRequest(url))
    return Html(rawHtml)
  }
}

class Document(
  private val html: Html,
  private val url: String
) {
  constructor(url: String) {
    this.url = url
    val htmlDownloader = HtmlDownloader()
    this.html = htmlDownloader.downloadHtml(url)
  }
}
```

### NetworkTransporter 의 문제

`NetworkTransporter` 자체는 저수준의 네크워크 통신 클래스이므로 `Html` 외에도 더 일반화된 기능을 수행해야 한다.
따라서 `HttpRequest` 클래스에 직접 의존하지 않아야 한다.
이러한 관점에서 LoD를 위반하고 있다고 볼 수 있다.

#### 수정된 코드

> **계산하는데 지갑을 건내주지 말자**
> - `HtmlRequest` = 지갑
> - `address`, `contents` = 돈
> - `NetworkTransporter` = 계산하는 직원

```kotlin
class NetworkTransporter {
  fun send(address: String, contents: Array<Byte>): Array<Byte> { //의존하지 않고 클래스가 속성을 스스로 가져오도록 함
    // ...
  }
}
```

### HtmlDownloader 의 문제

`NetworkTransporter` 의 수정사항을 바탕으로 `HtmlDownloader` 를 수정해보자.

#### 수정된 코드

```kotlin
class HtmlDownloader(
  private val networkTransporter: NetworkTransporter
) {

  fun downloadHtml(url: String): Html {
    val request = HtmlRequest(url) //지갑
    val rawHtml = networkTransporter.send(request.addeess, request.contents.bytes) //직원에게 돈을 건냄
    return Html(rawHtml)
  }
}
```

### Document 의 문제

- 생성자에서 `htmlDownloader.downloadHtml()` 메서드를 호출하는데 시간이 오래 걸리거나 오류가 발생할 작업이 있는 작업은 피해야 한다. 또한 생성자에 포함된 코드는 테스트가 매우 까다롭다.
- 생성자에서 `HtmlDownloader` 를 직접 생성하는 것은 인터페이스 기반의 설계를 위반하므로 테스트에 어려움이 있다.
- `Document` 클래스는 `HtmlDownloader` 클래스를 의존해서는 안됨에도 불구하고 종속되어있기 때문에 LoD 를 위반한다.

#### 수정된 코드

```kotlin
data class Document(
  private val html: Html,
  private val url: String
)

//Factory 메소드로 Document 객체를 생성
class DocumentFactory(
  private val htmlDownloader: HtmlDownloader
) {

  fun createDocument(url: String): Document {
    val html = htmlDownloader.downloadHtml(url)
    return Document(html, url)
  }
}
```
