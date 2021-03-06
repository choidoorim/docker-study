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

# Docker Compose Up 과 Down
Docker Compose 를 실행시키기 위해서는 Docker Compose 가 있는 파일로 이동해야 한다.    
```$ docker-compose up``` 하나의 명령으로 Compose 파일에서 찾을 수 있는 모든 서비스가 시작된다.
그리고 그것은 단지 컨테이너를 시작할 뿐만 아니라 필요한 모든 이미지를 가져와 빌드한다.

```$ docker ps -a``` 명령으로 확인해보면 컨테이너가 중지되어 있는 것을 확인할 수 있다.
만약 detached 모드에서 시작하고 싶다면 ```-d``` 플래그를 추가한, ```$ docker compose up -d``` 명령으로 실행하면 된다.

모든 서비스를 중지하고 모든 컨테이너 등을 제거하려면 ```$ docker-compose down``` 을 실행하면 된다.
관련된 모든 것이 삭제되지만 볼륨은 삭제되지 않늗다. 볼륨도 삭제하기 위해서는 ```-v``` 플래그를 추가하면 된다.
기본적으로 볼륨을 삭제하면서 서비스를 중단시키는 것은 좋지 않다.

```docker-compose up``` 은 모든 이미지를 가져와 컨테이너를 시작하는 것이고, ```docker-compose down``` 은 중지를 위한 것이다.

# 다중 컨테이너로 작업하기
기존에 이미지가 존재하지 않는다면 그것을 리빌드해서 사용할 수 있지만, Docker Compose 는 리빌드 단계를 대체할 수 있게 도와준다.
바로 ```build``` 옵션으로 이것을 수행한다. ```Dockerfile``` 이 찾을 수 있는 곳을 넣어주면 된다.
```
services:
  #...
  backend: 
    build: ./backend
```
이것이 빌드되면 이 컨테이너에 빌드된 이미지를 사용한다. 더 긴 형태로 build 옵션을 사용할 수도 있다.
```
services:
  #...
  backend:
    # 경로 
    context: ./backend
    # Dockerfile Name
    dockerfile: Dockerfile
```
context 옵션 내에 Dockerfile 을 보유하는 폴더의 경로를 특정한다.       
그리고 dockerfile 키 안에 파일 이름을 지정한다. 일반적으로는 Dockerfile 이다. 그러나 Dockerfile-dev 라던지 다른 이름을 사용한다면 
Docker Compose 에 사용할 Dockerfile 을 이런식으로 알릴 수 있다.        
따라서 backend 경로의 Dockerfile 만 사용할 수 있게 된다.

게시된 포트를 지정할 수도 있다. ```ports``` 옵션을 사용하면 된다.
```
services:
  #...
  backend:
    # ...
    ports:
      - '3000:80'
```
외부, 내부 포트를 대쉬(-) 를 통해 지정한다.

볼륨은 ```volumes``` 옵션을 사용하면 된다. 그리고 대쉬(-) 를 통해 추가해주면 된다. 
만약 Named Volume 일 경우 services 와 같은 위치에 추가해줘야 한다.
```
services:
# ...
volumes
  - data:
  - logs:
```
기존에는 바인트 마운트 시에 절대 경로를 사용했지만, Docker Compose 를 사용한다면 바인드 마운트 같은 경우에는 상대경로를 사용할 수 있다.
예를 들면 전체 폴더를 공유하고자하면 그 폴더를 지정하면 된다.
```
services:
  volumes:
    # /Users/maximilianschwarzmuller/development/teaching/udemy/docker-complete/backend:/app (X)
    - ./backend:/app
```
```...docker-complete/backend``` 에서도 backend 폴더를 공유하고 있으므로 ```./backend:/app``` 으로 표현이 가능하다.

