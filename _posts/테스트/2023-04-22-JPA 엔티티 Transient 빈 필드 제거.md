---
layout: post
title: "JPA 엔티티 내 @Transient Bean 필드를 제거한 이유"
categories: 테스트
tags: [junit, java, spring boot, aspectj]
comments: true
---

### 배경

간단한 API를 개발하는 작업을 진행하고 있었습니다. 해당 기능에 대한 도메인 모델(JPA 엔티티) 로직은 이미 작성되어 있는 상태라서 추가 작업이 많이 필요하지 않았습니다. 그래서 해당 모델 로직을 그대로 사용했고, 필요한 web 코드와 도메인 서비스 코드를 작성했습니다.

<br>

우선 개발 완료 후 Swagger 상으로 간단한 호출을 해봤는데 문제가 있었습니다. 이유는 도메인 JPA 엔티티 내부에 외부 의존성을 가지는 `@Transient` 필드가 존재했습니다.

<br>


```java

@Entity
public class Order {

	..
	
	
	@Transient
	@Inject
  	@Getter(AccessLevel.NONE)
	private DomainEventListener domainEventListener; // is null
	
	@Transient
	@Inject
  	@Getter(AccessLevel.NONE)
	private ImageUploadService imageUploadService; // is null

	public void order(OrderCommand orderCommand) {
		changeOrderStatus(OrderStatus.ACCEPTED);
		domainEventListener.publish(new OrderCreatedEvent(orderComamnd)); // NPE
		imageUploadService.upload(order.getOrderId(), ImageType.OrderCustomerRequest, order.getCustomerImageUrl()); // NPE
	}

	..
}
```

이처럼 스프링 컨테이너가 관리하는 빈이 아닌 JPA 엔티티에는 의존성을 Spring이 직접 주입해줄 수 없고, 필요하다면 Load time weaving 이라는 것을 사용해야 합니다. 제가 알기로는 해당 기능을 사용하려면 AspectJ 라이브러리 관련해서 설정이 필요한데, 설정이 적용되어 있지 않았습니다.

[엔티티에 의존성 주입이 필요한 경우? - 인프런 질문 & 답변](https://www.inflearn.com/questions/24903/%EC%97%94%ED%8B%B0%ED%8B%B0%EC%97%90-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85%EC%9D%B4-%ED%95%84%EC%9A%94%ED%95%9C-%EA%B2%BD%EC%9A%B0)

(물론 저도 영한님의 의견처럼 도메인 모델 내부에 외부 의존성을 가지는 것에 반대하는 입장이긴 했습니다  😂)

<br>

그래서 엔티티 내 해당 필드는 null인 상태였고, 메서드에서 NPE가 발생한 것이었습니다.

이를 해결하기 위해 다른 프로젝트를 참고하여 maven plugin 설정과 main 클래스에 관련 어노테이션들을 추가하여 설정해주었습니다.

<br>

API 호출을 재시도 해보니 잘 동작하였습니다. 하지만, 다른 곳에서 문제가 발생했습니다.

바로 테스트 코드였습니다.

해당 도메인 서비스에 대한 테스트 코드를 작성하는 과정에서 몇 가지 단점이 존재했습니다.

- Mocking 제한적
- 불가피한 `@SpringBootTest`
- `@DataJpaTest` 사용 불가능
- 테스트 메인 메서드에도 Load time weaving 설정 필요

<br>

그래서, 결국 Load time weaving 관련해서 설정해주었던 작업들은 롤백하고 엔티티 내부의 외부 의존성도 제거하기로 했습니다. (다행히 신규 개발 중인 모델에서는 외부 의존성 필드가 별로 없었습니다 ㅎㅎ..)

<br>

```java

@Entity
public class Order {

	..
	

	public void order(OrderCommand orderCommand) {
		changeOrderStatus(OrderStatus.ACCEPTED);
	}

	..
}
```

<br>

외부 의존성을 도메인 모델로 부터 분리하고, 외부 의존성 관련 로직은 응용 서비스의 역할이라 판단하여 응용 서비스 계층으로 옮겨주었습니다.

<br>

```java
@Service
@Transactional
@RequiredArgsConstructor
public class OrderApplicationService {

	..
	private final OrderCreateService orderCreateService;
	private final DomainEventListener domainEventListener;
	private final ImageUploadService imageUploadService;

	public void order(OrderRequestDto dto) {
		orderCreateService.order(new OrderComamnd(dto.getOrderId(),
																							dto.getOrderMenus(),
																							dto.getOrderDateTime()));
		domainEventListener.publish(new OrderCreatedEvent(dto));
		imageUploadService.upload(dto.getOrderId(), 
															ImageType.OrderCustomerRequest, 
															dto.getCustomerImageUrl());
	}

	..
}
```

<br>

이렇게 도메인 엔티티로 부터 외부 의존성을 제거했더니, 도메인 메서드에 순수 도메인에 대한 로직만 남아서 테스트가 수월했습니다.

이번 포스팅에 작성한 사례를 겪으며 ‘테스트를 작성하기 쉬운 코드가 좋은 코드다 ‘ 라는 말에 공감할 수 있었습니다. 이래서 TDD가 각광을 받는걸까요?

테스트 코드를 작성하다가 불편함을 느낀다면 개발된 형상이 좋은 구조인지 한번 쯤 되돌아보는 시간을 가지면 좋을 것 같습니다 :)
