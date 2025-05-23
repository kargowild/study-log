### 데이터 링크 계층
- 네트워크에서 데이터를 관리하고 전달하는 계층
- 데이터를 작은 프레임 단위로 분할
- MAC 주소로 장비를 식별
- 데이터 전송의 신뢰성과 효율성에 중요한 역할
  - 네트워크에서 여러 기기가 동시에 데이터를 전송하려 하면 신호 충돌이 발생
  - 데이터 링크 계층이 데이터의 원활한 흐름과 네트워크 매체에서 충돌 관리를 수행
  - 오류를 탐지하고 수정하는 역할도 데이터 링크 계층이 수행 (회선 제어, 오류 제어, 흐름 제어) -> 이 3가지 제어와 데이터 링크 계층의 연관이 잘 이해가 안감
 
### 회선 제어
- 오류를 감지하기보다는 회피하는 방법으로 신호 간 충돌 현상이 발생하지 않도록 제어
- 신호의 시작을 의미하는 ENQ와 끝을 의미하는 EOT를 명시적으로 지정
- 수신자는 송신자로부터 신호를 받으면 ACK를 응답
- 이러면 신호 간 충돌을 피할 수 있다?

### 오류 제어
- 송수신 데이터가 외부 간섭, 시간 지연 등에 의해 데이터가 변형되거나 순서가 어긋나는 등 통신 장애가 발생하지 않도록 오류를 검출하고 정정하여 통신에 대한 신뢰성을 확보
- 오류 검사
  - 패리티 검사 : 1을 항상 짝수개 보내기 위한 패리티 비트 추가
  - CRC : 데이터에 CRC 코드를 추가하여 오류를 감지. 송신자가 CRC 코드를 생성하고 수신자 측에서도 동일한 계산을 수행하여 일치 여부를 확인
  - Checksum : 데이터의 각 바이트 값을 더하거나 연산하여 값을 생성하여 추가
  - 해밍코드 : 데이터 비트와 조합되는 패리티 비트를 추가해 오류가 발생한 비트를 식별하여 수정 가능

### 흐름 제어
- 송신자와 수신자의 데이터 처리 속도 차이를 해결하기 위해 수신자 상황에 따라 송신자의 데이터 전송량을 조정하는 방법
- Stop & Wait : 송신자가 하나의 데이터를 전송한 후 ACK를 받아야만 다음 데이터를 전송

### 이더넷
- 데이터 링크 계층에서 사용되는 네트워크 구조
- 다수의 컴퓨터, 허브, 스위치 등을 하나의 인터넷 케이블에 연결한 네트워크 구조 (그냥 컴퓨터마다 허브에 직접 연결하면 안 되나?)
- 랜과 왠에서도 이더넷을 사용하기 때문에 일반적인 인터넷 환경에서 사용하는 기술이다.
- CSMA/CD (Carrier Sense Multiple Access/Collision Detection)
  - 2대 이상의 컴퓨터가 동시에 프레임을 보내는 충돌 현상을 방지하기 위해, 전류의 강도를 확인해 케이블이 사용 중인지 확인하는 방식
  - 전류의 강도 세기가 높다면, 낮아질 때까지 기다렸다가 프레임을 송신한다.
  - 눈치 게임 방식
    - 1-persistent : 케이블이 사용 중인지 상태를 계속 확인하다가 아무도 사용하지 않는 상태이면 바로 데이터를 전송. 충돌 위험성 가장 높음
    - nonpersistent : 누구도 케이블을 사용하고 있지 않다면 데이터를 바로 보내고, 누군가 사용 중이라면 임의의 시간을 기다렸다가 상태를 다시 확인
    - p-persistent : 케이블이 사용 중이면 기다렸다가 다시 상태를 확인하고, 아무도 사용하고 있지 않다면 일정 확률로 데이터를 전송
  - 스위치 장비가 CSMA/CD의 역할을 대신 하기 때문에, 현재 CSMA/CD는 잘 사용하지 않는다.

### MAC(Media Access Control) 주소
- 랜 카드에 할당된 고유한 48비트 값
  - 상위 24비트는 랜 카드 제조사에 부여된 코드
  - 하위 24비트는 제조사가 랜 카드에 부여한 고유 번호
