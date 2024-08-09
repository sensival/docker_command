## 1. 컨테이너 안에 파일 쓰기
```bash
# 동일한 컨테이너 2개 실행 후 컨테이너 내부에 txt 파일 호스트로 복사
# 컨테이너의 파일 시스템은 읽기 전용 레이어와 쓰기 가능 레이어로 이루어짐
docker container run --name rn1 diamol/ch06-random-number
docker container run --name rn2 diamol/ch06-random-number

docker container cp rn1:/random/number.txt number1.txt
docker container cp rn2:/random/number.txt number2.txt

cat number1.txt
cat number2.txt
```
```bash
# 로컬 컴퓨터의 파일을 컨테이너로 복사해 덮어쓰기
docker container run --name f1 diamol/ch06-file-display

echo "www.naver.com" > url.txt

docker container cp url.txt f1:/input.txt

docker container start --attach f1


# 동일한 이미지로 실행한 컨테이너이지만 새로 실행한 컨테이너에는 반영이 안됨
docker container run --name f2 diamol/ch06-file-display

docker container rm -f f1

docker container cp f1:/input.txt
```
## 2. 도커 스트립트로 볼륨 연결하기
```bash
FROM diamol/dotnet-sdk AS builder

WORKDIR /src
COPY src/ToDoList.csproj .
RUN dotnet restore

COPY src/ .
RUN dotnet publish -c Release -o /out ToDoList.csproj

# app image
FROM diamol/dotnet-aspnet

WORKDIR /app
ENTRYPOINT ["dotnet", "ToDoList.dll"]

# set in the base image - `/data` for Linux, `C:\data` for Windows
VOLUME $SQLITE_DATA_DIRECTORY

# set in the base image - `root` for Linux, `ContainerAdministrator` for Windows
USER $SQLITE_USER

COPY --from=builder /out/ .
```
```bash
# 이미지 빌드
 docker image build --tag todo-list .
 ```
```bash
docker container run --name todo -d -p 8010:80 diamol/ch06-todo-list

docker container inspect --format '{{.Mounts}}' todo

docker volume ls

# 로컬에서 확인하려면 volume 명령에서 나온 경로를 sudo로 실행
sudo ls /var/lib/docker/volumes/
```
## 3. 서로 다른 컨테이너가 같은 스토리지 사용
```bash
# 볼륨 생성하는 컨테이너
docker container run --name  todo2 -d diamol/ch06-todo-list

# 여기는 아무것도 없음
docker container exec todo2 ls /data

# 이건 볼륨 연결
docker container run -d --name t3 --volumes-from todo diamol/ch06-todo-list

# 이건 /data에 todo-list에서 만든 볼륨 연결
docker container exec t3 ls /data
```

## 4. 같은 볼륨을 사용하는 컨테이너 실습
```bash
target='/data'

# 저장할 볼륨 생성
docker volume create todo-list

# v1 애플리케이션 실행 후 add a item 몇 개 추가하기
docker container run -d -p 8011:80 -v todo-list/$target --name todo-v1 diamol/ch06-todo-list

# v1 컨테이너 삭제
docker container rm -f todo-v1

# 같은 볼륨을 사용하는 v2 실행
docker container run -d -p 8011:80 -v todo-list:$target --name todo-v2 diamol/ch06-todo-list:v2
```
## 5. 호스트 컴퓨터의 파일/디렉토리를 사용하기(바인드 마운트)
```bash
# 환경 변수 설정
source="$(pwd)/databases" && target='/data'

# 디렉토리 생성
mkdir ./databases

# 디렉토리 지정하여 컨테이너 실행
docker container run --mount type=bind,source=$source,target=$target -d -p 8012:80 diamol/ch06-todo-list

# URL에서 데이터 다운로드
curl http://localhost:8012

# todo-list.db 생성
ls ./databases
```
바인드 마운트는 컨테이너/호스트에서 쓰기 가능

## 6. 호스트 컴퓨터의 설정 파일을 사용
```bash
cd /080258/ch06/exercises/todo-list

# 경로 문자열을 환경 변수로 정의
source="$(pwd)/config" && target='/app/config'

# 바인드 마운트 적용하여 컨테이너 실행
docker container run --name togo-configured -d -p 8013:80 --mount type=bind,source=$source,target=$target,readonly diamol/ch06-todo-list

# 애플리케이션 작동여부 확인
curl http://localhost:8013

# 컨테이너 로그 확인
docker container logs togo-configured
```

## 7. 마운트하면 원래 이미지에 포함된 디렉토리는 사라짐
```bash 
cd /080258/ch06/exercises/bind-mount

source="$(pwd)/new" && target='/init'

docker container run diamol/ch06-bind-mount
>> abc.txt
   def.txt

# 바인드 마운트 적용하여 컨테이너 실행
docker container run --mount type=bind,source=$source,target=$target diamol/ch06-bind-mount
>> 123.txt
   456.txt
```

## 8. 단일 파일 마운트
```bash
docker container run diamol/ch06-bind-mount
>> abc.txt
   def.txt

docker container run --mount type=bind,source="$(pwd)/new/123.txt",target=/init/123.txt diamol/ch06-bind-mount
>> 123.txt
   abc.txt
   def.txt

```
## 9. 연습 문제
```bash
docker container rm -f $(docker container ls -aq)

docker container run -d -p 8015:80 diamol/ch06-lab

docker volume create ch06-lab

configSource="$(pwd)/solution"
configTarget='/app/config'
dataTarget='/new-data'

docker container run -d -p 8016:80 --mount type=bind,source=$configSource,target=$configTarget,readonly --volume ch06-lab:$dataTarget diamol/ch06-lab

```