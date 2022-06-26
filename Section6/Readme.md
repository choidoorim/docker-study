# Section 6: Docker Compose 우아한 다중 컨테이너 오케스트레이션
# 모듈 소개
컨테이너를 실행하기 위한 명령들은 기본적으로 매우 길다는 단점이 있다.
여러 개의 컨테이너를 실행시키기 위해서는 각각 컨테이너 별로 명령을 수행해야 한다.     
예를 들면 네트워크를 생성하고, 데이터베이스 컨테이너, 백엔드 컨테이너, 프론트 컨테이너를 각각 실행하는 것처럼 말이다.

이것이 도커 생태계에서 도커 컴포즈(Docker Compose) 라 불리는 도구를 제공하는 이유이다.
다중 컨테이너 설정을 더 쉽게 관리할 수 있게 해주며 시간을 절약할 수 있다는 것이다. 설정 프로세스를 자동화하는데 도움이 된다.

다수의 도커 명령들을 단 하나의 구성 파일로 가진다. 이 파일로 모든 서비스 모든 컨테이너를 즉시 시작하고 
필요하다면 모든 필요한 이미지를 빌드하는 자동화 명령 집합이다.
즉, 구성 파일를 가진 하나의 명령으로 다중 컨테이너 어플리케이션을 시작하거나 중단할 수 있다.    
단일 컨테이너에서도 사용할 수 있지만, 컨테이너가 여러 개인 경우에 가장 유용하다.

도커 컴포즈는 커스텀 이미지를 위한 Dockerfiles 을 대체하지 않는다.
또한 이미지나 컨테이너를 대체하지도 않는다. 단지 컨테이너를 더 쉽게 시작할 수 있게 한다.

그리고 중요한 점은 도커 컴포즈는 다수의 호스트에서 다중 컨테이너를 관리하는데는 적합하지 않다.
도커 컴포즈는 **하나의 동일한 호스트에서 다중 컨테이너를 관리**하는데 좋다.
현실에서 개발용으로 많이 사용되며 개발 외의 영역에서도 사용된다.

서비스라고 하면 실제로 컨테이너를 의미한다. 게시해야 하는 포트를 정의하고 이 컨테이너에 필요한 환경 변수를 정의할 수 있고,
컨테이너에 할당해야 하는 볼륨, 네트워크 등을 정의하고 할당할 수 있다.

# Compose 파일 만들기
프로젝트 폴더에 ```docker-compose.yml``` 파일을 만들어준다. 다중 컨테이너 환경, 프로젝트 설정 등을 기재한다.

```version``` 은 우리 앱이나 파일의 버전이 아닌 Docker Compose 사양의 버전을 지정하는 것이다.
여기에서 정의한 버전은 컴포즈 파일에 사용할 수 있는 기능에 영향을 미친다.
버전을 정의해줌으로써 도커는 어떤 기능을 사용할 수 있고 어떤 것을 사용할 수 없는지 알게되는 것이다.

```services``` 는 몇가지의 중첩된 값을 가진다. 예를 들면 3 개의 컨테이너가 있을 경우 services 아래에 3 개의 하위 요소를 가진다.
원하는 이름을 가지고 컨테이너에 이름을 지정할 수 있다. yaml 파일을 들여쓰기를 통해 종속성을 판단하기에 유의하며 작성해야 한다.

# Compose 파일 구성(configuration) 자세히 알아보기
```
docker run --name mongodb \
  -e MONGO_INITDB_ROOT_USERNAME=max \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  -v data:/data/db \
  --rm \
  -d \
  --network goals-net \
  mongo
```
위의 MongoDB 컨테이너를 실행시키는 명령을 Docker Compose 로 바꾸기 위해서는 mongodb 에서 **사용하려는 이미지를 가리키는 것으로 시작**한다.
작은 따옴표('')나 큰따옴표("") 사이에 mongo 라는 이미지 이름을 지정한다. 
```
#...
services:
  mongodb:
    image: 'mongo'
```
따라서 도커는 결국 mongodb 컨테이너가 ```'mongo''``` 라는 이미지를 기반으로 해야 한다는 것을 알린다.
단순히 이미지명으로 로컬이나 도커 허브 저장소에서 조회된다. 이미지명을 사용해도되지만 이미지를 가리키는 주소, URL 이 될 수도 있다.

