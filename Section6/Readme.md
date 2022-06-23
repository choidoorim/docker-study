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