- 2계층의 헤더에는 목적지 MAC 주소, 출발지 MAC 주소, 유형(프로토콜)이 포함된다.
  - 프로토콜의 종류에는 IPv4(32bit), ARP, IPv6(128bit)가 있다.
  - ARP(Address Resolution Protocol) : IP 주소와 MAC 주소를 매핑하기 위한 프로토콜
- ARP를 이용해 목적지 MAC 주소를 알아내는 방법은 다음과 같다.
  - 컴퓨터 A는 허브 A(정확히는 스위치)에게 172.168.10.33이라는 IP 주소의 MAC 주소를 묻는다.
  - 허브 A가 해당 IP를 관리하지 않는 경우 허브 A는 라우터에게 다시 질의한다.
  - 라우터는 허브 B에게 질의한다.
  - 허브 B는 브로드캐스트로 모든 컴퓨터에게 MAC 주소를 묻는다.
  - 모든 컴퓨터는 자신의 IP를 172.168.10.33과 비교하고, 일치하는 컴퓨터는 자신의 MAC 주소를 허브 B에게 응답한다.
  - 이후 컴퓨터 A는 컴퓨터 B의 IP와 MAC 주소를 메모리에 저장하는데 이것을 ARP 테이블이라고 한다.
- 의문
  - IP 주소로 수신지를 찾아갈 수 있는데 굳이 MAC 주소가 필요한 이유를 잘 모르겠다. 수신지의 랜카드로 그냥 데이터 꽂으면 되는거 아닌가?
  - 허브가 특정 IP에 해당하는 MAC 주소를 얻기 위해 브로드캐스트를 하는 이유가 뭘까? IP로 하나의 컴퓨터를 특정할 수 있는 능력이 허브에게 없나?

### 스위치
- 허브의 경우, 컴퓨터 A에서 데이터를 보내면 허브에 물려 있는 모든 컴퓨터에 그 데이터가 전송된다. -> 비효율적 -> 스위치 등장
- 스위치는 소규모 네트워크 안에서 컴퓨터, 프린터 등 모든 장치를 서로 연결해서 데이터를 쉽게 공유할 수 있도록 하는 장비
- 스위치는 포트에 연결된 컴퓨터의 MAC 주소를 MAC 테이블로 관리한다.
  - MAC 테이블에는 포트 번호와 MAC 주소가 맵핑되어 있다. (IP 주소는? 2계층에서는 고려 안하는건가?)
  - 스위치가 처음부터 모든 컴퓨터의 MAC 주소를 가지고 있는게 아니다.
  - 컴퓨터 A가 특정 MAC 주소를 가진 장치에게 데이터 전송을 스위치에게 요청하는 경우
    - 스위치의 MAC 테이블에 해당 MAC 주소가 없다면, 스위치에 물려 있는 모든 컴퓨터에게 데이터를 보낸다. (플러딩)
    - 모든 컴퓨터는 자신의 MAC 주소를 응답하고, 스위치는 포트마다 해당 MAC 주소를 MAC 테이블에 업데이트한다.
  - 컴퓨터 A가 수신자의 MAC 주소를 모르는 경우엔 ARP 과정이 필요할 듯

### 단방향 통신
- 선로가 하나만 있어서 일방 통행만 가능 (TV, 라디오)
- 네트워크에서는 이 선로를 채널이라고 부른다.

### 양방향 통신
- 하나의 통신 채널에서 송수신이 모두 가능한 방식
- 반이중 방식 (Half Duplex)
  - 양쪽 방향에서 통신이 가능하지만 동시에 통신할 수는 없음
  - 송신과 수신을 번갈아가며 통신하기 때문에 데이터 전송 속도가 빠르며, 주로 허브에서 사용하는 방식
  - 채널을 하나만 사용하기 때문에 속도는 빠르지만 충돌 문제가 남아있음
  - 충돌이 발생할 때 그 영향이 미치는 범위를 충돌 도메인이라고 하며, 하나의 허브에 컴퓨터들이 연결되어 있다면 여기에 연결된 모든 컴퓨터가 충돌 도메인이 됨
- 전이중 방식 (Full Duplex)
  - 채널을 2개 두어서 양쪽 방향에서 동시에 데이터를 주고 받을 수 있는 방식이며 스위치에서 사용됨
  - 채널을 2개 사용하기 때문에 충돌 문제를 해결할 수 있음
