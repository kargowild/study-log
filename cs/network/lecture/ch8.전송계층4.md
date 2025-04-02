### TCP Intuition
- 네트워크를 물 파이프에 비유
- 파이프의 두께는 알 수 없고, 심지어 중간중간 두께가 다 다르다.
  - 가장 두께가 얇은 파이프가 critical한 지점이다.
  - sender는 해당 파이프가 받아들일 수 있는 양을 고려하여 CongWinSize를 결정해야 한다.
  - 라우터가 report를 해주지 않기 때문에, 그 지점이 어딘지, 두께가 얼마인지 알 수 없다.
  - packet loss가 발생하는 순간을 바탕으로 네트워크 상황을 유추하여 CongWinSize를 결정한다.
- sender 입장에서는 현재 네트워크 상황을 모르면서 처음부터 막 퍼부을 수는 없다.
  - 파이프는 public 하기 때문에 다같이 망할 수 있기 때문이다.
  - 처음에는 한 방울 보내는 걸로 시작해야 한다.

### TCP Congestion Control
- 3 main phases
  1. slow start : 네트워크 상황을 모르므로 겸손하게 시작하지만, 윈도우 크기가 증가하는 속도는 exponential
  2. additive increase : threshold 지점에 도달하면 그때부터는 linear 하게 증가시킨다.
  3. multiplicative decrease
     - 네트워크가 한 번 막혔다면 모든 사람들이 발을 다같이 확 빼야 막힌게 풀리기 때문에 1/2씩 확확 줄인다.
     - 네트워크가 막혔다는 것은 패킷 유실로 판단하고, 패킷 유실은 3 dup ACK 또는 time out에 의해 탐지된다.
     - 3 dup ACK의 경우, 다른 패킷들은 다 잘 갔는데 한 놈만 문제가 생긴 경우다. 
       - Threshold 값을 패킷 유실이 감지된 지점의 1/2 값으로 변경하고, Threshold 값부터 시작해서 Additive increase를 다시 시작한다.
     - time out의 경우 네트워크가 매우 혼잡하여 모든 패킷이 유실된 경우다.
       - Threshold 값을 패킷 유실이 감지된 지점의 1/2 값으로 변경하고, 1부터 다시 시작한다.
     - TCP ver1(Tahoe)은 둘을 구분하지 않았지만 ver2(Reno)부터 둘을 구분하기 시작
- 참고 : MSS(Maximum Segment Size) = 500B
  - 즉 첫번째로 TCP segment를 보낼 때는 500B까지 보낼 수 있고, 그 다음부터 CongWinSize가 커지는데, 이 때 커지는 최소 단위가 MSS

### TCP Fairness
- K개의 TCP sessions가 같은 bottleneck link를 공유할 때, link의 bandwidth R에 대해 R/K만큼 공평하게 할당받는가
- 통신을 일찍 시작한 놈과 늦게 시작한 놈이 있으면, 각자 1/2로 줄어들고 linear 하게 증가하고를 반복하면 결국 R/K로 수렴하게 된다.
- 따라서 각 TCP 소켓 관점에서는 공평하게 분배된다.
  - 하지만 하나의 클라이언트가 여러 개의 TCP 소켓을 열게 되면, 그만큼 많은 bandwidth를 할당받게 된다.
