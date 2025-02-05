# 질답 형식으로 요약
(2회독 때 추가 예정)
<details>
<summary>...</summary>

...
</details>

<hr style="height: 3px; background-color: black; border: none;">

# 이론 정리

### Creating a network app
- 네트워크를 활용한 프로그램을 개발할 때, application layer 이후는 고려할 필요가 없다.
  - application layer 이후 동작은 네트워크 시스템에서 이미 다 공통적으로 구축해놨기 때문
  - 따라서 application layer만 잘 신경써서 프로그램을 짜면, 어떻게든 수신자의 application layer로 도달한다고 생각하면 된다.
- 라우터에는 network 계층까지만 있다.
  - 반면 복잡한 로직은 모두 transport layer, application layer에 있음
  - 즉 라우터는 지능적인 업무를 수행하지 않고 단순 업무에 최적화 되어 있다.

### Socket
- 하나의 컴퓨터 내에서 서로 다른 프로세스 간 통신은, 시스템콜을 통해 OS가 제공하는 IPC를 활용한다.
- OS에서는 서로 다른 컴퓨터에서 두 프로세스 간 통신을 위한 인터페이스도 만들어놨다.
  - 그게 바로 Socket
  - 따라서 Socket에 write, read하는 작업도 시스템 콜이다.
  - 네트워크를 사용하는 프로세스라면, 프로세스마다 소켓이 존재하는 듯 
  - application layer에서는 그저 소켓에 대고 application layer protocol에 맞는 데이터를 write하거나 read할 뿐
  - TCP에서는 클라이언트 프로세스의 소켓과 서버 프로세스의 소켓간 TCP 연결을 수립한 후 해당 소켓을 통해 데이터를 주고 받는다.
- 소켓 주소 = (IP 주소 + 포트 번호)
  - 웹서버는 일반적으로 80포트를 사용하기 때문에, 웹 브라우저에 IP 주소만 적어주면 80 포트라고 생각하고 소켓 주소를 완성해준다.
  - 일반적으로 서버의 소켓 주소는 고정되어 있어야 한다. 그래야 클라이언트들이 찾아올 수 있음

### applicaiton layer가 transport layer에게 원하는 기능
- 네트워크 계층에서 상위 계층은 하위 계층이 제공하는 서비스를 이용할 수 있는 개념
  - 따라서 application layer는 transport layer가 제공하는 서비스를 이용할 수 있다.
- application layer는 다음과 같은 서비스를 원한다.
  - data integrity
    - 데이터가 온전하게 목적지에 도착하길 기대
    - 사실 transport layer는 딱 이것까지만 제공하고, 아래 3가지는 제공하지 않음
      - 따라서 아래 3가지는 application layer에서 직접 구현해야함
    - data integrity 마저도 TCP만 제공하고 UDP는 제공하지 않음
  - timing
    - 송신자 입장에서 보내는 데이터가 특정 시간 안에 도착하길 기대
    - ex. 음성통화에서 중요
  - throughput
    - 송신자 입장에서 보내는 데이터 용량이 최소한 예를 들어 1Gbps 정도는 나오길 기대
    - ex. 영화 다운로드에서 중요
  - security

### HTTP overview
- hypertext를 전송
  - hypertext란 일반 텍스트와 크게 다를 것 없음. 그저 중간에 다른 text를 찾아가기 위한 링크가 함께 있을 뿐
- HTTP는 application layer에서 정의한 프로토콜
  - 송신자의 데이터가 수신자로 도달하려면 결국 transport layer를 거쳐야 한다.
  - 그리고 HTTP는 transport layer에서 제공하는 TCP의 기능을 사용한다.
  - 따라서 HTTP를 사용하면 당연하게도 TCP 연결이 필요한 것
- HTTP는 stateless하다.
  - 즉 서버 입장에서 과거 클라이언트 요청에 대한 정보를 기억하지 않는다.
- HTTP connections
  - non-persistent HTTP
    - TCP 연결을 맺은 후, 메시지를 딱 한 번 주고 받고 연결을 끊는다.
  - persistent HTTP
    - TCP 연결을 맺은 후, 연결을 재사용한다.
    - 실제 웹 브라우저에서 기본으로 사용
    - 실제로는 persistent + 파이프라이닝 방식으로 request를 여러 개 연속으로 보내고, response를 연속으로 받는다.
      - ex. 클라이언트의 웹 브라우저가 수신한 HTML 파일을 파싱 중, 10개의 이미지가 필요한 경우 
