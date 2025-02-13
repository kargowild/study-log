### RDT 복습
1. unreliable channel에서 발생할 수 있는 문제
  - packet error
  - packet loss
2. packet error에 대응하기 위한 매커니즘?
  - Error detection (checksum)
  - feedback (ACK, NAK)
  - retransmission
  - sequence# (중복패킷인지 새로운 패킷인지 구분)
3. packet loss에 대응하기 위한 매커니즘?
   - Timeout

### RDT의 퍼포먼스를 높이는 방법
- transmission time = L / R
  - L : packet length in bits
  - R : transmission rate, bps
- RDT3.0의 퍼포먼스 = (transmission time) / (RTT + transmission time)
  - transmission time보다 RTT가 훨씬 길기 때문에 퍼모먼스가 매우 나쁘다.
- sender가 ACK를 받기 전에, multiple 패킷을 보내는 pipelining을 사용하자!
  - pipelining의 퍼포먼스 = (window size * transmission time) / (RTT + transmission time)
  - Go-Back-N, selective repeat 방식이 있다. 
  - RDT3.0에서는 seq#를 0과 1로 표현 가능했지만, 여기서는 이 값이 늘어나야 한다.
  - sender나 receiver단에 버퍼를 추가해야 한다.

### Go-Back-N
- window size만큼의 패킷을 한꺼번에 보내고, 가장 왼쪽 패킷에 대한 ACK가 올 때마다 window를 오른쪽으로 한 칸씩 슬라이딩한다.
- cumulative ACK
  - 예를 들어 ACK11을 sender가 받았다면, 11번 패킷까지는 완벽하게 receiver가 잘 받았다고 이해할 수 있다.
- receiver는 굉장히 단순
  - 그저 자기가 받아야 하는 seq#만 주구장창 기다린다.
  - 중간에 문제가 생기면, receiver가 기다리는 seq#부터 sender가 다시 보내줘야 한다.
- n번 패킷의 타이머에서 타임아웃이 발생한다면, sender는 n번 패킷이 포함된 window의 모든 패킷을 재전송한다.
  - sender는 재전송을 대비해서 window의 모든 패킷을 버퍼에 저장해둬야 한다.
  - 타이머는 패킷단위가 아니라 윈도우 단위로 존재하는 듯

### Selective Repeat
- receiver 입장에서 순서에 맞지 않게 들어온 패킷이라도, 에러가 없으면 버퍼에 저장함으로써 문제가 있는 패킷만 재전송 받는다.
- not cumulative ACK
  - receiver는 순서에 맞지 않는 패킷이라도 오류가 없고 중복 패킷이 아니라면, 버퍼에 저장하고 ACK를 보낸다.
    - 순서에 맞게 온 패킷은 곧바로 application layer로 올리고, 순서가 맞지 않는 패킷은 순서가 맞을 때까지 버퍼에 저장해둔다.
  - sender는 ACK를 받지 못한 패킷만 재전송한다.
  - sender는 보내긴 했지만 아직 ACK를 받지 못한 모든 패킷에 대해 개별적으로 타이머를 관리해야 한다. (이것이 매우 큰 단점이라 TCP에서는 이를 개선)
    - 순서에 맞지 않게 ACK를 받더라도 해당 패킷의 타이머를 제거할 수 있다.
    - window 가장 왼쪽의 패킷에 대해 ACK를 받으면, window를 오른쪽으로 슬라이딩할 수 있다.
    - 이 떄, 늦게 보낸 패킷에 대해 ACK를 미리 받아놨다면, 여러 칸에 대한 슬라이딩을 한 번에 할 수도 있다.
- Go-Back-N과 마찬가지로 window 방식을 사용하며, sender는 window의 패킷을 버퍼에 저장해두어야 한다.
- window size가 n일 때, seq#는 2n까지만 사용해도 문제없다.
  - seq#가 2n보다 작아지면 새로 들어오는 패킷과 duplicated 패킷을 구별하지 못하는 상황이 발생할 수 있다. -> 버퍼에 저장할 패킷을 정확히 선별 불가
