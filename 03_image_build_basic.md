## 1. 도커이미지 내려받기
```bash
docker image pull diamol/ch03-web-ping
```

## 2.이미지 실행
```bash
docker container run -d --name web-ping diamol/ch03-web-ping
# -d: detach, 백그라운드 실행 --name: CONTAINER ID 외에 별칭 지정

docker container logs web-ping
# 로그확인

docker rm -f web-ping
# 종료
```

## 3. 환경변수 바꾸기
```bash
docker container run --env TARGET=google.com diamol/ch03-web-ping
# 구글로 ping을 보내도록 설정, 이것은 이미지 작성자가 작성시 변경 가능하도록 설정하는 것
```

## 4. 도커이미지 build
#### 도커 스크립트 준비
```dockerfile
FROM diamol/node

ENV TARGET="blog.sixeyed.com"
ENV METHOD="HEAD"
ENV INTERVAL="3000"

WORKDIR /web-ping
COPY app.js .

CMD ["node", "/web-ping/app.js"]
```
#### js 파일 준비
```javascript
const https = require("https");

const options = {
  hostname: process.env.TARGET,
  method: process.env.METHOD
};

console.log(
  "** web-ping ** Pinging: %s; method: %s; %dms intervals",
  options.hostname,
  options.method,
  process.env.INTERVAL
);

process.on("SIGINT", function() {
  process.exit();
});

let i = 1;
let start = new Date().getTime();
setInterval(() => {
  start = new Date().getTime();
  console.log("Making request number: %d; at %d", i++, start);
  var req = https.request(options, res => {
    var end = new Date().getTime();
    var duration = end - start;
    console.log(
      "Got response status: %s at %d; duration: %dms",
      res.statusCode,
      end,
      duration
    );
  });
  req.on("error", e => {
    console.error(e);
  });
  req.end();
}, process.env.INTERVAL);
```
#### 해당 디렉토리에서 명령어 실행
```bash
 docker image build --tag web-ping .
 # --tag:이미지 이름 .: 현재 디렉토리 
```
#### 빌드된 이미지 확인
```bash
docker image ls 'w*'
```

## 5. build된 이미지 실행
```bash
docker container run -e TARGET=docker.com -e INTERVAL=5000 web-ping
```
## 6. web-ping 이미지 히스토리 확인하기
```bash
docker image history web-ping
```
## 7. 이미지 용량
```bash
docker image ls
# 여기나오는 SIZE 보다 실제 용량이 작을 수 있다. 다른 이미지와 공유하는 레이어가 있다면 실제 차지하는 용량이 더 적다

docker system df
# ls에서 나온 용량 총합-system df에서 나온 용량 = 이미지끼리 공유하는 공간
```
## 8. Dockerfile 스크립트 최적화
만약, 코드를 한 줄 수정 후 빌드를 다시 빌드할 경우 코드와 관련된 step 이후의 스텝은 변화가 없어도 캐시에서 재사용이 불가능하다.

따라서 도커 스크립트에서 자주 수정되는 인스트럭션은 뒤로 보내는 것이 좋다(ex. COPY app.js) 

## 9. 연습 문제
도커스크립트 없이 도커 빌드하기
```bash
docker container run -it --name ch03lab diamol/ch03-lab

echo sensival >> ch03.txt 

exit

docker container commit ch03lab ch03-lab-soln
      
docker container run ch03-lab-soln cat ch03.txt
```
