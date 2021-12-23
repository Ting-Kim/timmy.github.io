---
layout: post
title: "Spring boot에서 Firebase Realtime DB 접근하기"
categories: (SW마에스트로) 덕스팀_굿즈덕 플랫폼
tags: [spring-boot, firebase, rtdb, nosql]
comments: true
---

# Spring boot에서 Firebase Realtime DB 접근

[https://firebase.google.com/docs/database/admin/start?hl=ko](https://firebase.google.com/docs/database/admin/start?hl=ko)

(Java)`Spring boot 서버`에서 접근할 수 있는 방법은 생각보다 간단했다.

하지만 `Realtime Database` 특성 상 `비동기 리스너`를 통해서 데이터를 쓰고, 받아오는 것 같았다.

결국에는 `RTDB`에 변경사항이 있는 경우 트리거가 요청됨.

따라서, 기존에 채팅방 목록을 `API 서버`에서 `RTDB`를 통해 직접 가져오는 방식은 현재로서는 어려울 것 같다. (멘토님께 질문을 드려야 하나?)

`비동기 리스너`를 통해 채팅 메시지가 `RTDB`에 추가되었을 때, `FCM`과 엮어서 사용하는 형태로 할 수 있을 것 같다.

- Firebase RTDB 데이터 구조를 변경해야 하는가?
- 자바 FirebaseDatabase 리스너를 여러개 작동시켜야 하는가?
    - 채팅방이 열린 시점에서 해당 채팅방에 대한 리스너 동작 시작
        - 이렇게 되면 리스너가 많아지는 경우 문제가 초래되지는 않는가?

기존에 알림 테이블은 `NoSQL` 형태의 `Redis`를 사용하는 형태로 변경해야 한다.
