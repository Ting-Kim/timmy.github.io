---
layout: post
title: "[Spring boot] FCM Initializer 실행 안됨"
categories: (SW마에스트로)GOODSDUCK
tags: [spring-boot, fcm, firebase]
comments: true
---

# [Spring boot] FCM Initializer 실행 안됨

팀원들이랑 이제 구글 플레이스토어와 애플 앱스토어에 앱을 배포하는 일만 남았는데, 갑자기 FCM 메시지가 작동하지 않았다. 

에러 문구: `FirebaseApp with name [DEFAULT] doesn't exist`

원인을 찾다가 삽질을 굉장히 오래했는데, 서버 로그를 뒤져보니 `FCMInitializer`가 아예 실행되지 않는 것 같았다.

기존에 `FCMInitializer`에서 필요한 환경변수를 `PropertyUtil` 클래스를 통해서 받아왔는데, 확인해보니 환경변수를 읽어오지를 못했다.

스프링 빈 라이프 사이클에 대해서 생각해보니 많이 모르고 있었고, 이와 관련된다고 생각하여 여기저기 찾아봤다.

결국 스프링 내부에서는 스프링 컨테이너가 생성되고 스프링 빈들이 생성되는데, 여기서 **빈 생성 → 빈 초기화(@PostConstruct 등) → 사용 → 빈 소멸 전 콜백(@PreDestroy 등)** 으로 이루어진다.

`application context`는 모든 빈들이(`BeanFactory`는 Lazy, `ApplicartionContext`는 Eager) 초기화 되어야(?) 로딩된다. `PropertyUtil`은 `applicaiton context` 에서 환경변수를 가져오는데, `FCMInitializer`(Bean)가 생성될 때 `application context`는 로딩이 완료되지 않았기 때문에 환경변수를 못 읽어오는 것 같았다.

그래서 클래스 변수의 `final`을 제거하고, `@Value` 어노테이션을 통해서 가져오는 형태로 변경해서 해결했다.

(`@Value`는 `@PostConstruct` 전에 불러올 수 있다고 함)

```sql
//PropertyUtil.java
@Slf4j
public class PropertyUtil {

    public static final String SUBJECT_OF_JWT = "For Member-Checking";
    public static final String KEY_OF_USERID_IN_JWT_PAYLOADS = "userId";
    public static final Integer PAGEABLE_SIZE = 20;
    public static final Integer POST_PAGEABLE_SIZE = 3;
    public static final String BASIC_IMAGE_URL = "https://goodsduck-s3.s3.ap-northeast-2.amazonaws.com/sample_goodsduck.png";

    public static String getProperty(String propertyName) {

        var applicationContext = ApplicationContextServe.getApplicationContext();

        if (applicationContext != null && applicationContext.getEnvironment().getProperty(propertyName) != null) {
            return applicationContext.getEnvironment().getProperty(propertyName);
        } else if (applicationContext == null) {
            log.debug("applicationContext was not loaded!");
        } else if (applicationContext.getEnvironment().getProperty(propertyName) == null) {
            log.debug(propertyName + " properties was not loaded!");
        }
        return "No Value";
    }
}
```

```sql
// 수정 전
@Service
@Slf4j
public class FCMInitializer {
    private final String FIREBASE_CONFIG_PATH = PropertyUtil.getProperty("firebase.config-path");
    private final String FIREBASE_DATABASE_URL = PropertyUtil.getProperty("firebase.database-url");
    @PostConstruct
    public void initialize() {
        try {
            log.debug("FCMInitializer processed!");
            FirebaseOptions options = FirebaseOptions.builder()
                    .setCredentials(GoogleCredentials.fromStream(
                            new ClassPathResource(FIREBASE_CONFIG_PATH).getInputStream()
                    ))
                    .setDatabaseUrl(FIREBASE_DATABASE_URL)
                    .build();

            if (FirebaseApp.getApps().isEmpty()) {
                log.info("Firebase application is empty..");
                FirebaseApp.initializeApp(options);
                log.info("Firebase application has been initialized");
            }
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
}
```

```sql
// 수정 후
@Service
@Slf4j
public class FCMInitializer {

    @Value("${firebase.config-path}")
    private String FIREBASE_CONFIG_PATH;

    @Value("${firebase.database-url}")
    private String FIREBASE_DATABASE_URL;

    @PostConstruct
    public void initialize() {
        try {
            log.debug("FCMInitializer processed!");
            log.info("FCM path: " + FIREBASE_CONFIG_PATH);
            FirebaseOptions options = FirebaseOptions.builder()
                    .setCredentials(GoogleCredentials.fromStream(
                            new ClassPathResource(FIREBASE_CONFIG_PATH).getInputStream()
                    ))
                    .setDatabaseUrl(FIREBASE_DATABASE_URL)
                    .build();

            if (FirebaseApp.getApps().isEmpty()) {
                log.info("Firebase application is empty..");
                FirebaseApp.initializeApp(options);
                log.info("Firebase application has been initialized");
            }
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
}
```

---

참고 링크

- [https://yonguri.tistory.com/100](https://yonguri.tistory.com/100)
- [https://kth990303.tistory.com/26](https://kth990303.tistory.com/26)
