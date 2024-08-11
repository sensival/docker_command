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