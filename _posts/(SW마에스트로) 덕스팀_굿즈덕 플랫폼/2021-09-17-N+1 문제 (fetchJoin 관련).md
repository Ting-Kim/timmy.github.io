---
layout: post
title: "Spring-boot JPA/Hibernate N+1 문제 (batch 설정)"
categories: (SW마에스트로) 덕스팀_굿즈덕 플랫폼
tags: [spring-boot, firebase, rtdb, nosql]
comments: true
---

# N+1 문제 (fetchJoin 관련)

참고 링크

- [https://pasudo123.tistory.com/426](https://pasudo123.tistory.com/426)

---

Item에는 List<Image> 를 가지고 있어서 아이템을 여러개 조회할 때 마다 이미지 쿼리가 아이템 갯수만큼 날아감.

근데 배포 서버에도 비슷하게 구현한 경우인데, 이 경우에는 쿼리가 한번 나간다. (아래 이미지)

![20210917_1](https://github.com/Ting-Kim/Ting-kim.github.io/blob/main/images/20210917_1.png?raw=true)

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 100
```

위와 같은 설정으로 인해서 `Hibernate`가 배치 처리를 지원해주는 것 같다.