```-d``` 라는 Detached 모드를 명시해줄 필요는 없다. 도커 컴포즈를 사용하면 Detached 모드가 Default 이다.
```-rm``` 옵션과 ```-d``` 옵션은 추가하지 않아도 된다.

volume 을 추가하기 위해서는 volumes 아래에 ```'-'``` 을 통해 모든 볼륨을 추가한다.
```
#...
services:
  mongodb:
    image: 'mongo'
    volumes:
      - data:/data/db
      - ...
```
이 구문은 컨테이너 실행 시 ```-v``` 플래그와 함께 사용한 것과 정확히 일치한다.
그리고 콜론을 추가하여 읽기 전용(ro)과 같은 부가 옵션을 추가할 수도 있다. ```- data:/data/db:ro```

환경 변수는 ```environment``` 옵션을 images, volumes 와 동일한 들여쓰기 수준으로 추가한다.
```
# 1. key: value
# 2. - key = value
services:
  mongodb:
    image: 'mongo'
    volumes:
      - data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: max
      - MONGO_INITDB_ROOT_PASSWORD=secret
```
위의 2 가지 형태로 사용할 수 있다.

별도의 파일로 환경 변수를 가지게 된다면 ```env_file``` 옵션에 해당 컨테이너에 사용해야 하는 모든 환경 파일을 목록에 추가할 수 있다.
```
# env/mongo.env
services:
  env_files:
    - ./env/mongo.env
```
이렇게 하면 이 파일에 저장된 환경 변수를 읽고, 이 컨테이너에서 그 적용사항이 고려된다. 이것이 환경 변수를 파일로 추가하는 방법이다.

yaml 파일에서 ```key: value``` 형태일 경우 yaml 객체를 생성하기에 '-' 가 필요 없다.
즉, 콜론과 공백이 없는 단일 값의 경우 '-' 가 필요하다.

```networks``` 키를 추가하여 네트워크를 추가할 수 있으며 이 컨테이너가 속해야 하는 모든 네트워크를 특정할 수 있다.
하지만 도커 컴포즈를 사용한다면 필요없다. 도커가 컴포즈 파일에 특정된 모든 서비스에 대해 새 환경을 자동으로 생성하고 모든 서비스를 즉시 그 네트워크에 추가하기 때문이다.     
따라서 하나의 동일한 컴포즈 파일에 정의된 모든 서비스는 이미 도커에 의해 생성된 동일한 네트워크의 일부가 된다.      
```
services:
  networks
    - network_name
```
만약 네트워크를 사용해야 한다면 위와 같이 networks 하위에 네트워크 이름을 쓰면 된다.

services 와 동등한 들여쓰기 수준으로 volumes 키를 추가해야 한다. **services 에서 사용 중인 Named Volume 이 나열**되어야 한다.
```
volumes:
  data:
```
위와 같은 형태가 이상할 수는 있지만 이것은 도커가 services 를 위해 생성해야 하는 Named Volume 을 인식하기 위해 필요한 구문이다.
따라서 다른 컨테이너가 호스팅 머신 상의 동일한 볼륨 동일한 폴더를 사용할 수도 있다.
**Named Volume 만 지정하면 되고, 익명 볼륨과 바인드 마운트는 여기에 지정할 필요가 없다**.

# Linux 에 Docker Compose 설치하기
maxOS 나 Window 에서는 Docker 와 함께 설정되기에 Docker Compose 가 미리 설치되었을 것이다.
하지만 Linux 시스템에서는 별도로 설치해야 한다. [링크](https://docs.docker.com/compose/install/)
```
1. $ sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
2. $ sudo chmod +x /usr/local/bin/docker-compose
3. $ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
4. verify: $ docker-compose --version
```
