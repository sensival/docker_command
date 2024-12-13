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

# 컨테이너 상태 확인: 컨테이너 상태는 여전히 Up으로 나옴
docker container ls


```