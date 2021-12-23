---
layout: post
title: "Redis-Server 설치하기 (파일 형태)"
categories: (SW마에스트로) 덕스팀_굿즈덕 플랫폼
tags: [redis, redis-server, redis-cli, nosql]
use_math: true
comments: true
---

{% raw %}

# redis-server 설치

Ubuntu 18.0.4 (AWS EC2) 기준

```bash
### wget 대신 다운로드 페이지에서 최신 버젼을 직접 받아도 된다 
$ wget http://download.redis.io/redis-stable.tar.gz 
$ tar xvzf redis-stable.tar.gz 
$ cd redis-stable
$ make

#출처:
#[https://dgkim5360.tistory.com/entry/install-redis-for-linux-or-windows](https://dgkim5360.tistory.com/entry/install-redis-for-linux-or-windows)
#[개발새발로그]
```

에러 발생, pkg-config가 없다고 함.

그래서,

```bash
$ sudo apt-get update
$ sudo apt install pkg-config
```

하니까 

```bash
cd src && make all
make[1]: Entering directory `/opt/redis-5.0.7/src'
    CC Makefile.dep
make[1]: Leaving directory `/opt/redis-5.0.7/src'
make[1]: Entering directory `/opt/redis-5.0.7/src'
    CC adlist.o
In file included from adlist.c:34:0:
zmalloc.h:50:10: fatal error: jemalloc/jemalloc.h: No such file or directory
 #include <jemalloc/jemalloc.h>
          ^~~~~~~~~~~~~~~~~~~~~^
```

이런 에러를 뱉었다.. 찾아보니까 이 방법이 잘 맞았다!

```bash
[root@redis-5.0.7]# make clean
[root@redis-5.0.7]# cd deps
[root@deps]# make hiredis jemalloc linenoise lua
[root@deps]# cd ..
[root@redis-5.0.7]# make

# 출처: https://mozi.tistory.com/536
```

하지만 다시 이런 에러가..

```bash
cc: error: ../deps/hdr_histogram/hdr_histogram.o: No such file or directory
Makefile:359: recipe for target 'redis-benchmark' failed
make[1]: *** [redis-benchmark] Error 1
make[1]: Leaving directory '/home/ubuntu/redis-stable/src'
Makefile:6: recipe for target 'all' failed
make: *** [all] Error 2
```

근데 그냥 `make distclean` 하면 된다고 해서 해봤더니 잘 됐다.. (출처: [여기](https://the7sign.github.io/server/2019/06/20/redis_install_errorfix.html))

make는 잘 넘어갔는데.. make test 했더니 이런 에러가 또..

```bash
cd src && make test
make[1]: Entering directory '/home/ubuntu/redis-stable/src'
    CC Makefile.dep
You need tcl 8.5 or newer in order to run the Redis test
Makefile:385: recipe for target 'test' failed
make[1]: *** [test] Error 1
make[1]: Leaving directory '/home/ubuntu/redis-stable/src'
Makefile:6: recipe for target 'test' failed
make: *** [test] Error 2
```

`sudo apt-get install tcl` 한 후에 잘 작동하는 것 같더니.. 끝에

```bash
I/O error reading reply
    while executing
"{*}$r rpop $k"
    ("uplevel" body line 1)
    invoked from within
"uplevel 1 [lindex $args $path]"
    (procedure "randpath" line 3)
    invoked from within
"randpath {{*}$r lpush $k $v}  {{*}$r rpush $k $v}  {{*}$r lrem $k 0 $v}  {{*}$r rpop $k}  {{*}$r lpop $k}"
    (procedure "createComplexDataset" line 51)
    invoked from within
"createComplexDataset $r $ops"
    (procedure "bg_complex_data" line 5)
    invoked from within
"bg_complex_data [lindex $argv 0] [lindex $argv 1] [lindex $argv 2] [lindex $argv 3] [lindex $argv 4]"
    (file "tests/helpers/bg_complex_data.tcl" line 13)
I/O error reading reply
    while executing
"{*}$r lpush $k $v"
    ("uplevel" body line 2)
    invoked from within
"uplevel 1 [lindex $args $path]"
    (procedure "randpath" line 3)
    invoked from within
"randpath {
                {*}$r set $k $v
            } {
                {*}$r lpush $k $v
            } {
                {*}$r sadd $k $v
        ..."
    (procedure "createComplexDataset" line 30)
    invoked from within
"createComplexDataset $r $ops"
    (procedure "bg_complex_data" line 5)
    invoked from within
"bg_complex_data [lindex $argv 0] [lindex $argv 1] [lindex $argv 2] [lindex $argv 3] [lindex $argv 4]"
    (file "tests/helpers/bg_complex_data.tcl" line 13)I/O error reading reply
    while executing
"{*}$r type $k"
    (procedure "createComplexDataset" line 27)
    invoked from within
"createComplexDataset $r $ops"
    (procedure "bg_complex_data" line 5)
    invoked from within
"bg_complex_data [lindex $argv 0] [lindex $argv 1] [lindex $argv 2] [lindex $argv 3] [lindex $argv 4]"
    (file "tests/helpers/bg_complex_data.tcl" line 13)

Killing still running Redis server 21467
Killing still running Redis server 21485
Killing still running Redis server 21498
Makefile:385: recipe for target 'test' failed
make[1]: *** [test] Error 1
make[1]: Leaving directory '/home/ubuntu/redis-stable/src'
Makefile:6: recipe for target 'test' failed
make: *** [test] Error 2
```

이건 무슨 에러냐..ㅠㅠ

`sudo apt-get install tcl8.5` 했더니 된다..? 참고는 [여기](https://bryan7.tistory.com/801)

근데 redis를 파일 형태로 다운 받아서 쓰는건 좋은 방법은 아니라고 한다. 
그래서 배포 서버에서 sudo apt-get install redis-server 등의 명령어로 설치하여 변경했다.

{% endraw %}
