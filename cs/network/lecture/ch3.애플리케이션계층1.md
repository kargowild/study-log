### 소켓
- 애플리케이션과 네트워크 사이의 인터페이스
  - 프로세스와 프로세스 간 통신을 위해 OS가 제공하는 API
- 애플리케이션 개발자 입장에서는 OS가 소켓 기반으로 어떻게 동작하는 지는 몰라도, OS가 제공하는 API를 어떻게 사용하는 지는 알아야 한다.
- OS에는 application layer 아래의 layer들이 모두 구현되어 있다.
  - 그 중에서 application layer와 소통하는 것이 transport layer
  - OS는 transport layer를 TCP와 UDP로 구현해놨음
  - 따라서 애플리케이션 입장에서는 목적에 따라 TCP 소켓이나 UDP 소켓을 선택해서 프로세스 간 통신을 하면 된다!

### TCP 소켓을 사용하기 위한 API 호출 흐름
- TCP 서버 입장
  - int socket(int domain, int type, int protocol) 
    - type 파라미터로 TCP 소켓을 생성할 지 UDP 소켓을 생성할 지 결정
    - 생성된 소켓id가 반환된다.
  - int bind(int sockfd, struct sockaddr* myaddr, int addrlen)
    - 생성한 소켓을 특정 포트에 바인딩할 때 사용하는 API
    - 웹서버는 포트를 고정시켜놔야 하지만, 웹 클라이언트는 남는 포트를 아무거나 사용하면 되므로 bind()를 사용할 필요가 없다.
    - 바인딩에 성공하면 0을 리턴
  - int listen(int sockfd, int backlog)
    - 서버니까 이 소켓을 listen 용도로 사용하겠다.
    - 동시에 여러 요청이 들어오면 최대 몇 개까지 큐에 담아놓고 순서대로 처리할 것인지 결정
    - non-blocking이라 호출 즉시 문제가 없다면 0을 리턴
  - int accept(int sockfd, struct sockaddr* cliaddr, int* addrlen)
    - blocking이라서 클라이언트로부터 요청이 올 때까지 return하지 않고 대기
    - 요청이 오면, cliaddr라는 구조체에 클라이언트의 주소 정보를 담고 accept 함수는 리턴한다.
- TCP 클라이언트 입장
  - socket()
  - int connect(int sockfd, struct sockaddr* servaddr, int addrlen)
    - 연결하고 싶은 서버의 IP와 포트번호를 입력해서 TCP three-way handshaking
    - 성공하면 0을 리턴
- 연결이 된 이후에는 소켓에 대고 read() 또는 write()를 호출하면 된다.
  - 소켓A와 소켓B가 연결이 되면, 이제 소켓 A에 데이터를 쓰면 전 세계 수많은 소켓 중에 소켓 B에서만 소켓 A가 보낸 데이터가 뿅 하고 나오게 된다.
  - int write(int sockfd, char* buf, size_t nbytes)
    - blocking이라서 데이터가 보내진 후에 return한다
  - int read(int sockfd, char* buf, size_t nbytes)
    - blocking이라서 데이터를 전달 받은 후 return한다.

### UDP 소켓을 사용하기 위한 API 호출 흐름
- TCP 서버 입장
  - socket()
  - bind()
- TCP 클라이언트 입장
  - socket()
- 이후 소켓에 대고 sendto(), recvfrom() 호출

### Multiplexing, Demultiplexing
- TCP든 UDP든 transport layer라면 기본적으로 제공해야 하는 기능
  - error checking도 있다. error가 포함된 데이터가 도착하면 application layer에 전달하지 않는 기능
- Multiplexing
  - application layer에서 여러 프로세스가 각자의 소켓으로 데이터를 write했을 때, 이들을 취합하여 세그먼트로 만들고 아래 layer로 전달
  - 들어오는 창구는 여러 개인데 나가는 창구가 하나
- Demultiplexing
  - network layer에서 온 여러 세그먼트들의 헤더 정보를 보고 매칭되는 application layer의 socket에 데이터를 집어넣는 작업
  - 들어오는 창구는 하나인데 나가는 창구가 여러 개
  - UDP의 경우 connection이 없으므로 dest IP, dest port만 가지고 demultiplexing이 가능하다.
    - 즉 서버 입장에서는 UDP 소켓이 하나만 있으면 된다.
  - TCP의 경우 개별적인 connection이 있으므로 src IP, src port, dest IP, dest port를 모두 사용해서 demultiplexing을 한다.
    - 이 중 하나라도 다르면 다른 소켓으로 올라간다. 
    - 즉 서버 입장에서는 동시에 connection을 맺고 있는 사용자 수만큼 TCP 소켓이 필요하다.

### WAS에서 동시 HTTP 요청의 갯수만큼 TCP 소켓과 스레드가 필요한가? (지피티에게 질문)
- 100개의 HTTP 요청이 동시에 들어오면 100개의 TCP 소켓이 생성되지만, 그것이 곧 100개의 스레드로 매핑된다는 의미는 아님.
  - Blocking I/O 방식 (Tomcat BIO 모드 등):
    - 요청당 하나의 스레드가 필요하므로 100개의 요청 → 100개의 스레드 & 100개의 TCP 소켓
  - Non-Blocking I/O 방식 (Tomcat NIO, Netty 등):
    - 적은 수의 스레드가 여러 개의 TCP 소켓을 처리할 수 있음.
    - 100개의 요청 → 100개의 TCP 소켓 → 10~20개 스레드가 관리 가능
    - 💡 즉, WAS가 어떤 I/O 모델을 사용하느냐에 따라 스레드와 소켓의 개수는 다르게 동작할 수 있습니다. 🚀
