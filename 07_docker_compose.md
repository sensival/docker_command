## 1. 도커 컴포즈 YAML파일
```bash
version: '3.7'

services:
  
  todo-web:
    image: diamol/ch06-todo-list
    ports:
      - "8020:80"
    networks:
      - app-net

networks:
  app-net:
    external:
      name: nat
```
#### 도커 컴포즈 파일의 최상위 문(statement)
* version: 이 파일에 사용된 파일 형식의 버젼
* services: 애플리케이션을 구성하는 모든 컴포넌트를 열거
* networks: 서비스 컨테이너가 연결될 모든 도커네트워크를 열거하는 부분

## 2. 컴포즈 명령으로 실행
```bash
docker network create nat

cd /080258/ch07/exercises/todo-list

docker-compose up
```
## 3. 여러개의 컨테이너로 구성된 이미지 갤러리를 기술한 컴포즈 스크립트
```bash
version: '3.7'

services:

  accesslog:
    image: diamol/ch04-access-log
    networks:
      - app-net

  iotd:
    image: diamol/ch04-image-of-the-day
    ports:
      - "80"
    networks:
      - app-net

  image-gallery:
    image: diamol/ch04-image-gallery
    ports:
      - "8010:80" 
    depends_on:
      - accesslog
      - iotd
    networks:
      - app-net

networks:
  app-net:
    external:
      name: nat
```
```bash
cd /080258/ch07/exercises/image-gallery

docker-compose up --detach

# 여러개의 컨테이너가 결합한 컨테이너 중 일부 컨테이너의 갯수 늘리기
docker-compose up -d --scale iotd=3
```
```bash
# 멈췄다 재시작하기
docker-compose stop

docker-compose start

docker container ls
```
```bash
# 지웠다가 재시작하기
docker-compose down

docker-compose up -d

docker container ls
```
## 4. 컨테이너 간 통신
```bash
docker compose up -d --iotd=3

docker container exec -it image-of-the-day_image-gallery_1 sh

nslookup accesslog #해당 컨테이너의 주소 출력
```
