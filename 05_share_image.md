## 1. 도커 허브 로그인
```bash
# 환경 변수로 내 아이디 설정
export dockerId="alstjs883"

docker login --username $dockerId
# 안되면 ~/.docker/config.json에서 ,"credsStore": "secretservice" 이부분 삭제
```
## 2. 레지스트리에 푸쉬 
```bash
# 이미지에 참조/태그 부여
docker image tag image-gallery $dockerId/image-gallery:v1

# 참조 목록 확인
docker image ls --filter reference=image-gallery --filter reference='*/image-gallery'

# 레지스트리에 푸쉬
docker image push $dockerId/image-gallery:v1

# 도커허브레 푸쉬된 웹페이지 url 출력
echo "https://hub.docker.com/r/$dockerId/image-gallery/tags"
```
## 나만의 로컬 레지스트리 만들기 
```bash
docker container run -d -p 5000:5000 --restart always diamol/registry

# 로컬 컴퓨터에 대한 네트워크 별명 설정
echo $'\n127.0.0.1 registry.local' | sudo tee -a /etc/hosts
ping registry.local

docker image tag image-gallery registry.local:5000/gallery/ui:v1
docker image tag access-log registry.local:5000/gallery/logs:v1
docker image tag image-of-the-day registry.local:5000/gallery/api:v1

# 로컬 레지스트리에 push
docker image push registry.local:5000/gallery/ui:v1
 ```

 
[Registry v2 API docs](https://docs.docker.com/registry/spec/api)

> If you're using PowerShell on Windows, the `curl` command is an alias for `Invoke-WebRequest`, which has different behaviour from the real cURL. Recent versions of Windows 10 have cURL available as `curl.exe`, so I'd recommend using that instead (just add `.exe` to the commands below).

## Push

```
docker image push registry.local:5000/gallery/ui
```

> Pushes all tags for the repo

## Check 

```
curl http://registry.local:5000/v2/gallery/ui/tags/list
```

## Get manifest for `latest`

```
curl --head \
  http://registry.local:5000/v2/gallery/ui/manifests/latest \
  -H 'Accept: application/vnd.docker.distribution.manifest.v2+json'
```
> Output headers include `Docker-Content-Digest`, this is the manifest you need

e.g. 

```
Docker-Content-Digest: sha256:127d0ed6f7a8d148a39b7ea168c083694d030e2bffbda60cb53057e731114fbb
```

## Delete

```
curl -X DELETE \
  http://registry.local:5000/v2/gallery/ui/manifests/sha256:127d0ed6f7a8d148a39b7ea168c083694d030e2bffbda60cb53057e731114fbb
```

## Check 

```
curl http://registry.local:5000/v2/gallery/ui/tags/list