```depends_on``` 옵션은 ```docker run``` 명령에는 없는 도커 컴포즈에만 있는 옵션이다. 개별 명령을 실행할 때는 의미가 없기 때문이다.
도커 컴포즈를 사용하면 여러 서비스를 만들고 시작한다. 즉, 동시에 여러 컨테이너가 생성되는 것이다. 때로는 하나의 컨테이너가 이미 실행되고 있는 다른 컨테이너에 의존할 수 있다.
예를 들면 Backend 서비스는 Database 서비스에 의존하고 있다.     
즉, 어떤 컨테이너를 먼저 불러와야 함을 컴포즈에게 알려주기 위해서 사용한다.
```
services:
  backend:
    #...
    depends_on:
      - 
```
```depends_on``` 옵션에 대쉬(-) 를 통해 해당 서비스가 의존하는 서비스의 이름을 지정하기만 하면 된다.
물론 여러 요구사항이 있을 경우 대쉬(-) 를 추가해서 여러 서비스에 의존할 수도 있다.

Interactive 모드로 실행하기 위해서는 ```stdin_open``` 옵션을 true 로 설정해서 개방형 입력 연결이 필요하다는 것을 도커에게 알리면 된다. 
그리고 ```tty``` 옵션을 넣고 true 로 설정해야 한다. 
결국 ```-it``` 플래그는 개방형 표준 입력을 위한 대쉬(-) 와 입력 플래그의 조합인 것이다. 그리고 ```tty``` 는 터미널에 연결하기 위한 것이다.
```
services:
  frontend:
    #...
    stdin_open: true
    tty: true
```

예를 들면 최종적으로 아래와 같은 Docker Compose 를 통해 3 개의 컨테이너를 명령을 사용하지 않고 실행할 수 있게 됩니다.
따라서 읽기 쉽고, 조작하기 쉽고, 유지 관리하기 쉬워진다.

# 이미지 빌드 & 컨테이너 이름 이해하기
```$ docker-compose up``` 명령에 ```--build``` 옵션을 추가하면 이미지 리빌드를 강제할 수 있다.      
그렇지 않으면, 자체 Dockerfile 기반으로 빌드하는 backend 이미지와 같은 이미지가 한 번만 빌드된다.
그리고 도커 컴포즈가 이 이미지를 찾으면 그 다음에 이 모든 서비스를 시작하려고 시도한다.
즉, 기존 이미지를 재사용한다. 하지만 코드에서 무언가가 변경되어 이미지를 강제로 리빌드해서 새 이미지를 빌드해야 한다면
```$ docker-compose up``` 명령에 ```--build``` 옵션을 추가해서 이를 강제할 수 있다.

도커 허브나 외부 레파지토리에서 가져오는 이미지가 아닌 커스텀 이미지를 컨테이너를 시작하지 않고 빌드만 하려는 경우 ```docker-compose build``` 를 사용하여 이를 수행할 수 있다.
up 옵션으로 실행하면 빌드 단계가 포함된다. 이는 컨테이너를 시작하기 위해 이미지를 빌드해야 한다면 빌드가 된다. 

docker compose 를 통해 실행 시 컨테이너명은 Docker 에서 알아서 지정해준다. 하지만 ```docker-compose.yaml``` 파일에서 ```container_name``` 옵션을 추가하면 내가 원하는 컨테이너 명을 지을 수 있다.
```
sevices:
  mongodb:
    images: 'mongo'
    #...
    container_name: mongo
```

이렇게 Docker Compose 는 Docker 명령의 모든 것을 대체하지는 못하지만 수동적이고 반복적인 Docker 의 많은 명령들을 대체해주는 역할을 한다.
또한 볼륨과 바인드 마운트를 쉽게 설계할 수 있고, 자동으로 Docker Compose 파일 내의 모든 컨테이너에 대해 Default 된 네트워크를 생성해준다.
즉, Docker Compose 설정파일 하나만으로 명령 프롬프트나 터미널에 긴 명령을 입력하는 것보다 더 편리해진다.

하지만 Docker Compose 가 Dockerfile 을 대체하는 것은 아니라는 점을 기억하고 있어야 한다. ```docker run``` 과 ```docker build``` 명령을 대체해준다.