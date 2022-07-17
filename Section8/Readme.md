# 섹션 8: 더 복잡한 설정: Laravel & PHP 도커화 프로젝트
# Target 설정
NodeJS 와 다르게 PHP/Laravel 은 PHP 가 서버를 구축하지 않기 때문에 PHP 만을 설치한다고해서 충분하지 않다. 그 외에 서버가 별도로 필요하기 때문이다.
그렇기에 도커를 사용하게 되면 쉽게 해결이 가능하다.

PHP 인터프리터 컨테이너와 코드를 실행하는데 필요한 부가 서버인 Nginx 웹 서버가 들어가 있다.
웹 서버 컨테이너는 기본적으로 들어오는 요청을 받은 다음 PHP 인터프리터로 이동해서 응답을 생성하여 요청을 전송한 클라이언트에게 그 응답을 돌려주도록 한다.     
그리고 데이터를 저장하기 위한 MYSQL 데이터베이스 컨테이너가 추가적으로 있어야 한다.
- 인터프리터: 프로그래밍 언어의 소스 코드를 바로 실행하는 컴퓨터 프로그램 또는 환경을 말한다.

각각 분리된 세 개의 컨테이너는 어플리케이션 컨테이너이다. 그리고 추가로 몇 개의 유틸리티 컨테이너가 필요하다.
1. 예를 들면 Composer 가 필요하다. Node 의 npm 같은 것으로 패키지를 설치하는데 사용하는 패키지 관리자이다.    
2. 그리고 Laravel Artisian 이라는 자체 도구를 제공한다. 데이터베이스에 대해 마이그레이션을 실행하고 초기 시작 데이터를 데이터베이스에 쓰는데 사용하는 명령이다.
3. 프론트엔드 로직의 일부에 npm 을 사용하기에 npm 컨테이너가 필요하다.

<img width="1348" alt="스크린샷 2022-07-06 오후 11 55 26" src="https://user-images.githubusercontent.com/63203480/177580481-b065e49c-7a82-4480-9892-6feb8de80a87.png">

# Nginx(웹 서버) 컨테이너 추가
nginx 서버는 들어오는 모든 요청을 받아들여서 PHP 인터프리터를 트리거한다.     
그러므로 php 컨테이너를 가지게 된다. php 컨테이너는 PHP 코드와 Laravel 코드를 실행한다.     
MYSQL 데이터베이스 컨테이너도 필요하다.     
그리고 유틸리티 컨테이너는 composer, artisan, npm 컨테이너가 필요하다.
```
services:
  server:
  php:
  mysql:
  composer:
  artisan:
  npm:
```

우선 서버는 강력하고 효율적인 웹 서버인 nginx 를 사용한다. Docker Hub 에서 공식적인 nginx 이미지를 제공한다.  
그리고 이것을 사용해서 nginx 서버를 설정할 수 있다. 이미지를 넣어줄 때는 잘못 해석되어 설정되지 않도록 작은따옴표(' ') 를 넣어주는게 좋다.    
이 서버 컨테이너에서 하는 일을 들어오는 요청을 살펴보고 PHP 컨테이너로 전달하는 것이다. 따라서 자체 구성을 제공하기 위해 볼륨을 추가해야 한다.
전체 파일을 바인드 마운트하지 않고 개별 파일을 대상으로 지정할 수도 있다. ```/etc/nginx/nginx.conf``` 해당 경로는 공식 문서에서 지정한 것이다.
그리고 컨테이너가 그 구성을 변경해서는 안 되기 때문에 ```ro``` 옵션으로 읽기전용으로 설정해야 한다.
```
services:
  server:
    image: 'nginx:stable-alpine'
    ports:
      - '8000:80'
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
```

로컬의 ```./nginx``` 경로의 ```nginx.conf``` 파일은 아래와 같다.
```
server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    root /var/www/html/public;
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:3000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

# PHP 컨테이너 추가
Dockerfile 은 아래와 같이 구성한다.
```dockerfile
FROM php:8.0-fpm-alpine

WORKDIR /var/www/html

RUN docker-php-ext-install pdo pdo_mysql
```
php 이미지는 Docker 에서 공식적으로 지원하는 이미지이다. 웹 사이트를 제공하는 웹 서버의 표준적인 폴더인 ```/var/www/html``` 을 작업 디렉토리로 사용한다.
그리고 ```nginx.conf``` 파일에서도 ```/var/www/html``` 경로를 작성했다. 이것은 결국 Laravel PHP 어플리케이션을 보관해야 하는 컨테이너 내부의 폴더이기 때문이다.

만약 Dockerfile 의 끝에 CMD 또는 ENTRYPOINT 가 없다면 베이스 이미지의 CMD 나 ENTRYPOINT 가 사용된다.

바인드 마운트시에 ```:delegated``` 옵션을 추가하면 컨테이너가 일부 데이터를 기록해야 하는 경우에 그에 대한 결과를 호스트 머신에 즉시 반영하지 않고 그 대신, 배치(batch) 로 기본 처리함으로써 
그 성능이 약간 더 나아진다.
```
services:
  php:
    build:
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes: 
      - ./src:/var/www/html:delegated
