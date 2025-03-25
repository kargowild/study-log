### java.util.concurrent
- synchronized의 단점을 보완하기 위해 자바 1.5부터 제공하는 동시성 문제 해결을 위한 라이브러리 패키지
- LockSupport를 포함한 수많은 클래스가 있다.

### LockSupport 
- synchronized의 가장 큰 단점인 무한 대기 문제를 해결할 수 있다.
- LockSupport의 아래 메서드를 수행하는 스레드를 BLOCKED 상태가 아닌 WAITING 또는 TIMED_WAITING 상태로 변경한다.
  - park(), parkNanos(nanos)
- LockSupport.unpark(thread)를 호출하면 WAITING 상태에 있던 스레드가 RUNNABLE로 깨어난다.
  - unpark(thread)가 아닌 thread.interrupt()를 사용할 수도 있다.
- 참고) BLOCKED, WAITING, TIMED_WAITING 상태 모두 실행 스케줄링에 들어가지 않기 때문에 CPU 입장에서는 실행하지 않는 비슷한 상태이다.
  - 다만 BLOCKED 상태는 무한 대기에 빠질 수 있으며, synchronized에서만 사용하는 특별한 대기 상태라고 이해하면 된다.

### ReentrantLock
- LockSupport를 활용해서 synchronized의 단점을 극복하면서도 매우 편리하게 임계 영역을 다룰 수 있는 다양한 기능을 제공
  - LockSupport는 너무 저수준이므로, 개발자가 이를 직접 활용해서 동시성 문제를 해결하는 코드를 짜기란 굉장히 어렵다.
  - Lock 인터페이스를 구현한 ReentrantLock을 활용하자!
- Lock 인터페이스가 제공하는 메서드
  - void lock()
    - 락 획득을 시도하고, 락이 이미 다른 스레드에 의해 점유되었다면 WAITING 상태로 대기한다.
    - 다만 인터럽트에 응답하지 않는다. (순간적으로 WAITING -> RUNNABLE이 되지만, lock 메서드 안에서 해당 스레드를 곧바로 WAITING 상태로 변경)
    - 주의! 여기서 사용하는 락은 객체 내부에 있는 모니터 락이 아니다. Lock 인터페이스와 ReentrantLock이 제공하는 기능이다.
    - 모니터 락과 BLOCKED 상태는 synchronized에서만 사용된다.
  - void lockInterruptibly()
    - 락 획득을 시도하고, 락이 이미 다른 스레드에 의해 점유되었다면 WAITING 상태로 대기한다.
    - 대기 중에 인터럽트가 발생하면 InterruptedException이 발생하며 락 획득을 포기한다.
  - boolean tryLock()
    - 락 획득을 시도하고, 즉시 성공 여부를 반환한다.
  - boolean tryLock(long time, TimeUnit unit)
    - 주어진 시간 동안 락 획득을 시도한다.
    - 대기 중에 인터럽트가 발생하면 InterruptedException이 발생하며 락 획득을 포기한다.
  - void unlock()
    - 락을 해제한다. 락을 해제하면 락 획득을 대기 중인 스레드 중 하나가 락을 획득할 수 있게 된다.
    - 락을 획득한 스레드가 호출해야 하며, 그렇지 않으면 IllegalMonitorStateException이 발생할 수 있다.
  - Condition newCondition()
    - Condition 객체를 생성하여 반환한다.
    - Condition 객체는 락과 결합되어 사용되며, 스레드가 특정 조건을 기다리거나 신호를 받을 수 있도록 한다.
    - 이는 Object 클래스의 wait, notify, notifyAll 메서드와 유사한 역할을 한다.
- ReentrantLock은 공정성 모드와 비공정 모드로 설정할 수 있으며, 두 모드는 락을 획득하는 방식에서 차이가 있다.
  - 비공정 모드는 락을 획득하는 속도가 빠른 대신, 새로운 스레드가 기존 대기 스레드보다 먼저 락을 획득할 수 있어 기아 현상이 발생할 수도 있다.
    - (락을 반납하는 스레드가 대기 큐의 스레드를 깨우는 잠깐 사이에 새로운 스레드가 먼저 락을 가져가는 경우) 
  - 공정 모드는 대기 큐에서 먼저 대기한 스레드가 락을 먼저 획득하도록 보장하는 대신 락을 획득하는 속도가 느려질 수 있다.
