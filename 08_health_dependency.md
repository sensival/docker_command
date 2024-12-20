## 1. 헬스체크를 지원하는 도커이미지 빌드하기

```bash
## 헬스체크 로직이 없는 상태에서 생기는 문제

# API 컨테이너를 실행한다
docker container run -d -p 8080:80 diamol/ch08-numbers-api

# API를 세번 호출한다 -각 호출마다 무작위 숫자 반환
curl http://localhost:8080/rng
curl http://localhost:8080/rng
curl http://localhost:8080/rng

# 네 번째부터 호출이 실패한다
curl http://localhost:8080/rng

# 컨테이너 상태 확인: 컨테이너 상태는 여전히 Up으로 나옴, 도커는 프로세스 상태만 확인하므로 비정상 감지 못함
docker container ls

```
```dockerfile
FROM diamol/dotnet-aspnet

ENTRYPOINT ["dotnet", "/app/Numbers.Api.dll"]
# 헬스 체크시에는 /health로 요청을 보내는데 응답은 애플리케이션의 정상여부, --fail은 curl이 전달받은 상태를 도커에 전달
HEALTHCHECK CMD curl --fail http://localhost/health

WORKDIR /app
COPY --from=builder /out/ .

```
```bash
docker image build -t diamol/ch08-numbers-api:v2 -f ./numbers-api/Dockerfile.v2 .
docker container run -d -p 8081:80 diamol/ch08-numbers-api:v2

#  30 초 뒤 Up .... (healthy) 로 나옴
docker container ls

# API를 세번 호출한다 -각 호출마다 무작위 숫자 반환
 curl http://localhost:8081/rng
 curl http://localhost:8081/rng
 curl http://localhost:8081/rng
 curl http://localhost:8081/rng

#  90 초 뒤 Up ....(unhealthy)  로 나옴
docker container ls
```
```bash
# 헬스체크 로그 확인
docker container inspect $(docker container ls --last 1 --format '{{.ID}}')
>>
 "Health": {
                "Status": "unhealthy",
                "FailingStreak": 14,
                "Log": [
                    {
                        "Start": "2025-01-21T21:09:49.836556588+09:00",
                        "End": "2025-01-21T21:09:49.873153188+09:00",
                        "ExitCode": 22,
                        "Output": "  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current\n                                 Dload  Upload   Total   Spent    Left  Speed\n\r  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0\r  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0\ncurl: (22) The requested URL returned error: 500 Internal Server Error\n"
                    },

```
## 2. 디펜던시 체크가 적용된 컨테이너 실행하기
```bash
# 현재 컨테이너 지우고 새로 시작하기
docker container rm --force $(docker container ls --all --quiet)

docker container run -d -p 8082:80 diamol/ch08-numbers-web

docker container ls

# 웹브라우저에서 접속(호스트 컴퓨터로 접속하려면  ip addr show eth0 확인)
```
```dockerfile
FROM diamol/dotnet-aspnet

ENV RngApi:Url=http://numbers-api/rng

## && 앞에오는 명령이 성공하면 뒤에 있는 명령어 실행
CMD curl --fail http://numbers-api/rng && \
    dotnet Numbers.Web.dll

WORKDIR /app
COPY --from=builder /out/ .
```
```bash
# 디펜던시 체크에 실패하면 종료되는 컨테이너
docker container run -d -p 8084:80 diamol/ch08-numbers-web:v2

#   Exited (6) 16 seconds ago  
docker container ls --all

```

## 3. 애플리케이션 체크를 위한 커스텀 유틸리티 만들기

```dockerfile
# http 테스트 유틸리티가 적용된 빌드 과정
FROM diamol/dotnet-aspnet

ENTRYPOINT ["dotnet", "Numbers.Api.dll"]
HEALTHCHECK CMD ["dotnet", "Utilities.HttpCheck.dll", "-u", "http://localhost/health"]

WORKDIR /app
COPY --from=http-check-builder /out/ .
COPY --from=builder /out/ .
```
```bash
docker container rm --force $(docker container ls --all --quiet)

docker container run -d -p 8082:80 diamol/ch08-numbers-web

docker container ls

docker container run -d -p 8080:80 --health-interval 5s diamol/ch08-numbers-api:v3

docker container ls

curl http://localhost:8080/rng
curl http://localhost:8080/rng
curl http://localhost:8080/rng
curl http://localhost:8080/rng

docker container ls
```
```dockerfile
# 디펜던시 체크에도 커스텀 유틸리티 사용하기
FROM diamol/dotnet-aspnet

ENV RngApi__Url=http://numbers-api/rng

# -t 유틸리티가 요청에 대한 응답을 기다릴 제한 시간 -c 옵션은 애플리케이션과 같은 설정 파일을 읽어 그 설정대로 URL을 지정
CMD dotnet Utilities.HttpCheck.dll -c RngApi:Url -t 900 && \
    dotnet Numbers.Web.dll

WORKDIR /app
COPY --from=http-check-builder /out/ .
COPY --from=builder /out/ .
```

## 4.