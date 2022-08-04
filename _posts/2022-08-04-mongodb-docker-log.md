---
layout: post
title: Mongodb docker container log 저장
excerpt: Mongodb docker container log 저장
date: 2022-08-04
tags: [docker, mongodb, ubunutu]
toc: true
---

## Mongodb docker container log 디렉토리 볼륨 연결 문제

mongodb docker container 이용시 log가 저장되는 부분을 호스트와 volume으로 연결시 실행이 되지 않는 현상이 발생했다.

volume 디렉토리 권한 문제인듯하여 다음과 같이 하여 해결하였다. (v4.4.1 기준, 우분투)

> 1. 볼륨 연결하지 않은 상태로 mongodb container 실행.
> 2. 컨테이너에 내부로 돌입
> 3. 로그가 기록되는 /var/log/mongodb 디렉토리의 소유자를 root:root 로 변경하고 권한을 777로 설정한다.
> 4. 볼륨연결할 호스트의 디렉토리의 디렉토리 또한 권한을 777로 변경한다.

명령어

```bash
# 컨테이너 실행
$ docker run -it -d --rm --name db mongo:4.4.1
# 컨테이너 내부 bash 실행
$ docker exec -it db bash
$ cd /var/log
$ chown root:root mongodb
$ chmod 777 mongodb
# 호스트로 나오기
$ exit

# 호스트에서 volume으로 연결할 디렉토리 권한 설정
$ chmod 777 <your_log_directory>
# 이후 컨테이너 재실행전 commit으로 설정 저장
$ docker commit db <image_name>
# 컨테이너 종료후 볼륨연결하여 재실행
$ docker stop db
$ docker run -it -d --rm --name db -v <your_log_directory>:/var/log/mongodb <image_name>
```
