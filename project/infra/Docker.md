# 실용적인 도커 사용법
### 도커를 사용하는 일반적인 과정
1. 도커 이미지 pull
  - 예시) docker pull nginx:stable-perl
2. 도커 이미지를 바탕으로 도커 컨테이너 생성
3. 도커 컨테이너 실행
- 위 3가지 과정을 한 번에 실행시키는 것이 docker run
  - 기본적으로 포그라운드에서 실행하므로, 백그라운드 실행을 위해선 -d 옵션을 사용하자.
  - docker run -d -p 4000:80 nginx
  
## docker image
### docker image 조회
- docker image ls
- docker images

### docker image 삭제
- docker image rm [이미지 ID]
  - 컨테이너(중지된 컨테이너 포함)에서 사용하고 있지 않은 이미지만 삭제 가능
  - -f 옵션 붙이면, 중지된 컨테이너에서 사용 중인 이미지도 삭제 가능
- docker image rm $(docker images -q)
  - 컨테이너(중지된 컨테이너 포함)에서 사용하고 있지 않은 모든 이미지 삭제
  - -q는 id만 나오게 하는 옵션
  - -f 옵션 붙이면, 중지된 컨테이너에서 사용 중인 이미지도 모두 삭제

## docker container
### docker container 실행
- docker start [컨테이너 ID]
- docker run --name my-mysql -d -p 13306:3306 -e MYSQL_ROOT_PASSWORD=password123 mysql
  - 이미지 pull, 도커 컨테이너 생성, 도커 컨테이너 실행을 올인원으로 수행 

### docker container에 bash로 접속
- docker exec -it [컨테이너 ID] bash
  - -i 옵션 : 표준 입력을 유지하여, 컨테이너와 상호작용할 수 있도록 해주는 옵션
  - -t 옵션 : 가상의 터미널을 할당하여 터미널 환경을 제공하는 옵션
- bash 자리에 mysql -u root -p와 같이 그냥 컨테이너에서 실행할 명령어를 바로 넣어줄 수도 있다.

### docker container 조회
- docker ps
  - 현재 실행 중인 모든 컨테이너 조회
  - -a 옵션 붙이면, 중지된 컨테이너까지 확인 가능

### docker container 중지
- docker stop [컨테이너 ID]
- docker kill [컨테이너 ID]
  - 컨테이너가 먹통일 때 강제종료

### docker container 삭제
- docker rm [컨테이너 ID]
  - 중지되어 있는 컨테이너만 삭제 가능
  - -f 옵션 붙이면 실행되고 있는 컨테이너도 삭제 가능
- docker rm $(docker ps -qa)
  - 중지되어 있는 모든 컨테이너 삭제
  - -f 옵션 붙이면 실행되고 있는 컨테이너까지 모두 삭제

## Volume
- 도커 컨테이너에서 데이터를 영속적으로 저장하기 위해 사용
- 볼륨은 컨테이너 자체의 저장 공간을 사용하지 않고, 호스트의 자체의 저장 공간을 공유해서 사용하는 형태
- docker run -v [호스트의 디렉토리 절대경로]:[컨테이너의 디렉토리 절대경로] [이미지명]:[태그명]
  - 호스트에 이미 디렉토리가 존재하면, 이를 컨테이너에 덮어 씌운다.
  - 호스트에 디렉토리가 존재하지 않으면, 컨테이너의 디렉토리 파일들을 호스트의 디렉토리로 복사한다.
- mysql을 도커 컨테이너로 띄우되, 데이터가 날아가지 않도록 볼륨을 사용하는 명령어
  - docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password123 -v /Users/minho/soma/docker-study/volume-test/my-data:/var/lib/mysql mysql

## Dockerfile
- 도커 허브에 존재하지 않는 나만의 이미지를 직접 만들어서 사용하고 싶을 때, Dockerfile을 활용
- 예시1
```dockerfile
FROM ubuntu

COPY app.txt /app.txt # 파일 복사
COPY my-app /my-app/ # 디렉토리 복사
COPY *.txt /text-files/ # 와일드 카드

ENTRYPOINT ["/bin/bash", "-c", "sleep 500"]
```
- 예시2
  - 아래 도커 파일로 도커 이미지를 만들기 전에, 프로젝트 경로에서 ./gradlew clean build를 수행해줘야 한다.
  - ./gradlew clean build를 따로 수행해주기 귀찮은데, 어떻게 해결할 수 있을까? docker compose?
