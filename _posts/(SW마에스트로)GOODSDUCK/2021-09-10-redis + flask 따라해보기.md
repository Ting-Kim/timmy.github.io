---
layout: post
title: "redis + flask 따라해보기"
categories: (SW마에스트로)덕스팀_GOODSDUCK
tags: [redis, redis-server, redis-cli, nosql, flask]
comments: true
---

# redis + flask 따라해보기

참고: [https://niceman.tistory.com/200](https://niceman.tistory.com/200)

- `$ sudo apt install python3-pip`
    - pip 버전이 너무 낮음
    - 따라서 다시 설치해야 하는데, 파이썬 버전 여러개를 관리하는게 필요할 듯.

- [여기](https://lhy.kr/configuring-the-python-development-environment-with-pyenv-and-virtualenv)서 `pyenv`, `virtualenv` 이용방법 참고
    - `$ sudo apt install git curl`
    - `$ curl -L [https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer](https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer) | bash`
    - `~/.profile`, `~/.bashrc`(여긴 꼭 해야되는지 모르겠음)
    
    ```bash
    $ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
    $ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
    $ echo 'eval "$(pyenv init --path)"' >> ~/.profile
    $ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.profile
    ```
    
    - `$ pyenv`
        - 설치되었는지 확인
    - `$ sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
    libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
    xz-utils tk-dev libffi-dev liblzma-dev python-openssl git`
        - `pyenv`를 `Ubuntu`에서 사용하기 위한 패키지들 설치

- `$ pyenv install —list` 로 설치가능한 버전들을 조회
- `$ pyenv install 3.8.3` → `파이썬 3.8.3` 버전을 설치한다.
- `$ pyenv versions` → `pyenv`로 설치한 파이썬 버전들을 나열
- `$ pyenv virtualenv <version> <virtual_name>` → `pyenv`로 특정 파이썬 버전에 대한 가상환경 설치
- `$ pyenv local <virtual_name>` 특정 디렉토리에 가상환경을 적용

- `vsc`로 `EC2 인스턴스` 내에 접근해서 작업하기 ([참고](https://tttap.tistory.com/141))
- 예제를 따라하다가 `$ rq-dashboard` 하는 단계에서 `ValueError: 'default' must be a list when 'multiple' is true.` 에러가 발생한다.
    - 패키지 `click==8.0.0` 부터 발생하는 문제라고 하는데, `$ pip install click==7.1.2` 로 디그레이드 해줬더니 됨.
