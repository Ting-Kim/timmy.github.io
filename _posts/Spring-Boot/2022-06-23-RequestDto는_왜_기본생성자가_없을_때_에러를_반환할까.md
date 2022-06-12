---
layout: post
title: "RequestDto는 왜 기본생성자가 없을 때 에러를 반환할까?"
categories: Spring-Boot
tags: [Backend, SpringBoot, RequestDto]
comments: true
---


일반적으로 클라이언트가 Request Body를 통해서 서버에 API 요청할 때, 서버에서는 이걸 `RequestDto`에 매핑해서 전달 받는다. (ex. `public ResponseDto call(@RequestBody RequestDto requestDto) { ... }`)

<div align="center">

![클라이언트와 서버가 주고 받는 DTO](https://user-images.githubusercontent.com/59888684/175822524-d754f28c-7748-47e3-a4d7-ceccf12c6f54.png)

<span style="color:gray">클라이언트와 서버가 주고 받는 DTO</span>

</div>
  
<br>

여기서 만약 RequestDto에 기본 생성자를 구현하지 않았다면, 어떤 일이 벌어지게 될까?

<div align="center">

![RequestDto에 기본 생성자가 없는 경우 요청 시 벌어지는 상황](https://user-images.githubusercontent.com/59888684/175822557-6e5c369f-6daf-4e82-ba2b-b1eb0015b984.png)

<span style="color:gray">RequestDto에 기본 생성자가 없는 경우 요청 시 벌어지는 상황</span>

</div>
  
<br>


### 직렬화/역직렬화는 jackson 라이브러리를 통해서 수행된다

클라이언트 - 서버 간 요청/응답에 대한 Body 데이터와 객체 바인딩은 `jackson` 라이브러리를 통해 수행한다.

(`jackson` 라이브러리는 `spring-boot-starter-web` 라이브러리에 포함되어 있다)

클라이언트 -> 서버의 경우는 역직렬화, 서버 -> 클라이언트의 경우는 직렬화가 이루어진다.

이때, `Jackson2HttpMessageConverter`가 내부적으로 `ObjectMapper`를 사용해서 데이터를 처리한다.

`ObjectMapper`는 `Getter(public)` / `Setter`을 통해 프로퍼티명을 확인한다.

(`BeanDeserializerFactory`에서 프로퍼티명 찾는 작업 수행, `BeanDeserializer`에서 DTO에 값을 주입)

값을 주입하는 것은 `java.lang.reflect` 패키지를 사용하기 때문에 `setter` 없이도 가능하다. (`Field` 자료형)

(Getter, Setter 중 하나만 존재하면 값 주입이 가능)

### 그래서 ObjectMapper에서 기본 생성자를 어떻게 쓰고, 왜 없을 때 안되는건데?

내부 코드는 크게 이렇게 동작한다고 한다.

- **기본 생성자가 있는 경우**
    - `super.createUsingDefault()`
        - `return _defaultCreator.call()`
- **기본 생성자가 없는 경우**
    - `deserializeFromObjectUsingNonDefault()`
        - `deletegateSerializer` 사용

여기서, 기본 생성자가 없는 경우 `deletegateSerializer`를 사용해야 하는데, 이는 DTO에 생성을 위임하거나 프로퍼티를 별도로 설정이 필요하다.

이러한 조건이 만족되지 않는다면 위와 같은 예외를 반환하는 것이다.

DTO에 생성을 위임하거나 프로퍼티를 별도 설정하는 방법은 Json 관련 어노테이션을 사용하는 것이다.

(`@JsonProperty`, `@JsonAutoDetect`, `@JsonCreator`)

---

### 참고 링크

- [@RequestBody 모델에 기본생성자, setter/getter가 필요한가?](https://bbbicb.tistory.com/46)
- [@RequestBody에 왜 기본 생성자는 필요하고, Setter는 필요 없을까? #2](https://velog.io/@conatuseus/RequestBody%EC%97%90-%EC%99%9C-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%A0%95%EC%9E%90%EB%8A%94-%ED%95%84%EC%9A%94%ED%95%98%EA%B3%A0-Setter%EB%8A%94-%ED%95%84%EC%9A%94-%EC%97%86%EC%9D%84%EA%B9%8C-2-ejk5siejhh)
- [@RequestBody에 왜 기본 생성자는 필요하고, Setter는 필요 없을까? #3](https://velog.io/@conatuseus/RequestBody%EC%97%90-%EC%99%9C-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EB%8A%94-%ED%95%84%EC%9A%94%ED%95%98%EA%B3%A0-Setter%EB%8A%94-%ED%95%84%EC%9A%94-%EC%97%86%EC%9D%84%EA%B9%8C-3-idnrafiw)
