### Network edge
- applications and hosts
- end systems 간 데이터 전송이 목표
  - TCP
    - connection-oriented 
    - 데이터를 전송하기 전에 준비 과정이 필요
    - 내가 보낸 메시지가 신뢰성 있게 전달된다.
      - 라우터 큐가 꽉 차서 패킷이 유실될 경우, 재전송한다.
      - 이 때, 라우터에서 재전송하는게 아니라 송신자가 처음부터 재전송한다.
      - 라우터는 어마어마한 데이터를 계속해서 처리해야 하므로 단순 작업에 최적화 시켜놨기 때문에 재전송 로직이 들어갈 수 없다.
      - 만약 라우터에 유실패킷재전송 기능을 추가하게 되면, 전체적인 네트워크 성능은 굉장히 느려질 것이다.
      - 따라서 모든 지능은 end point에 집약시켜놓고 TCP에 맞게 검증한다.
    - flow control : receiver의 수용력에 맞춰서 전송
    - congestion control : sender와 receiver를 연결하는 네트워크의 수용력에 맞춰서 전송
  - UDP
    - connectionless
    - no flow control, no congestion control
    - 즉 sender는 receiver나 네트워크 상태를 고려하지 않고 마구잡이로 데이터를 보낼 수 있다.
    - 패킷 몇 개 유실돼도 사람이 인지하지 못하는 음성통화와 같은 경우에 사용.
    - DNS는 왜 UDP를 쓸까? (출처 : 구글링)
      - IP를 한 번만 넘겨주면 끝이므로 굳이 TCP 연결을 맺는 것은 과하다.
      - TCP 에서는 buffer를 위해 자원을 할당하므로 DNS 서버가 TCP를 쓰면 동시에 받을 수 있는 사용자 요청 수가 제한된다.

### Network Core
- 라우터들이 얽혀있는 구조
- net을 통해 데이터 전송하는 방법
  - circuit switching
    - 사용자에게 dedicated circuit 할당
    - 사용자가 할당받은 circuit은 다른 사용자와 공유하지 않는다.
    - 성능이 보장되지만, 동시 사용자의 수가 물리적으로 제한된다.
  - packet switching
    - 사용자의 데이터를 그때그때 discrete chunks로 간주하고, 올바른 방향을 찾아서 전송
    - 기본적으로 유저 수에 제약이 없다.
    - 인터넷 사용 패턴을 보면, 모든 사용자가 동시에 네트워크를 사용하지는 않는다. 오히려 멍때리는 시간이 더 길다.
      - 따라서 인터넷에는 circuit switching보다 packet switching이 적합
      - 다만 확률적으로 bandwidth를 초과할만큼 사용자들이 동시에 요청을 보낸다면, 그 때는 문제가 생길 수 있다.

### packet delay의 4가지 요소
1. nodal processing delay
   - 라우터가 새로 들어온 패킷의 bit errors를 확인하고 output link를 결정하는데 걸리는 시간
   - 성능이 좋은 라우터를 사면 delay를 줄일 수 있다.
     - 고속도로 톨게이트에서 현금결제를 하이패스로 바꾸는 방법과 유사
2. queueing delay
   - output link로 패킷이 나가는 속도보다 라우터로 들어오는 속도가 빠를 경우, 데이터가 유실되지 않도록 라우터 내에 임시 공간(큐)이 필요
     - 큐는 output link마다 각각 존재한다.
   - 유저 수가 많으면 패킷이 큐에서 대기하는 시간이 길어진다.
   - 목적지까지 도달하는데 라우터가 여러 개라면, 각각의 큐에서 독립적으로 queueing delay가 발생
   - 큐가 꽉 찼는데도 패킷이 더 들어오면, 어쩔 수 없이 해당 패킷이 유실된다.
     - 실제 패킷 유실의 90%가 여기서 발생. 요즘 link는 성능이 좋아서 유실이 거의 발생 안 함
   - queueing delay는 네트워크 공급자가 어떻게 해결할 수 있는 문제가 아님
     - 유저 수가 많은 시간대에는 커지고, 적은 시간대에는 작아짐
     - 고속도로 톨게이트를 하이패스로 바꾸고 차선을 늘려도 추석에는 막히는 것과 유사
3. transmission delay
   - 패킷의 첫 번째 bit가 link로 뿜어져 나가는 시점부터, 마지막 bit가 뿜어져 나가는 시점까지의 시간
   - 패킷은 한 묶음으로 처리돼야 하기 때문에 패킷의 앞 부분이 먼저 다음 라우터에 도착했다고 해서 다음 단계를 진행할 수는 없다.
     - 따라서 하나의 패킷 내에서도 transmission delay를 고려해야 하는 것
   - 케이블의 bandwidth를 늘리면 delay를 줄일 수 있다.
     - 고속도로 톨게이트의 차선을 늘리는 것과 유사
4. propagation delay
   - 마지막 bit가 link에 올라와서 다음 라우터까지 도달하는데 걸리는 시간
   - 전자기파의 속도(빛의 속도)와 link의 길이에 의존
   - delay 개선이 불가능한 영역 

### DNS
- DNS 서버는 트리 구조를 이루고 있다.
  - 상위 서버는 하위 서버의 IP 주소를 알고 있다.
  - Root NS, TLD NS, Sub Domain NS로 계층이 나뉘게 된다.
- 클라이언트가 www.staccato.com의 IP 주소를 알고 싶다면, 
  - 클라이언트가 브라우저 캐시를 확인한다. 없으면
  - 컴퓨터 내의 host 파일과 캐시를 확인한다. 없으면
  - 로컬 DNS 서버(SKT와 같은 ISP가 제공)에게 질의한다. 로컬 DNS 서버는
    - Root NS에게 질의하여 .com NS의 주소를 알아낸다.
    - .com NS(TLD NS)에게 질의하여 staccato.com NS(Sub Domain NS)의 주소를 알아낸다.
      - 가비아를 쓰면, ns.gabia.co.kr의 주소를 알아내고, 기관에서 직접 네임서버를 운영하면, 해당 주소를 줄 것 같다.
    - staccato.com에게 질의하여 www.staccato.com의 주소를 알아낸다.
      - 가비아를 쓰면, ns.gabia.co.kr에게 www.staccato.com의 주소를 질의해 알아내는 듯
  - 로컬 DNS 서버는 www.staccato.com의 주소를 클라이언트에게 반환한다.
- 로컬 DNS 서버는 DNS 정보를 실제로 관리하고 있지 않으므로, 권한 없는 네임서버라고 부른다.
- Root NS, TLD NS, Sub Domain NS는 DNS 정보를 실제로 관리하고 있으므로, 권한 있는 네임서버라고 부른다. (Sub Domain NS만 해당하나?) 
- A 타입의 레코드는 도메인 주소를 키로, IP 주소를 값으로 갖는다.
- NS 타입의 레코드는 도메인 주소를 키로, 해당 도메인을 관리하는 Authoritative NS의 도메인명을 값으로 갖는다.
