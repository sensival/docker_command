## 1. 도커 컴포즈 파일
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

## 2. 컴포즈 
```bash
 docker-compose up