```

# MySQL 컨테이너 추가
docker hub 에서 MySQL 공식 이미지를 지원한다. 컨테이너가 시작되면 도커 허브에서 이미지를 Pull 하고, 이미지 내부에서 MySQL 데이터베이스가 시작된다.   
데이터베이스는 PHP 컨테이너에서 처리될 PHP 코드에 의해서 접근한다.    

그리고 데이터베이스 이미지에 의해 사용될 데이터베이스 설정, 사용자, 비밀번호 등을 몇몇 환경 변수를 설정해야 한다.  
```
services:
  mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql
```

# Composer 유틸리티 컨테이너 추가
Composer 유틸리티 컨테이너는 처음 Laravel 어플리케이션을 설정하는데 사용할 수 있다.     
Docker Hub 에서 [Composer 이미지](https://hub.docker.com/_/composer)를 지원한다.

```composer.dockerfile``` 이라는 새로운 Dockerfile 을 만든다. 
```dockerfile
FROM composer:latest

WORKDIR /var/www/html

ENTRYPOINT ["composer", "--ignore-platform-reqs"]
```
ENTRYPOINT 를 사용하기 위해서 Composer 이미지를 만드는 것이다.
ENTRYPOINT 를 위와 같이 설정하면 일부 종속성이 누락되더라고 경고나 오류 없이 실행할 수 있다.      
Working Directory 는 '/var/www/html' 로 설정한다. 왜냐하면 이 경로에 우리의 코드들이 있을 것이기 때문이다.

그리고 Dockerfile 을 만들었기에 docker compose 파일에서 만든 Dockerfile 을 추가해줘야 한다.
```
services:
  build:
    context: ./dockerfiles
    dockerfile: composer.dockerfile
  volumes:
    - ./src:/var/www/html
```
그리고 소스 폴더를 컨테이너 내부의 '/var/www/html' 폴더에 바인딩하기 위해서 바인드 마운트 설정을 위와 같이 해줘야 한다.     

예를 들면 **Composer 를 통해 컨테이너 내부의 해당 폴더에서 Laravel 어플리케이션을 생성한다면 로컬 호스트 머신의 소스 폴더로 다시 미러링**된다.
즉, Composer 유틸리티 컨테이너는 php, mysql, server 컨테이너를 만드는데 필요한 것이다.

```$ docker-compose run --rm composer create-project --prefer-dist laravel/laravel .``` 명령을 통해 컨테이너들을 실행할 수 있다.    
그리고 로컬의 src 폴더를 보면 그 안에 Laravel 어플리케이션이 있다.

# 일부 Docker Compose 서비스만 구동하기
```src/.env``` 폴더는 Laravel 에 의해 생성되었고 Laravel 에 대한 구성을 가지고 있다.   

현재 server 는 php 의 현재 코드에 대해 아무것도 모르고 있다. 들어오는 요청이 먼저 server 에 도달하고, php 파일에 대한 요청만이 php 인터프리터에 전달되야 한다.    
그렇기에 docker compose 파일에 부가적인 볼륨이 추가되어야 한다. ```- ./src:/var/www/html```     
그리고 그 부가 볼륨은 또다른 바인딩 마운트가 된다. ```/src``` 폴더를 컨테이너 내부의 ```/var/www/html``` 폴더에 바인딩 한다.
```
services: 
  server:
    image: 'nginx:stable-alpine'
    ports: 
      - '8000:80'
    volumes: 
      - ./src:/var/www/html
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
```
```nginx.conf``` 파일을 보면 ```/var/www/html``` 폴더에서 콘텐츠를 제공하고, 파일을 찾는다. 
따라서 바인딩하는 것이 server 를 통해 콘텐츠를 사용하기 위해 해야만 하는 일인 것이다.

만약 docker compose 파일에서 원하는 컨테이너만 사용하기 위해서 ```$ docker-compose up``` 명령에서 특별한 기능을 제공한다.    
예를 들면 server, php, mysql 컨테이너만 실행하고 싶다면 ```$ docker-compose up server php mysql``` 명령을 실행하면 된다.    
그리고 server 서비스에는 의존하는 서비스를 지정해야 한다. nginx 서버는 php 서비스와 통신할 수 있고, mysql 서비스와 통신할 수 있는 경우에만 작동하기 때문이다.

Dockerfile 이 변경되거나 또는 Dockerfile 을 통해 복사된 일부 파일이 변경된다면 이러한 변경 사항은 docker-compose 에 의해 적용되지 않는다. 
즉, 이미지를 리빌드 하지 않을 것이다. docker-compose 없이 변경된 이미지를 리빌드하기 위해서는 수동으로 ```$ docker build``` 를 실행했어야 한다.   
그러므로 docker-compose 가 Dockerfile 을 재평가하여 필요한 경우에 이미지를 리빌드하도록 하려면 ```--build``` 옵션도 추가해야 한다.
```
$ docker-compose up -d --build server php mysql
```
변경된 사항이 있는 경우에 이미지를 다시 생성하도록 강제한다. **아무것도 변경하지 않았다면 이미지의 레이어 개념으로 인해 그리고 레이어가 캐시되기 때문에 당연히 이미지가 리빌드 되지 않는다**.    
따라서 결과적으로 항상 최신의 이미지를 사용할 수 있게 되었다.    
항상 ```--build``` 옵션을 사용하기보다는 이미지에 대해 변경된 사항이 있는 경우에만 추가할 수도 있다.