# 참고자료
- https://blog.review-me.page/blog/ci-cd

# CI with Github Actions
- Workflows
  - job의 집합으로서 repository에서 특정 이벤트가 트리거되면 실행된다.
  - ./github/workflows 디렉토리 밑에 .yml 또는 .yaml을 생성해야 한다.
  - workflow는 다른 workflow에서 재사용할 수도 있다.
- Events
  - workflow가 실행되게끔 하는 이벤트
  - PR을 열거나, 이슈를 열거나, push하거나 등 다양한 event가 존재한다.
- Jobs
  - workflow에 속하는 step의 집합이며, 같은 runner안에서 실행된다.
  - job끼리는 병렬적으로 실행된다.
    - 다른 job과의 의존관계가 존재한다면, 이를 needs로 명시해야 한다.
  - 각 step은 shell script이거나 action 중 하나이다.
  - step은 순서대로 실행되며, 이전 step이 다음 것에 영향을 끼친다.
- Actions
  - Github Actions에서 사용되는 custom application으로서 workflow에서의 중복되는 코드를 라이브러리화 한 것
- Runners
  - workflow가 돌아가는 os가 포함된 서버. 
  - Github는 Unbuntu Linux, Windows, macOS Runner를 제공한다.
  - 직접 runner를 호스팅할 수도 있는데, 이를 self-hosted runner라고 부른다.

  
# backend-ci.yml
- PR이 올라온 시점, 그리고 push가 발생한 시점마다 CI가 돌아간다. 왜?
- 아래에서 build는 하나의 job이며, 이 이름은 자유롭게 변경할 수 있다.
- working-directory: ./backend에서 경로가 2024-staccato repository 기준인건가?
  - CI를 돌리는 주체는 github의 서버인거지?
  - runs-on: ubuntu-latest가 github의 서버임을 암시하는 듯.
  - CD에는 runs-on: [ self-hosted, dev ]라고 되어 있음.
- 아... actions/checkout@v4가 현재 레포지토리를 clone해서 코드를 사용할 수 있도록 하는 action이었구나.
  - 아루 블로그 예시처럼 name을 붙여주는게 맞겠다.
- 근데 on에 pull_request만 적어줘도, PR이 열릴 때랑 PR에 코드가 변경될 때 모두를 포함하는건가?

- 아래는 맨 처음 만들었던 yml파일 
```yaml
name: Backend CI

on:
  pull_request:
    branches: [ "develop-be" ]

jobs:
  build:
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

      - name: Build with Gradle
        uses: gradle/actions/setup-gradle@417ae3ccd767c252f5661f1ace9f835f9654f2b5 # v3.1.0

      - name: Test with Gradle
        run: ./gradlew build

```

- 아래는 이후 수정한 yml파일
```yaml
name: Backend CI

on:
  pull_request:
    paths: 'backend/**'
    branches: [ "develop-be", "develop", "main" ]

permissions: write-all # jacoco에서 github PR에 댓글을 남기기 위해 write-all 권한을 줘야했나보다.

jobs:
  build:

    runs-on: ubuntu-latest

    defaults:
      run:
        shell: bash
        working-directory: ./backend

    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build with Gradle
        uses: gradle/actions/setup-gradle@417ae3ccd767c252f5661f1ace9f835f9654f2b5 # v3.1.0

      - name: Test with Gradle
        run: ./gradlew build

      - name: Publish Test Results
        uses: EnricoMi/publish-unit-test-result-action@v2
        if: always()
        with:
          files: ${{ github.workspace }}/backend/build/test-results/**/*.xml

      - name: Jacoco Report to PR
        id: jacoco
        uses: madrapps/jacoco-report@v1.6.1
        with:
          paths: ${{ github.workspace }}/backend/build/reports/jacoco/test/jacocoTestReport.xml
          token: ${{ secrets.GITHUB_TOKEN }}
          min-coverage-overall: 70
          min-coverage-changed-files: 70
          title: "🌻Test Coverage Report"
          update-comment: true

```

