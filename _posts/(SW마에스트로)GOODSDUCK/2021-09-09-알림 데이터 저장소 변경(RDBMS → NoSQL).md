---
layout: post
title: "알림 데이터 저장소 변경 시 데이터 구조 등 (RDBMS → NoSQL)"
categories: (SW마에스트로)덕스팀_GOODSDUCK
tags: [spring-boot, firebase, rtdb, nosql]
comments: true
---

# 알림 데이터 저장소 변경(RDBMS → NoSQL)

---

### 기존 RDBMS

![20210909_1](https://github.com/Ting-Kim/Ting-kim.github.io/blob/main/images/20210909_1.png)

### NoSQL

```bash
표시되는 알림 내용
hellodowonkwon님이 "엑소 핸드선풍기" 굿즈에 가격 제안을 했어요. [8000원]
1시간전
userId(receiver):notificationId (senderNickName), (notificationTitle), (notificationType), (<>itemPrice), (notificationCreatedAt)
```

```json
// snapshot = DataSnapshot 
{ key = chatRooms, value = 
	{
	 -Mirjowqjprwqrqwrqw={item={price=10000, imageUrl=https://s3.ap-northeast-2.amazonaws.com/3.GIF, name=gif 테스트, id=3}, createdWith={bcryptId=$RJ#O@JROPH#QOPFHOPQFH, nickName=makkk, isPresented=true, id=1, profileImg=https://s3.ap-northeast-2.amazonaws.com/4.jpg}, createdBy={nickName=팅카오, isPresented=true, id=757}, id=-Mirjowqjprwqrqwrqw10, timestamp=1630376711720}, 

	-Mirjowqjprwqrqwrqw2={item={price=10000, imageUrl=https://s3.ap-northeast-2.amazonaws.com/5.GIF, name=gif 테스트, id=3}, createdWith={bcryptId=$RJ#O@JROPH#QOPFHOPQFH, nickName=makkk, isPresented=true, id=1, profileImg=https://s3.ap-northeast-2.amazonaws.com/6.jpg}, createdBy={nickName=팅카오, isPresented=true, id=757}, id=-Mirjowqjprwqrqwrqw11, timestamp=1630376711720}, 

. 
.
.

	
}
```

```json
{
	"notification": {
		"user:3": [
			"nonChat:1": {
				"id": 1,
				"type": "REVIEW_FIRST",
				"reviewId": 1,
				"priceProposeId": null,
				"priceProposePrice": null,
				"senderNickName": "도원결의",
				"itemId": 5,
				"itemName": "블랙핑크 굿즈 한번에 마니 사요",
				"createdAt": "2010-04-15 15:42:15",
				"isRead": 0,				
			},
			"nonChat:2": {
				"id": 2,
				"type": "PRICE_PROPOSE",
				"reviewId": null,
				"priceProposeId": 3,
				"priceProposePrice": 15000,
				"senderNickName": "도원결의",
				"itemId": 5,
				"itemName": "블랙핑크 굿즈 한번에 마니 사요",
				"createdAt": "2010-04-15 15:42:15",
				"isRead": 0,				
			},
			.
			.
		]

# chat noty와 일반 noty를 분리하면?
{
	"chat": {
		"user:3": {
			"chatRoom:-Modewfwreww-wqr21r213": [
				"chat:-Modewfwreww-wqr21r212": {
					"type": "CHAT",
					"senderNickName": "Makkkk",
					"createdAt": "2010-04-05 12:00:11",
					"isRead": 1,
				},
				{
				},
				{
				},
				{
				},
				{
				},
				{
				},
				.
				.
			]
		},
		"chatRoom:-M2rw24rrwq-wkkgmetm": {
		},
		.
		.
	}
}
```

### 알림 데이터 정리

- 가격 제시
    - type
    - priceProposeId
    - userId (receiver)
    - senderNickName
    - **itemName**
    - **itemPrice**
    - createdAt
    - isRead
- 채팅
    - type
    - chatRoomId
    - userId (receiver)
    - senderNickName
    - createdAt
    - isRead
- 선 리뷰
    - type
    - reviewId
    - userId (receiver)
    - senderNickName
    - createdAt
    - isRead
- 후 리뷰
    - type
    - reviewId
    - userId (receiver)
    - senderNickName
    - createdAt
    - isRead
- 커뮤니티 댓글
    - type
    - PostId
    - userId (receiver)
    - senderNickName
    - createdAt
    - isRead

---

### NoSQL 데이터 구조 설계 시 고려사항

> 알림 데이터는 어떠한 `Key`로 검색이 되는가?
> 

> 읽기/쓰기 비율은 어떻게 되는가?
> 

> 집계가 필요한가?
> 

> 데이터 양은 얼마나 되는가?
> 

> **보통 NoSQL은 뎁스가 없는 형태로 구현된다.**
> 

user:userId:notification:notificationId : {}

user:userId:chatRoom:chatRoomId : { "id": 1, "content": "안녕하세요" .. }

이런식?

```json
{
    "success": false,
    "response": null,
    "error": {
        "message": "Could not read JSON: Cannot construct instance of `java.time.LocalDateTime` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)\n at [Source: (byte[])\"{\"id\":\"opjpqorjpo-rwqqwjrpoqw-jopr\",\"type\":\"PRICE_PROPOSE\",\"reviewId\":null,\"priceProposeId\":29,\"priceProposePrice\":50000,\"senderNickName\":\"hello\",\"itemId\":104,\"itemName\":\"누군진 버터앨범 모르지만 파라용\",\"createdAt\":{\"year\":2021,\"monthValue\":9,\"dayOfMonth\":2,\"hour\":21,\"minute\":37,\"second\":36,\"month\":\"SEPTEMBER\",\"dayOfWeek\":\"THURSDAY\",\"dayOfYear\":245,\"nano\":357880000,\"chronology\":{\"id\":\"ISO\",\"calendarType\":\"iso8601\"}},\"expiredAt\":{\"year\":2021,\"monthValue\":9,\"dayOfMonth\":2,\"\"[truncated 176 bytes]; line: 1, column: 241] (through reference chain: com.ducks.goodsduck.commons.model.redis.NotificationRedisDto[\"createdAt\"]); nested exception is com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `java.time.LocalDateTime` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)\n at [Source: (byte[])\"{\"id\":\"opjpqorjpo-rwqqwjrpoqw-jopr\",\"type\":\"PRICE_PROPOSE\",\"reviewId\":null,\"priceProposeId\":29,\"priceProposePrice\":50000,\"senderNickName\":\"hello\",\"itemId\":104,\"itemName\":\"누군진 버터앨범 모르지만 파라용\",\"createdAt\":{\"year\":2021,\"monthValue\":9,\"dayOfMonth\":2,\"hour\":21,\"minute\":37,\"second\":36,\"month\":\"SEPTEMBER\",\"dayOfWeek\":\"THURSDAY\",\"dayOfYear\":245,\"nano\":357880000,\"chronology\":{\"id\":\"ISO\",\"calendarType\":\"iso8601\"}},\"expiredAt\":{\"year\":2021,\"monthValue\":9,\"dayOfMonth\":2,\"\"[truncated 176 bytes]; line: 1, column: 241] (through reference chain: com.ducks.goodsduck.commons.model.redis.NotificationRedisDto[\"createdAt\"])",
        "status": 500
    }
}
```

위와 같은 에러는 [여기](https://eeyatho.tistory.com/4)를 참고했다. Serialize 해주는 Jackson 라이브러리가 Java8의 LocalDateTime 자료형을 모르기 때문이라고 한다. 일단 외부 라이브러리를 사용하라는 해결책을 봤는데, 우선 참고한 링크에서 사용한 어노테이션이 현재도 사용가능하길래 추가해봤다.

```java
package com.ducks.goodsduck.commons.model.redis;

import com.ducks.goodsduck.commons.model.enums.NotificationType;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateDeserializer;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.Id;
import java.io.Serializable;
import java.time.LocalDateTime;
import java.util.UUID;

import static com.ducks.goodsduck.commons.model.enums.NotificationType.PRICE_PROPOSE;

@Data
@NoArgsConstructor // 링크에서 생성자를 못찾아서 에러가 발생한다는 내용이 있어서 추가
@AllArgsConstructor // 링크에서 생성자를 못찾아서 에러가 발생한다는 내용이 있어서 추가
public class NotificationRedisDto implements Serializable {

    @Id
    private String id;
    private NotificationType type;
    private Long reviewId;
    private Long priceProposeId;
    private Integer priceProposePrice;
    private String senderNickName;
    private Long itemId;
    private String itemName;
    
		@JsonDeserialize(using = LocalDateTimeDeserializer.class)
    @JsonSerialize(using = LocalDateTimeSerializer.class)
    private LocalDateTime createdAt;

    @JsonDeserialize(using = LocalDateTimeDeserializer.class)
    @JsonSerialize(using = LocalDateTimeSerializer.class)
    private LocalDateTime expiredAt;

    private Boolean isRead;

    public void read() {
        this.isRead = true;
    }

    // HINT: REVIEW
    public NotificationRedisDto(NotificationType type, Long reviewId, String senderNickName) {
        this.id = UUID.randomUUID().toString();
        this.type = type;
        this.reviewId = reviewId;
        this.senderNickName = senderNickName;
        this.createdAt = LocalDateTime.now();
//        this.expiredAt = createdAt.plusWeeks(2L);
        this.expiredAt = createdAt.plusMinutes(1L);
        this.isRead = false;
    }

    // HINT: PricePropose
    public NotificationRedisDto(Long priceProposeId, Integer priceProposePrice, Long itemId, String itemName, String senderNickName) {
        this.id = UUID.randomUUID().toString();
        this.type = PRICE_PROPOSE;
        this.senderNickName = senderNickName;
        this.priceProposeId = priceProposeId;
        this.priceProposePrice = priceProposePrice;
        this.itemId = itemId;
        this.itemName = itemName;
        this.createdAt = LocalDateTime.now();
//        this.expiredAt = createdAt.plusWeeks(2L);
        this.expiredAt = createdAt.plusMinutes(1L);
        this.isRead = false;
    }
}

```

결국 `@Deserializer`, `@Serializer` 어노테이션 모두 붙여서 해결되었다! 

Notification  데이터 구조

Chat 데이터 구조

```java
// ChatRedis

/** REDIS */
{
	"user:3:chatRoom:-Mwroqjorq-ojwrowjqrq": {
		[
			{
				"id": "-Mnwqorlxf-owjrw2kt",
				"chatRoomId": "-Mwroqjorq-ojwrowjqrq",
				"senderNickName": "도원결의",
				"content": "안녕하세요",
				"createdAt": "",
			},
			{
				"id": "-Mnwqorlxf-nwjrw2kt",
				"chatRoomId": "-Mwroqjorq-ojwrowjqrq",
				"senderNickName": "makkk",
				"content": "네 하이요",
				"createdAt": "",
			}
		]
	}
	
	
}

{
	"user:3:notification": {
		[
			{
				"id": "-Mnwqorlxf-owjrw2kt",
				"chatRoomId": "-Mwroqjorq-ojwrowjqrq",
				"senderNickName": "도원결의",
				"content": "안녕하세요",
				"createdAt": "",
			},
			{
				"id": "-Mnwqorlxf-nwjrw2kt",
				"chatRoomId": "-Mwroqjorq-ojwrowjqrq",
				"senderNickName": "makkk",
				"content": "네 하이요",
				"createdAt": "",
			}
		]
	}
	
	
}
```

```java
// 유저 ID에 해당하는 채팅방 목록 조회 API

/** json */
{
	[
		"chatRoomId": "-Mwroqjorq-ojwrowjqrq",
		"chat": [
			{
				"id": "-Mnwqorlxf-nwjrw2kt",
				"chatRoomId": "-Mwroqjorq-ojwrowjqrq",
				"senderNickName": "makkk",
				"content": "네 하이요",
				"createdAt": "",
			},
			{
				"id": "-Mnwqorlxf-nwjrw2kt",
				"chatRoomId": "-Mwroqjorq-ojwrowjqrq",
				"senderNickName": "makkk",
				"content": "네 하이요",
				"createdAt": "",
			}
		]
	]
	"user:3:chatRoom:-Mwroqjorq-ojwrowjqrq": {
		[
			{
				"id": "-Mnwqorlxf-owjrw2kt",
				"chatRoomId": "-Mwroqjorq-ojwrowjqrq",
				"senderNickName": "도원결의",
				"content": "안녕하세요",
				"createdAt": "",
			},
			{
				"id": "-Mnwqorlxf-nwjrw2kt",
				"chatRoomId": "-Mwroqjorq-ojwrowjqrq",
				"senderNickName": "makkk",
				"content": "네 하이요",
				"createdAt": "",
			}
		]
	}
	
	
}
```

### 참고 링크

- [https://tape22.tistory.com/25](https://tape22.tistory.com/25)
- Serializer 문제
    - [https://minholee93.tistory.com/entry/ERROR-DefaultSerializer-requires-a-Serializable-payload-but-received-an-object-of-type](https://minholee93.tistory.com/entry/ERROR-DefaultSerializer-requires-a-Serializable-payload-but-received-an-object-of-type)
- Redis는 Quick List를 사용한다
    - [http://redisgate.kr/redis/configuration/internal_quicklist.php](http://redisgate.kr/redis/configuration/internal_quicklist.php)
- Redis에서 keys가 아닌, Scan으로 key들을 조회하는 방법
    - [https://jaepils.github.io/redis/2018/11/20/redis-scan.html](https://jaepils.github.io/redis/2018/11/20/redis-scan.html)

```java
package com.ducks.goodsduck.commons.service;

import com.ducks.goodsduck.commons.repository.SmsAuthenticationRepository;
import com.ducks.goodsduck.commons.util.AwsSecretsManagerUtil;
import com.ducks.goodsduck.commons.util.PropertyUtil;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import net.nurigo.java_sdk.api.Message;
import net.nurigo.java_sdk.exceptions.CoolsmsException;
import org.json.JSONObject;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Random;

@Service
@RequiredArgsConstructor
@Slf4j
public class SmsAuthenticationService {

    private final JSONObject jsonOfAwsSecrets = AwsSecretsManagerUtil.getSecret();
    private String apiKey = jsonOfAwsSecrets.optString("coolsms.apikey", "");
    private String apiSecret = jsonOfAwsSecrets.optString("coolsms.apisecret", "");
    private String senderNumber = jsonOfAwsSecrets.optString("coolsms.sendernumber", "");

    private final SmsAuthenticationRepository smsAuthenticationRepository;

    public Boolean sendSmsOfAuthentication(String phoneNumber) throws Exception {
        if (jsonOfAwsSecrets.isEmpty()) {
            apiKey = PropertyUtil.getProperty("coolsms.apikey");
            apiSecret = PropertyUtil.getProperty("coolsms.apisecret");
            senderNumber = PropertyUtil.getProperty("coolsms.sendernumber");
        }

        Message coolsms = new Message(apiKey, apiSecret);
        String authenticationNumber = generateAuthenticationNumber();

        // 4 params(to, from, type, text) are mandatory. must be filled
        HashMap<String, String> params = new HashMap<>();
        params.put("to", phoneNumber);    // 수신전화번호
        params.put("from", senderNumber);    // 발신전화번호. 테스트시에는 발신,수신 둘다 본인 번호로 하면 됨
        params.put("type", "SMS");
        params.put("text", "굿즈덕(GOODSDUCK) 인증번호는 \"" + authenticationNumber + "\" 입니다.");
        params.put("app_version", "goodsduck app 1.0"); // application name and version

        try {
            org.json.simple.JSONObject sendedMessage = coolsms.send(params);
            log.debug("Send message from CoolSMS: {}", sendedMessage.toString());
            if (!sendedMessage.get("error_count").equals(0L)) {
                return false;
            }
            smsAuthenticationRepository.saveKeyAndValueOfSMS(phoneNumber, authenticationNumber);
        } catch (CoolsmsException e) {
            throw new CoolsmsException("Exception of CoolSMS: " + e.getMessage(), e.getCode());
        } catch (Exception e) {
            throw new Exception("Exception occurred in sending CoolSms: " + e.getMessage());
        }
        return true;
    }

    public Boolean authenticate(String phoneNumber, String authenticationNumber) {
        if (smsAuthenticationRepository.hasKey(phoneNumber)) {
            if (smsAuthenticationRepository.getValueByPhoneNumber(phoneNumber).equals(authenticationNumber)) {
                smsAuthenticationRepository.removeKeyAndValueOfSMS(phoneNumber);
                return true;
            }
        }

        return false;
    }

    public String generateAuthenticationNumber() {
        Random random  = new Random();
        String randomNumbers = "";
        for(int i=0; i<6; i++) {
            String randomNumber = Integer.toString(random.nextInt(10));
            randomNumbers = randomNumbers.concat(randomNumber);
        }
        return randomNumbers;
    }
}
```

```java
package com.ducks.goodsduck.commons.repository;

import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Repository;

import java.time.Duration;

@Repository
public class SmsAuthenticationRepository {

    private final StringRedisTemplate stringRedisTemplate;
//    ListOperations<String, NotificationRedisDto> stringNotificationRedisDtoListOperations;

    private final String PREFIX_OF_SMS = "sms:";
    private final Long TTL_OF_SMS = 3 * 60L;

    public SmsAuthenticationRepository(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
//        stringNotificationRedisDtoListOperations = stringRedisTemplate.opsForList();
    }

    public void saveKeyAndValueOfSMS(String phoneNumber, String authenticationNumber) {
        stringRedisTemplate.opsForValue().set(PREFIX_OF_SMS + phoneNumber, authenticationNumber, Duration.ofSeconds(TTL_OF_SMS));
    }

    public String getValueByPhoneNumber(String phoneNumber) {
        return stringRedisTemplate.opsForValue().get(PREFIX_OF_SMS + phoneNumber);
    }

    public void removeKeyAndValueOfSMS(String phoneNumber) {
        stringRedisTemplate.delete(PREFIX_OF_SMS + phoneNumber);
    }

    public boolean hasKey(String phoneNumber) {
        return stringRedisTemplate.hasKey(PREFIX_OF_SMS + phoneNumber);
    }

    public void saveKeyAndSet(String key) {
//        stringRedisTemplate.opsForSet().
    }
}
```
