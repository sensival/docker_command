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