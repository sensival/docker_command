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
docker compose up -d --scale iotd=3

docker container exec -it image-of-the-day-image-gallery-1 sh

/web # 
nslookup accesslog #해당 컨테이너의 주소 출력
```
```bash
docker container rm -f image-of-the-day_accesslog_1

docker compose up -d --scale iotd=3

docker container exec -it image-of-the-day-image-gallery-1 sh

/web # 
nslookup accesslog

/web # 
nslookup iotd # 여러번 실행하면 매번 순서가 바뀌는 것을 볼 수 있음 -> 로드 밸런싱

/web # 
exit
```
## 5. 도커 컴포즈로 애플리케이션 설정값 지정하기
```bash
#  데이터베이스 SQLite -> PostgreSQL로 변경
version: "3.7"

services:
  todo-db:
    image: diamol/postgres:11.5
    ports:
      - "5433:5432"
    networks:
      - app-net

  todo-web:
    image: diamol/ch06-todo-list
    ports:
      - "8030:80"
    environment:
      - Database:Provider=Postgres
    depends_on:
      - todo-db
    networks:
      - app-net
    secrets:
      - source: postgres-connection
        target: /app/config/secrets.json

networks:
  app-net:

secrets:
  postgres-connection:
    file: ./config/secrets.json

```
```bash
# 디렉토리 이동
docker compose up -d 

docker-compose ps
```
## 6. 연습문제
1. 호스트 컴퓨터가 재부팅되거나 도커 엔진이 재시작되면 컨테이너도 재시작되게 하라 <br>
2. 데이터베이스 컨테이너는 바인드 마운트에 파일을 저장행 애플리케이션을 재시작하더라도 데이터를 유지할 수 있도록 하라 <br>
3. 테스트를 위해 웹어플리케이션을 80번 포트를 주시하게 하자
* 참고 :https://docs.docker.com/compose/compose-file
```bash
version: "3.7"

services:
  todo-db:
    image: diamol/postgres:11.5
    restart: unless-stopped
    environment:
      - PGDATA=/var/lib/postgresql/data
    volumes:
      - type: bind
        source: /data/postgres
        target: /var/lib/postgresql/data
    networks:
      - app-net

  todo-web:
    image: diamol/ch06-todo-list
    restart: unless-stopped
    ports:
      - "8050:80"
    environment:
      - Database:Provider=Postgres
    depends_on:
      - todo-db
    secrets:
      - source: postgres-connection
        target: /app/config/secrets.json

secrets:
  postgres-connection:
    file: postgres-connection.json

networks:
  app-net:
    external:
      name: nat

```
```
sudo mkdir -p /data/postgres

docker-compose -f docker-compose-test.yml up -d
```