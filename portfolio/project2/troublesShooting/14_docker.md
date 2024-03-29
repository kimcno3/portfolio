# :pushpin: Docker를 활용한 개발 환경 관리
## 문제점
만약 프로젝트를 제작하면서 개인 로컬 환경에서 모든 플랫폼을 직접 설치해 코드 작성 및 테스트를 진행하고 문제가 없어 외부 서버에 배포를 한다고 가정했을 때, 배포 환경과 개발 환경의 차이로 인해 발생할 수 있는 문제점은 개발 환경에서 제어할 수 없고 예측하기 힘들 것입니다. 이러한 문제를 해결하기 위해서는 어플리케이션 구성을 위한 여러 플랫폼의 종류나 버전이 항상 동일하게 유지되고 수정될 수 있도록 독립적인 개발 및 배포 환경이 필요했습니다.

## 해결방안
### Docker
![](https://tecoble.techcourse.co.kr/static/4b1c0d70e42521b54fd3a9ee81ef82e7/d1228/docker.png)
Docker는 애플리케이션을 신속하게 구축, 테스트 및 배포할 수 있는 소프트웨어 플랫폼으로 소프트웨어를 컨테이너라는 표준화된 유닛으로 패키징하며, 컨테이너에는 라이브러리, 시스템 도구, 코드, 런타임 등 소프트웨어를 실행하는 데 필요한 모든 것이 포함되어 있습니다.

Docker Desktop을 직접 다운받아 컨테이너를 띄우고 터미널 명령어를 통해 개발 환경을 구축해도 되지만 Docker compose를 이용하면 yml파일에 명령어를 작성함으로써 간편하게 환경 관리 및 실행을 할 수 있습니다.

### Docker compose
Docker compose란, 여러 개의 컨테이너로부터 이루어진 서비스를 구축, 실행하는 순서를 자동으로 하여, 관리를 하나의 파일에서 간단하게 할 수 있도록 해주는 기능을 의미합니다. 

#### dockor-compose.yml
```yml
version : "3"
services:
  db:
    platform: linux/x86_64
    container_name: soldout-db
    image: mysql
    environment:
      MYSQL_DATABASE: soldout_db
      MYSQL_USER: soldout
      MYSQL_PASSWORD: 1234
      MYSQL_ROOT_PASSWORD: 1234
    volumes:
      - ./db/data:/var/lib/mysql:rw
    ports:
      - "3306:3306"
    restart: always

  redis-docker:
    container_name: soldout-redis
    image: redis:latest
    volumes:
      - ./example-docker-data/redis:/data
    ports:
      - "6379:6379"
```

위 코드처럼 compose 파일을 준비하여 커맨드를 1회 실행하면, 그 파일로부터 설정을 읽어들여 모든 컨테이너 서비스를 실행시키는 것이 가능하게 해줍니다.

## 마치며
Docker를 활용해 서버 환경을 독립적으로 운용하고 관리하면서 플랫폼의 버전관리를 용이하게 할 수 있고, 환경 변화에 따른 장애에 대한 대비를 할 수 있게 되었습니다.

## 참고자료
- https://aws.amazon.com/ko/docker/
- https://engineer-mole.tistory.com/221