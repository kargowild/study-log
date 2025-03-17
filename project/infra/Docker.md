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
