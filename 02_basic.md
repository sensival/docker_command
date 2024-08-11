## 1. hello world 실행하기

```bash
docker container run diamol/ch02-hello-diamol
```
### 

## 2. 컨테이너를 컴퓨터처럼 사용하기
```bash
docker container --interactive --tty diamol/base
```
이 상태에서 아래 명령어를 치면 컨테이너 내 환경 확인
```bash
hostname
date
```
다른 cmd 창을 열어 아래 명령어를 입력
```bash
docker container ls
# 여기서 결과로 나온 CONTAINER ID가 abc...... 이라고 하면

docker container top abc
# 컨테이너 안에서 실행 중인 프로세스 확인

docker container logs abc
# 로그 확인

docker container inspect abc
# 상세정보 확인
```
### 

## 3. 웹사이트 호스팅하기
```bash
docker container run --detach --publish 8088:80 diamol/ch02-hello-diamol-web
# --detach: 백그라운드 실행 --publish: 포트를 호스트에 공개
```
호스트 컴퓨터 웹브라우저에서 localhost:8088을 하면 웹브라우저를 확인할 수 있지만 wsl 사용하는 경우 wsl의 IP를 ifconfig에서 eth0 또는 eth1 인터페이스의 IP 주소를 확인 '<wsl ip>/8088' 형식으로 요청한다.

## 4. 컨테이너 삭제
```bash
docker container rm --force $(docker container ls --all --quiet)
```
## 5. 메모리 사용확인
```bash
docker container stats
```

## 6. 연습 문제
#### index.html 파일 교체하기
#### 1. 예제 이미지 실행
```bash
docker container run --detach --publish 8088:80 diamol/ch02-hello-diamol-web

```
#### 2. 새로운 index.html이 있는 위치로 이동
```bash
cd 호스트내 경로
```

#### 3. index.html 복사
```bash
docker container cp index.html <컨테이너 아이디 앞자리 3개>:/usr/local/apache2/htdocs/index.html
```

#### 만약 컨테이너 내 index.html 이 정확히 있는지 확인하고 싶으면 ls로 경로 확인
```bash
docker container exec <컨테이너 아이디 앞자리 3개>ls /usr/local/apache2/htdocs
```