```dockerfile
FROM openjdk:17-jdk

COPY build/libs/*SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

### FROM
- 베이스 이미지 지정

### COPY
- 호스트 컴퓨터에 있는 파일을 컨테이너로 복사
- 파일 하나만 복사할 수도 있고, 디렉토리 전체를 복사할 수도 있다.
- .dockerignore를 활용하면, 특정 파일 또는 폴더만 COPY의 대상에서 제외시킬 수 있다.

### ENTRYPOINT
- 컨테이너가 시작된 직후에 실행하고 싶은 명령어 지정

### RUN
- 이미지를 생성하는 과정에서 사용할 명령문 실행
  - 명령문이 실행된 결과까지 포함하여 하나의 이미지를 생성할 수 있다.

### Dokerfile을 기반으로 이미지를 만드는 명령어
- docker build -t [이미지명:태그] [도커파일 경로]

# 도커 이론
### Docker란?
- 컨테이너를 사용하여 각각의 프로그램을 분리된 환경에서 실행 및 관리할 수 있는 툴

### 가상서버 vs 컨테이너
- 컨테이너는 하나의 리눅스 서버가 마치 전용 서버에서 동작하고 있는 것 같은 분리 상태를 만들어 낸다.
  - 이는 리눅스 커널의 네임스페이스와 컨트롤 그룹(cgroup)이라는 기술을 기반으로 한다.

### 도커의 기능 및 원리
- 도커에는 로드밸런싱이나 스케일 아웃 지원, 프로세스 관리 등 다양힌 기능이 있지만 가장 핵심되는 기능은 다음과 같다.
  - 도커는 Pre-packaged된 도커 이미지(1 or more)로 컨테이너를 만들고, 그 위에서 애플리케이션을 실행시킨다.
  - 또한 컨테이너와 Host OS 커널이 통신할 수 있도록 네트워크 인터페이스를 구성하여 IP를 전달해준다.

### 도커와 커널 
- 리눅스 시스템은 커널과 user space로 구성되어 있다.
  - 커널은 하드웨어와 직접 상호작용하고 시스템의 자원을 관리한다.
  - user space는 애플리케이션, 라이브러리, 시스템 유틸리티 등이 실행되는 환경이다.
- 도커 컨테이너는 리눅스의 커널을 공유하지만, 각 컨테이너는 독립된 user space를 가지고 있다.
  - 즉 여러 도커 컨테이너들은 같은 호스트 OS 커널을 공유하면서도, 각각 자신만의 user space를 사용하여 실행된다.
  - 이렇게 각 컨테이너는 격리된 환경에서 애플리케이션을 실행할 수 있으며 ,이는 리소스 사용을 제한하거나 특정 환경을 설정할 수 있게 해준다.
- 도커 컨테이너는 자신이 리눅스 커널 위에서 실행될 것이라 기대함.
  - Host OS가 리눅스 기반이면 문제 없는데, 만약 윈도우나 맥OS에서 리눅스 컨테이너를 실행하려면, VM 위에서 실행해야 한다.
  - 도커 데스크탑이 내부적으로 리눅스 VM을 실행하여 리눅스 커널을 제공하는 방식으로 동작한다.

### 로컬에서도 도커 이미지를 사용해야 하는가?
- 여러 개발자가 협업하는 경우, 개발 환경이 제각각이라 로컬에서는 실행되지만 배포 환경에서 실패할 수 있다.
- 이상적으로는 로컬에서도 도커 이미지로 애플리케이션 실행시켜야 하는 것 같다.
  - 심지어 테스트 코드도 도커 이미지를 활용해서 실행시킬 수 있는 것 같다. 대신 속도는 느림

### Dockerfile
- docker 파일은 이미지를 빌드하기 위한 설명서 역할
- 이미지 빌드를 수행하는 주체는 Host(로컬머신)다.
- bacend-ci-cd-dev.yml에서 ubuntu latest가 build한 jar 파일을 도커 이미지로 만들어서 도커 허브에 업로드하는데, 이 때 도커 이미지를 만들기 위해 사용되는듯?

```dockerfile
FROM openjdk:17 # 도커 이미지를 만들 때 자바 17이 설치된 환경을 기반으로 컨테이너를 생성한다.
EXPOSE 8080 # 컨테이너가 실행될 때 8080포트를 외부에 공개하도록 설정한다. 이는 단순한 문서화 역할이며, 실제 포트 바인딩은 docker run -p 옵션을 사용하여 설정해야 한다.
ARG JAR_FILE=./build/libs/*-SNAPSHOT.jar # JAR_FILE이라는 빌드 시 사용할 변수를 정의 
COPY ${JAR_FILE} app.jar # JAR_FILE에서 지정한 JAR 파일을 app.jar 라는 이름으로 컨테이너 내부에 복사. 즉 빌드된 애플리케이션 JAR 파일을 컨테이너 내부로 가져옴
ENTRYPOINT ["java", "-jar","-Dspring.profiles.active=dev","app.jar"] # 컨테이너가 실행될 때 실행할 명령어를 설정. 즉, 스프링부트 애플리케이션을 dev 프로파일로 실행하도록 설정

```

### docker-compose.yml
- docker-compose를 안 쓰고, docker run -p 80:8080 어쩌구 이런 식으로 바꿀 수 있겠구나.
- 80 -> 8080 포트포워딩 하는 것 자체가 컨테이너와 Host OS 커널이 통신할 수 있는 네트워크 인터페이스가 컨테이너에 있기 때문인 듯

```yaml
version: "3.8"
services:
  database:
    container_name: staccato-database
    image: mysql:8.0.30
    environment:
      - MYSQL_DATABASE=staccato
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    volumes:
      - ./mysql:/var/lib/mysql
    ports:
      - "3306:3306"
    restart: always
    networks:
      - springboot-mysql-network
  application:
    container_name: staccato-backend-app
    image: ${STACCATO_IMAGE}
    depends_on:
      - database
    environment:
      - SPRING_DATASOURCE_URL=${SPRING_DATASOURCE_URL}
      - SPRING_DATASOURCE_USERNAME=${SPRING_DATASOURCE_USERNAME}
      - SPRING_DATASOURCE_PASSWORD=${SPRING_DATASOURCE_PASSWORD}
    ports:
      - "80:8080"
    restart: always
    networks:
      - springboot-mysql-network

networks:
  springboot-mysql-network:
    driver: bridge

```

### backend-ci-cd-dev.yml
```yaml
name: Backend CI/CD dev

on:
  push:
    branches: [ "develop-be" ]

jobs:
  ci:
    runs-on: ubuntu-latest

    defaults:
      run:
        shell: bash
        working-directory: ./backend

    permissions:
      contents: read

    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Setup with Gradle
        uses: gradle/actions/setup-gradle@417ae3ccd767c252f5661f1ace9f835f9654f2b5 # v3.1.0

      - name: Build with Gradle
        run: ./gradlew clean build

      - name: Login to Docker Hub
        uses: docker/login-action@v3.3.0
        with:
          username: ${{ secrets.DOCKERHUB_DEPLOY_USERNAME }}
          password: ${{ secrets.DOCKERHUB_DEPLOY_TOKEN }}

      - name: Docker Image Build
        run: |
          docker build --platform linux/arm64 -t hotea990/staccato -f Dockerfile .

      - name: Docker Hub Push
        run: docker push hotea990/staccato

  cd:
    needs: ci
    runs-on: self-hosted
    steps:
      - name: Pull Docker image
        run: |
          sudo docker login --username ${{ secrets.DOCKERHUB_DEPLOY_USERNAME }} --password ${{ secrets.DOCKERHUB_DEPLOY_TOKEN }}
          sudo docker pull hotea990/staccato

      - name: Docker Compose up
        run: sudo docker-compose -f /home/ubuntu/staccato/docker-compose.yml up -d

```
