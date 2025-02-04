# 질답 형식으로 요약

<details>
<summary>...</summary>

...
</details>

<hr style="height: 3px; background-color: black; border: none;">

# 이론 정리

### 가상머신
- 물리적인 컴퓨팅 환경을 소프트웨어로 구현한 것
  - 시스템 가상 머신
    - 하나의 컴퓨터로 여러 OS를 사용하는 환경을 고립되게 구축
    - 실제 컴퓨터가 제공하는 것과 다른 형태의 ISA를 제공
  - 프로세스 가상 머신
    - OS안에서 일반 응용 프로그램을 실행하고 단일 프로세스를 지원하는 형태
    - JVM은 프로세스 가상 머신이며, 그 중 스택 기반의 VM이다.
    - 스택 기반의 VM은 피연산자를 저장하고 가져올 때 하드웨어(레지스터, CPU)를 직접 다루지 않고 스택을 활용한다.
      - 명령어 수가 많아지고 레지스터 기반 VM에 비해 성능이 나쁘다.
      - 그럼에도 불구하고 JVM이 스택 기반의 VM을 사용하는 이유는 레지스터 기반 VM보다 하드웨어에 덜 의존적이기 때문이다.
      - 레지스터 기반 VM은 연산 결과를 레지스터에 저장하므로 구현이 레지스터의 개수, 레지스터의 사이즈에 의존적일 수 밖에 없다.

### JVM이란
- OS에 종속받지 않고 자바를 실행하기 위해 CPU처럼 행동하는 가상 기계
  - OS마다 매칭되는 JVM이 있다.
- JVM도 하나의 프로세스이고 CPU는 JVM의 바이너리 코드를 실행한다.
  - Java Compiler는 JVM 외부에서 .java 파일을 .class 파일로 컴파일한다.
    - .class 파일을 구성하는 명령어의 크기가 대부분 1바이트라서 Java bytecode라고도 부른다.
  - JVM은 .class 파일을 받고, JIT Compiler를 활용해 .class 파일을 네이티브 머신 코드로 변환한다.
  - CPU가 JVM의 코드 영역에 있는 코드를 실행하면, JIT Compiler가 생성한 바이너리 코드를 실행하는 꼴이 되는 듯?
    - JIT Compiler가 만든 네이티브 머신 코드는 JVM의 코드영역에 저장되지 않고, JVM이 실행되는 프로세스의 메모리 내에 새로 할당된 영역(코드 캐시 또는 메소드 캐시)에 저장된다.
      - 자주 수행되는 메서드만 캐시에 저장해서 인터프리팅이 필요 없이 바로 CPU가 수행할 수 있도록 하고,
      - 자주 수행되지 않는 메서드는 그때그때 인터프리팅하는 방식인 것 같다.
    - 코드 캐시 또는 메소드 캐시는 코드 영역, 데이터 영역, 힙 영역, 스택 영역과는 별도의 영역으로 JVM이 실행 중에 JIT Compiler에 의해 동적으로 생성되는 코드가 저장되는 공간

### JVM의 구조
- Class Loader
  - .class 파일을 Runtime Data Area의 메서드 영역으로 불러오는 역할
  - 자바 클래스들을 한 번에 메모리에 올리는게 아니라, 필요할 때 동적으로 올릴 수 있도록 동적 클래스 로딩 기능을 담당
  - Loading, Linking, Initialization으로 구성
- Runtime Data Area
  - 자바 애플리케이션의 실행과 관련된 모든 데이터를 저장
- Execution Engine
  - .class 파일과 같은 바이트 코드를 실행 가능하도록 해석
  - Interpreter, JIT Comiler, Garbage Collector로 구성
  - GC는 Heap 영역에 배치된 객체들을 관리


### 참고
https://lsj31404.tistory.com/105
