---
layout: post
title: Nginx 파일 용량 제한 문제 (CORS)
categories: (SW마에스트로)덕스팀_GOODSDUCK
tags: [Nginx, CORS]
---


# Nginx 파일 용량 제한 문제 (CORS)

1 MB 이상으로는 파일 전송이 안되는 문제가 있었다.

개발자 도구에는 CORS 에러가 발견되기도 했는데, 찾아보니 Nginx에서 따로 파일 용량제한에 대한 설정을 하지 않았을 때 나오는 에러라고 한다.

Ubuntu 18.0 기준 `/etc/nginx/nginx.conf` 에서 server 탭에 

```java
server {

		..

		**client_max_body_size [원하는 용량]; 

		..

}**
```

추가로 해결되었다.

해결에 참고한 글

[https://storyinglass.tistory.com/11](https://storyinglass.tistory.com/11)
