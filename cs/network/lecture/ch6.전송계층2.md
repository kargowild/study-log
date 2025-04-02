### TCP
- TCP는 point to point 간의 통신을 책임진다.
  - 프로세스 하나가 소켓을 여러 개 열 수도 있으니, 엄밀히 말하면 프로세스는 point가 될 수 없다.
  - 소켓 하나가 하나의 point가 된다.
- reliable, in-order byte stream
- pipelined
- send & receive buffers
- full duplex data
  - 하나의 커넥션에서 데이터가 양방향으로 전송되는 것을 full duplex라고 부른다.
- connection-oriented
- flow controlled
  - receiver의 버퍼가 소화할 수 있는 양 만큼만 sender가 보내야 한다.
- congestion control
  - 네트워크가 받아들일 수 있는 만큼만 데이터를 부어야 한다.

### TCP segment structure
- header + message -> TCP 입장에서 message에 뭐가 들었는지는 전혀 상관하지 않는다.
- header는 오버헤드임. 실제로 header의 크기는 message의 크기에 비해 매우 작다.
- header의 크기는 정해져있지만, message의 크기는 가변적이다.
- header
  - source port #, dest port #
  - sequence number (sender 입장에서 입력)
  - acknowledgement number (receiver 입장에서 입력)
  - 각종 flag bit
  - receive window (receiver 입장에서 입력. receive buffer에 얼만큼의 빈 공간이 있는지 sender에게 알리는 용도)
  - checksum

### TCP seq#와 ACK#
- seq#는 sender 입장에서 기입하는 byte 번호이고, ACK#는 receiver 입장에서 기입하는 byte 번호이다.
- seq#
  - 현재 보내는 데이터의 가장 첫 byte가 보내려는 총 데이터의 몇 번째 byte인지에 대한 순서 정보
- ACK#
  - cumulative ACK인데, Go Back N의 ACK10은 10번까지 잘 받았다는 의미지만 TCP의 ACK10은 10번을 달라는 뜻임에 주의
  - ACK10이 중간에 유실돼도 ACK11을 sender가 받았다면, cumulative 특성 덕분에 넘어갈 수 있다.
  - 실제로 receiver는 세그먼트를 받을 때마다 ACK를 보내는게 아니라, 세그먼트가 하나 들어왔으면 곧 더 들어올거라 판단해서 500ms 기다리고 취합해서 마지막에 ACK를 보낸다.
- TCP에서 A와 B가 연결된다면, A와 B는 각자 sender이면서 receiver다.
  - 따라서 각자 send buffer, receive buffer를 가진다.
  - send buffer는 재전송을 대비한 window buffer이고, receive buffer는 순서가 잘못 올 것을 대비하는 buffer이다.
  - A의 send buffer에서 결정한 번호로 seq#를 보내면 B의 receive buffer는 이를 바탕으로 ACK#를 결정해 응답한다.

### TCP에서 time out
- timer는 window에서 하나만 존재한다.
  - oldest unacked segment에 대해 time out을 트래킹한다.
  - 즉 timer는 하나지만 각 segment마다 송신 시간을 기록해둠으로써 timer가 트래킹하는 segment를 항상 oldest unacked segment로 변경할 수 있다.
- 이론상 time out 값을 RTT로 잡으면 될 것 같지만, RTT 값은 패킷의 이동 경로마다 다르고, 경로가 같더라도 Queueing delay 때문에 시간에 따라 달라진다.
  - 과거 RTT history와 바로 직전 RTT 값을 고려해서 EstimatedRTT 값을 만든 후, time out 값에 활용한다.
- time out 값을 EstimatedRTT로 잡으면 time out이 아닌데도 time out으로 처리하는 경우가 너무 많다.
  - DevRTT라는 값을 구해서 4*DevRTT만큼의 마진을 준 값을 time out으로 잡는다.
  - 즉 time out값은 생각보다 엄청 길다.
  - loss를 판단하기 위해 time out에만 기대기에는 recovery가 너무 늦어진다.
  - loss를 감지하기 위한 추가적인 매커니즘을 활용한다. fast retransmit 매커니즘
  - 예를 들어 sender에서 ACK10을 처음 받고, ACK10을 연속 3번(총 4번) 더 받으면, 10번이 유실됐다고 판단한다. 