# backend-ci-cd-dev.yml
- 이 파일에서 ci를 또 정의해줘야 하는 이유가 뭐지?
  - 빌드해서 이미지를 도커허브에 올려주는 작업을 여기서 해줌.
  - PR에 코드가 계속 push될 때마다 도커허브에 이미지를 올려줄 필요는 없으니까, 배포하기 전에 한 번만 그 작업을 한다. 
  - ci라고 부르기 좀 애매한 작업같네. 이름 선정이 미스인 듯?
- steps는 사실 그냥 터미널에서 수행하는 명령어들에 이름을 붙이고, 순서대로 실행하는 역할이구나.
  - 그 중에 actions에서 제공하는 명령어들도 쓸 수 있는거고
- 여기서 왜 도커를 썼는가 설명할 수 있어야겠다. 왜 도커랑 도커파일을 썼는가?
- self hosted runner는 polling을 통해 50초간 대기- 끊고 다시 연결을 반복한다. 
  - 그럼 github에서 신호를 주는 일이 아예 없고, 50초마다 ec2가 github에게 polling 요청을 하는건가?

- 아래는 맨 처음 작성한 yml파일
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
      contents: read # default가 read이므로 생략 가능

    steps:
      - uses: actions/checkout@v4 # workflow를 실행시키기 위해 github가 제공하는 runner에서 staccato repository를 git clone하는 작업
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

- 아래는 이후 수정한 yml파일
```yaml
name: Backend CI/CD dev

on:
  push:
    paths: [ 'backend/**', '.github/**' ]
    branches: [ "develop-be", "develop" ]

jobs:
  ci:
    runs-on: [ self-hosted, dev ]

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
          sudo docker build --platform linux/arm64 -t staccato/staccato:dev -f Dockerfile.dev .

      - name: Docker Hub Push
        run: |
          sudo docker login --username ${{ secrets.DOCKERHUB_DEPLOY_USERNAME }} --password ${{ secrets.DOCKERHUB_DEPLOY_TOKEN }}
          sudo docker push staccato/staccato:dev

  cd:
    needs: ci
    runs-on: [ self-hosted, dev ]
    steps:
      - name: Pull Docker image
        run: |
          sudo docker login --username ${{ secrets.DOCKERHUB_DEPLOY_USERNAME }} --password ${{ secrets.DOCKERHUB_DEPLOY_TOKEN }}
          sudo docker pull staccato/staccato:dev

      - name: Stop and remove existing container
        run: |
          sudo docker stop staccato-backend-app || true
          sudo docker rm staccato-backend-app || true

      - name: Docker run
        run: |
          sudo docker run --env-file /home/ubuntu/staccato/.env \
          -v /home/ubuntu/staccato/logs:/logs \
          -p 8080:8080 \
          -d --name staccato-backend-app staccato/staccato:dev
          sudo docker image prune -af


```

# backend-ci-cd-prod.yml
- 서버 인스턴스가 2개니까 prod 쪽은 cd를 할 때 두 개의 서버에게 self hosted로 돌리라고 한꺼번에 신호를 주기 위해 matrix를 사용하는 듯
  - 스크립트를 steps에 작성해도 됐을텐데, deploy.sh로 파일을 따로 뺐던 이유가 뭐더라? 

```yaml
name: Backend CI/CD prod

on:
  push:
    paths: [ 'backend/**', '.github/**' ]
    branches: [ "release-be" ]

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
          sudo docker build --platform linux/arm64 -t staccato/staccato:prod -f Dockerfile.prod .

      - name: Docker Hub Push
        run: |
          sudo docker login --username ${{ secrets.DOCKERHUB_DEPLOY_USERNAME }} --password ${{ secrets.DOCKERHUB_DEPLOY_TOKEN }}
          sudo docker push staccato/staccato:prod

  cd:
    needs: ci
    runs-on: [ self-hosted, prod ]

    strategy:
      matrix:
        runner: [ prod-a, prod-b ]

    steps:
      - name: execute deploy.sh
        run: bash /home/ubuntu/staccato/deploy.sh

```
