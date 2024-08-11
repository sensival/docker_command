## 1. 멀티 스테이지 빌드 토커 스크립트/명령어
```bash
FROM diamol/base AS build-stage
RUN echo 'Building...' > /build.txt

FROM diamol/base AS test-stage
COPY --from=build-stage /build.txt /build.txt
RUN echo 'Testing...' >> /build.txt

FROM diamol/base
COPY --from=test-stage /build.txt /build.txt
CMD cat /build.txt
```
```bash
 docker image build -t multi-stage .
 docker container run multi-stage
 ```

## 2. 자바 소스코드 빌드
```bash
FROM diamol/maven AS builder # jdk와 메이븐 포함

WORKDIR /usr/src/iotd
COPY pom.xml .# pom.xml 복사
RUN mvn -B dependency:go-offline # 의존 모듈내려받기

COPY . . # 나머지 소스코드를 현재 호스트 dir에서 이미지 dir로
RUN mvn package # 빌드하고 패키징

# app
FROM diamol/openjdk #jdk 만 있는 이미지

WORKDIR /app
COPY --from=builder /usr/src/iotd/target/iotd-service-0.1.0.jar . # builder 단계에서 만든 jar 복사

EXPOSE 80 # 외부에 공개하는 포트
ENTRYPOINT ["java", "-jar", "/app/iotd-service-0.1.0.jar"] # jar 실행
```
```bash
docker image build -t image-of-the-day .

docker network create nat 
# 컨테이너 간 통신에 사용되는 도커 네트워크 생성

docker container run --name iotd -d -p 800:80 --network nat image-of-the-day
# http://<wsl IP>:800/image로 접근
```

## 3.Node.js 소스코드 빌드
```bash
FROM diamol/node AS builder # node.js 런타임과 npm포함

WORKDIR /src
COPY src/package.json . # 패키지 복사
RUN npm install

# app
FROM diamol/node

EXPOSE 80
CMD ["node", "server.js"]

WORKDIR /app
COPY --from=builder /src/node_modules/ /app/node_modules/
COPY src/ .
```
```bash
docker image build -t access-log .

docker network create nat 
# 컨테이너 간 통신에 사용되는 도커 네트워크 생성

docker container run --name accesslog -d -p 801:80 --network nat  access-log
```
## 4. Go 소스코드 빌드
```bash
FROM diamol/golang AS builder

COPY main.go .
RUN go build -o /server

# app
FROM diamol/base
ENV IMAGE_API_URL="http://iotd/image" \
    ACCESS_API_URL="http://accesslog/access-log"

CMD ["/web/server"]

WORKDIR /web
COPY index.html .
COPY --from=builder /server .
RUN chmod +x server
```
```bash
docker image build -t image-gallery .

docker image ls -f reference=diamol/golang -f reference=image-gallery
# Go 빌드도구와 어플리케이션 크기비교

docker container run -d -p 802:80 --network nat image-gallery

```

## 5. 연습 문제
html 파일을 수정하더라도 다시 수행되는 빌드 단계가 7단계가 되게 하라
```bash
# 수정 전
FROM diamol/golang 

WORKDIR web
COPY index.html .
COPY main.go .

RUN go build -o /web/server
RUN chmod +x /web/server

CMD ["/web/server"]
ENV USER=sixeyed
EXPOSE 80
```

```bash
# 수정 후
FROM diamol/golang AS builder

COPY main.go .
RUN go build -o /server
RUN chmod +x /server

# app
FROM diamol/base

EXPOSE 80
CMD ["/web/server"]
ENV USER="sixeyed"

WORKDIR web
COPY --from=builder /server .
COPY index.html .
```