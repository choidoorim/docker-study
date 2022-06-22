# Section5: Docker 로 다중 컨테이너 어플리케이션 구축하기
# 모듈 소개
함께 작동하는 여러 서비스로 구성된 어플리케이션을 자주 사용하게 될 것이다. 
예를 들면 데이터베이스, 백엔드 서버, 프론트엔드 어플리케이션 등이 있다.

# Target 앱 & 설정
빌딩 블록을 도커화하는 것에 대해 생각할 때 중요하게 생각해야 하는 것이 있다.
Database 를 컨테이너에 넣으면 당연히 데이터가 지속되어야 한다. Database 컨테이너를 제거했다가 다시 생성해도 손실되지 않아야 한다.

또한 백엔드 서버에서도 로그 파일 등이 지속해서 손실되지 않고 유지되어야 한다.
그리고 호스트 머신에 있는 소스코드를 업데이터할 경우 이미지를 재빌드해서 컨테이너를 재시작하지 않고도 컨테이너 어플리케이션에서 항상 즉시 사용할 수 있어야 한다.

# MongoDB 서비스 도커화 하기
```$ docker run mongo``` 명령을 통해 Docker Hub 에 등록된 공식 이미지를 사용하여 컨테이너를 실행시킬 수 있다.
```-p``` 옵션을 통해 컨테이너에서 외부의 머신으로 포트를 노출시켜야 한다.

```
$ docker run --name mongodb --rm -d -p 27017:27017 mongo
```
이렇게 여러 옵션들을 사용해서 Mongo DB 컨테이너를 실행시킬 수 있다.

# Node 앱 도커화 하기
만약 로컬에서 컨테이너화 되어 있는 DB 에 접근하기 위해서는 도메인을 ```localhost``` 가 아닌 ```host.docker.internal``` 로 변경해줘야 한다.

```$ docker run``` 명령 실행 시 포트를 노출시키지 않는다면 외부에서 컨테이너에 요청을 하지 못하는 경우가 발생하므로 ```-p``` 옵션을 통해 노출시켜줘야 한다.

# React SPA 컨테이너로 옮기기
백엔드 서버와 다를 것이 없다. 프론트도 동일하게 React 어플리케이션을 호스팅하는 개발 서버를 가동하게 된다.
그리고 기본적으로 포트 3000 을 노출한다.

# 효율적인 컨테이너 간 통신을 위한 Docker 네트워크 추가하기
컨테이너가 동일한 네트워크에 있다면 굳이 ```$ docker run``` 명령 시에 포트를 노출시킬 필요가 없다. 
따라서 ```-p``` 옵션 대신에 ```--network``` 옵션을 사용해서 하나의 네트워크에 여러 컨테이너를 추가해서 통신할 수 있게 한다.

백엔드 어플리케이션에서는 Localhost 디비가 아닌 Network 를 사용하기 때문에 **DB 주소 대신에 DB 컨테이너 명**으로 변경해야 한다.
```
host.docker.internal -> mongodb(DB Container Name)
```
그리고 백엔드 어플리케이션과 프론트 어플리케이션 간의 통신을 하기 위해서 백엔드 컨테이너 실행 시 ```-p``` 옵션을 통해 포트를 노출시켜줘야 한다.

프론트 어플리케이션에서는 Backend 주소가 Network 주소이기에 만약 localhost 가 주소라면 백엔드 컨테이너 명으로 변경해야 할까?
하지만 **프론트는 컨테이너 내부에서 작동하는 것이 아닌 브라우저에서 작동**하기에 ```goals-backend``` 라는 백엔드 컨테이너가 무엇이 되어야 하는지 모르게 된다.
즉, 프론트의 코드는 **도커 컨테이너에서 실행되지 않고 브라우저에서 실행**된다.    
즉, React 프론트 어플리케이션에는 도커 컨테이너 내부에서 실행되는 Javascript 코드가 아닌 브라우저에서 실행되는 Javascript 코드가 있다.

# 볼륨으로 MongoDB 에 데이터 지속성 추가하기
DB 컨테이너를 삭제 후 재실행하게 된다면, 기존 데이터가 모두 사라지게 된다. 컨테이너에 저장된 모든 데이터가 손실되고 데이터베이스에 저장된 모든 데이터가 손실된다.
결국에는 여전히 데이터들은 하드 드라이브에 기록되어야 한다. 볼륨을 추가하면 해결이 가능하다.

Mongo DB 인 경우 내부 데이터가 존재하는 경로를 직접적으로 알 수는 없지만 [공식홈페이지](https://hub.docker.com/_/mongo)에 간다면 확인이 가능하다.    
데이터들이 삭제되는 문제는 **Named Volume 을 사용하면 해결이 가능**하다. 
```docker run -v data:/data/db ...```명령을 통해 컨테이너 내부의 /data/db 데이터를 ```data``` 라는 호스트 머신의 볼륨에 저장한다.
만약 기존에 있는 볼륨일 경우 도커는 호스트 머신의 폴더에 존재하는 데이터를 컨테이너 폴더로 로드한다. 이렇게 된다면 데이터가 손실되지 않는다.

# NodeJS 컨테이너의 볼륨, 바인딩 마운트 및 폴리싱(Polishing)
Named Volume 을 통해 컨테이너에서 소멸된 데이터가 살아남을 수 있도록 할 수 있다.
그리고 바인드 마운팅을 통해 호스팅 머신 내부에서 로그 파일을 읽을 수 있다.
```
$  docker run --name goals-backend -v "/Users/choidoorim/Desktop/Docker 강의/Source/multi-01-starting-setup/backend":/app -v logs:/app/logs -v /app/node_modules --rm -d -p 80:80 --network goals-net goals-node
```
컨테이너를 실행하는 명령에 볼륨을 추가해서 로그 데이터가 유지되고, 기존 컨테이너 내부의 ```node_modules``` 폴더는 덮어씌워지지 않도록 했다.

바인드 마운팅을 통해서 기존 호스트 머신의 코드가 컨테이너에 반영될 수 있도록 했지만, Node 서버는 ```node app.js``` 명령이 실행되는 시점에 코드가 반영이 되기 때문에 
이미 실행 중인 노드 서버에는 영향을 미치지 않는다.     
하지만 코드 변경 시 노드 서버가 다시 시작되는 것을 원할 것이다. 그것은 nodemon 패키지를 사용하면 해결할 수 있다. 

만약 하드코딩된 부분이 있다면 Dockerfile 에서 ENV 명령을 통해 개선할 수 있다.
```dockerfile
# ...
ENV MONGODB_USERNAME=root
ENV MONGODB_PASSWORD=password
```
```javascript
mongoose.connect(
  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals`,
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
//...
```
만약 컨테이너 실행 시에 ```-e``` 옵션을 넣지 않는다면 디폴트인 root 와 password 가 사용될 것이다.
```
$ docker run -e MONGODB_USERNAME=max ...
```

현재 Dockerfile 덕분에 모든 종속성이 설치된 이후에 백엔드 폴더의 모든 것을 컨테이너로 복사했다.
하지만 컨테이너에 복사하는 파일 중에 복사하지 않고 싶은 것이 있을 수도 있다.
이를 위해서는 ```.dockerignore``` 파일을 추가하면 된다.    
대표적으로 ```node_modules``` 는 컨테이너 내부에 이미 설치한 모든 종속성을 불필요하게 다시 복사하지 않도록 하기 위해서 제외한다.
