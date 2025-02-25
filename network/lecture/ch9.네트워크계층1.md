### Network layer에서 다루는 내용?
- Application layer는 packet loss 또는 error같은 건 신경쓰지 않는다.
  - sender가 소켓에 데이터를 write()하면, receiver 쪽 소켓에서 read()할 때 해당 데이터가 무조건 뿅 하고 나온다고 생각한다.
- Transport layer는 sender가 보낸 세그먼트가 어떤 경로로 receiver에게 도착할 수 있는지 신경쓰지 않는다.
  - 어떤 경로를 통했든 packet loss가 없는 한, 일단 receiver 쪽으로 잘 도착할 것이라 생각한다.
  - 따라서 포트 번호는 일단 receiver의 transport layer까지 세그먼트가 잘 도착했다는 가정하에 활용할 수 있는 번호다.
- Network layer부터는 정말 어떤 라우터를 거쳐 데이터가 receiver에게 전달되고 있는지를 배운다.
  - 라우터는 패킷을 받아 패킷의 목적지 방향을 찾아서 forwarding할 수 있어야 한다.
  - 따라서 라우터도 Network layer까지는 처리할 수 있어야 한다.
  - 배송지를 결정하는 과정에서 IP protocol이 사용된다.

### Network layer의 주요한 두가지 기능
- forwarding
  - router input으로 들어온 packet을 적절한 router output으로 이동시킨다.
- routing
  - 패킷이 목적지까지 가기 위해 필요한 경로(route)를 결정한다.
  - 패킷의 목적지는 IP 헤더에 적혀있다.
  - 라우팅 알고리즘을 활용해 local forwarding table을 만드는 과정
  - local forwarding table은 해당 라우터 안에서 IP 헤더를 보고 어떤 output link로 연결해야 하는지를 결정한다.
  - 모든 IP를 독립적으로 라우팅 테이블의 엔트리에 대응시키는 것은 현실적으로 불가능하기 때문에, 주소 범위로 그룹핑하여 output link와 대응시킨다.
- 라우터는 결국 forwarding table을 보고 forwarding만 계~속 한다.
