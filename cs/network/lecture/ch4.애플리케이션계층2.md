### RDT (Reliable Data Transfer)
- RDT?
  1. Message error가 없음이 보장되어야 한다.
  2. Meesage loss가 없음이 보장되어야 한다.
- Application layer 입장에서는 대부분 RDT를 원한다.
- 하지만 실제 네트워크는 RDT를 보장할 수 없다.
- 따라서 Transport layer에서 TCP의 여러 로직을 통해 Application layer간 통신에서 RDT를 보장한다.

### Simple RDT
- only stop-and-wait 프로토콜을 가정

### RDT1.0
- 완벽하게 reliable한 채널에서 데이터를 전송하는 경우
- Transport layer에서 reliable을 위한 특정 로직을 수행해줄 필요가 없다.

### RDT2.0
- 채널에서 패킷 에러는 발생할 수 있지만, loss는 발생하지 않는 경우
- 세그먼트 헤더에 checksum bits를 추가해서 전송하면, receiver에서 error detection이 가능하다.
  - receiver에서 checksum을 확인했을 때 에러가 없으면 ACK, 있으면 NAK를 응답한다.
  - NAK를 받을 경우, sender는 패킷을 재전송한다.

### RDT2.1
- 피드백(ACK나 NAK)에 에러가 있는 경우? 
  - sender는 receiver가 데이터를 잘 받았는지 알 수 없으므로, 보수적으로 그냥 패킷을 다시 보낸다.
  - receiver 입장에서 ACK를 보냈는데 중복패킷이 다시 올 경우, seq#를 보고 이것이 중복 패킷인지 판단하여, 중복 패킷이라면 버리고 ACK를 보낸다.
  - RDT2.0에서는 seq#가 0 또는 1의 값만 가져도 중복패킷인지 구분할 수 있다.  

### RDT2.2
- NAK를 없애되, receiver가 ACK를 보낼 때 가장 마지막으로 잘 받은 seq#를 포함시킨다.
  - sender 입장에서는 duplicate ACK를 받는다면, NAK와 같은 신호라고 판단한다.

### RDT3.0
- 패킷 loss까지 발생할 수 있는 환경에서는, 타이머가 필요하다.
- sender는 타이머를 합리적인 시간으로 설정해두고, ACK가 오지 않아 타임아웃이 발생하면, 패킷을 재전송한다.
  - 타이머를 짧게 잡으면, loss가 발생했을 때 recovery가 빠르지만, delay가 긴 경우에도 loss로 착각해서 중복패킷을 계속 보낼 수도 있다.
  - 중복 패킷을 계속 보내면 네트워크 오버헤드가 늘어난다.
  - 타이머를 어떻게 합리적으로 설정할 것인지는 TCP 얘기할 때 자세히 다룸
- 한계
  - sender 입장에서 데이터를 보내고 ACK가 올 때까지, 채널에 여유가 있어도 아무런 동작도 하지 않는다.
  - reliable은 보장할 수 있지만, 프로토콜을 지키느라 physical resources를 낭비하는 버전이다.
  - 실제로는 sender가 패킷을 쏟아붓고, ACK를 한 번에 받는 식으로 동작한다.
    - reliable 보장을 위한 피드백, seq# 관리가 RDT3.0에 비해 복잡해진다.
