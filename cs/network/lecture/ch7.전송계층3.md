### flow control
- sender는 recv buf의 현재 상태를 고려해서 데이터의 양을 조절해야 한다.
  - TCP 헤더의 recv buf 관련 필드가 있어서, sender 입장에서는 얼마나 보내야 할 지 명확하게 알 수 있다.
  - 만약 recv buf size = 0이면, sender가 주기적으로 의미 없는 segment라도 찔러줘야 ACK가 새로 와서 recv buf size 값을 갱신받을 수 있다.
    - 물론 보통 receiver도 동시에 sender 역할을 수행하기 때문에, 자기가 보내야 할 데이터를 보내면서 recv buf size까지 갱신하게 된다.
- TCP에서 가장 중요한 기능을 꼽으라면 RDT, flow control, congestion control
  - 그 중 flow control과 congestion control은 sender 입장에서 보낼 수 있는 데이터 양을 조절하는데 활용하는 매커니즘이다.
  - recv buf의 상태와 네트워크의 상태는 실시간으로 변하므로, 이를 sender 입장에서는 실시간으로 트래킹하면서 더 상태가 안 좋은 쪽에 맞춰서 데이터 양을 결정해야 한다.
  - recv buf 상태는 TCP 헤더 값으로 명확하게 알 수 있지만, 네트워크의 상태는 명확히 알 수가 없다.
    - 라우터가 자신의 큐 상태를 모든 클라이언트들에게 알려주면 좋겠지만, 라우터는 매우 바빠서 그럴 시간이 없다.
    - 따라서 end to end의 소통에서 발생하는 정보만으로 네트워크의 혼잡상황을 유추해야 한다.
    - ACK가 오는 속도와 time out이 발생하는 빈도를 통해 유추한다. -> 정확하진 않지만 도움이 됨.
- 참고 : TCP 세그먼트들이 recv buf로 순서대로 잘 들어오면 세그먼트가 도착하는 족족 application layer로 바로 넘어가는 줄 알았는데, 그게 아닌 것 같다.
  - 애플리케이션 측에서 TCP read를 위한 API를 호출하면 그제서야 넘어간다. 따라서 순서에 맞게 왔음에도 recv buf에서 대기해야 하는 경우도 발생할 수 있다.

### 3way handshake
- TCP sender, receiver는 data segment를 주고받기 전에 connection을 맺어야 한다.
  - connection을 맺는 과정에서 클라이언트와 서버 간 각자의 initial seq# 값을 공유한다.
  - 3way handshake 과정이 끝나야만 비로소 send buffer, recv buffer 자원을 할당한다.  
- 3way handshake 과정
  - SYN (c -> s)
    - TCP segment의 data는 비어있고 헤더의 SYN 1bit 필드가 보통은 0인데 이 때만 1로 셋팅되어 나간다.
    - 서버의 recv buf 쪽에서 준비할 수 있도록 클라이언트의 sender가 사용할 seq#를 서버에게 알린다.
  - SYNACK (s -> c)
    - TCP segment의 data는 비어있고 헤더의 SYN 1bit 필드와 ACK 1bit 필드를 1로 셋팅한다.
    - 클라이언트의 recv buf 쪽에서 준비할 수 있도록 서버의 sender가 사용할 seq#를 클라이언트에게 알린다.
    - 클라이언트는 SYNACK를 통해 서버가 살아있다는 사실을 확인할 수 있다.
  - ACK + 데이터 (c -> s)
    - 서버 입장에서 클라이언트와 연결이 잘 됐다는 확신을 갖기 위해 필요
    - 이 과정까지 끝나면 서버는 buffer를 만든다.
    - +) 보안 측면에서 클라이언트가 보내는 SYN만으로 서버가 buf를 할당하면, 서버를 공격하기가 매우 쉬워진다.

### closing TCP connection
- 이 과정이 있어야 buffer들을 release할 수 있다.
- 과정
  - FIN (c -> s)
    - 클라이언트 측에서 보낼 데이터가 더이상 없다면 FIN을 요청한다.
  - ACK (s -> c)
    - 클라이언트가 보낸 FIN에 대한 ACK인 듯
    - 서버 측에서는 FIN으로 답하기 전에, 보낼 데이터가 남아있다면 FIN을 보내기 전 일단 보내야 할 데이터를 보낸다.
  - FIN (s -> c)
  - ACK (c -> s)
    - 클라이언트가 보낸 ACK를 서버가 받으면, 서버는 자원을 해제할 수 있다.
    - 하지만 클라이언트는 ACK를 보내고 나서 일정 시간 timed wait을 거친 후 자원을 해제할 수 있다.
    - 왜냐하면 클라이언트가 마지막으로 보낸 ACK가 유실된 경우, 서버측 time out이 발동해서 서버가 FIN을 다시 보낼 것을 대비하기 위해
    - timed wait는 이상적으로 서버의 time out 시간과 유사하게 잡으면 좋지만, time out 시간은 독립적으로 운용되며 유동적으로 변하는 값이라서 참고 불가능
    - 따라서 적당히 널널한 시간으로 설정한다.
