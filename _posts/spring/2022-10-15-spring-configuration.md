---
title: "Spring Configuration"
date: '2022-10-15'
categories: [ Spring, Core ]
---

# @Configuration이란

`@Configuration` 어노테이션 안에는 `@Component` 어노테이션이 붙어있어서 `@Configuration`이 붙어있는 클래스 역시 스프링의 빈으로 등록이 된다.

그럼에도 불구하고 스프링이 `@Configuration`을 따로 만든 이유는 **CGLib으로 프록시 패턴을 적용해 수동으로 등록하는 스프링 빈이 반드시 싱글톤으로 생성됨을 보장하기 위해서**이다.

스프링은 `@Configuration`이 있는 클래스를 객체로 생성할 때 **CGLib 라이브러리를 사용해 프록시 패턴을 적용**한다. 그래서 `@Bean`이 있는 메소드를 여러 번 호출하여도 항상 동일한 객체를
반환하여 싱글톤을 보장한다.

CGLib은 상속을 **사용해서 프록시를 구현**한다.

# @Component vs @Configuration

스프링의 `@ComponentScan`은 서버 실행시 `@Component`, `@Configuration` 어노테이션을 이용하여 사용자가 생성한 클래스들을 지정하여 등록한다.

### `@Component`

- **수동**으로 스프링 컨테이너에 빈을 등록하는 방법
- 개발자가 직접 **제어가 불가능**한 라이브러리를 빈으로 등록할 때 불가피하게 사용
- 유지보수성을 높이기 위해 애플리케이션 **전범위적으로 사용되는 클래스**나 다형성을 활용하여 **여러 구현체를 빈으로 등록 할 때** 사용
- 1개 이상의 `@Bean`을 제공하는 클래스의 경우 반드시 `@Configuration`을 명시해 주어야 싱글톤이 보장됨

### `@Configuration + @Bean`

- **자동**으로 스프링 컨테이너에 빈을 등록하는 방법
- 스프링의 **컴포넌트 스캔** 기능이 `@Component` 어노테이션이 있는 클래스를 자동으로 찾아서 빈으로 등록함
- 대부분의 경우 `@Component`를 이용한 자동 등록 방식을 사용하는 것이 좋음
- `@Component` 하위 어노테이션으로 `@Configuration`, `@Controller`, `@Service`, `@Repository` 등이 있음

![image](https://user-images.githubusercontent.com/55419159/199490319-1d82b2cb-c591-4baf-b4c5-387dad7c1469.png)
