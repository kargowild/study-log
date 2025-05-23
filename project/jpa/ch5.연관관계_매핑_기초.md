### 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다.
- 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾지만, 객체는 참조를 사용해서 연관된 객체를 찾는다.
- 엔티티가 아닌 객체가 중개해주면 되지 않나? 그것도 비효율적이라고 생각하는건가.

### 객체지향모델링
- Member의 Team 필드에 @JoinColumn(name = "TEAM_ID")를 적어주는 건, JPA에게 Team을 가져오는 방법을 알려주는 것
- @ManyToOne을 사용하면, Member를 가져올 때 left join으로 Team까지 함께 가져온다.
  - fetch = EAGER가 디폴트이기 때문. LAZY로 깔고, 필요한 경우에만 fetch join 사용하자.
  - 급 궁금점. 대규모 데이터 환경에서 join의 성능이 걱정되면, FROM 절에서 서브쿼리를 써서 임시테이블을 활용하거나 JOIN에 ON절을 추가하면 되나?

### mappedBy
- Team과 Member의 일대다 매핑의 경우, Team 테이블에는 member_id 컬럼이 없다.
- (mappedBy = "team")을 적어주면 Member 엔티티의 Team 필드를 찾아가 @JoinColumn을 활용할 수 있다.

### 연관관계의 주인
- 양방향 매핑 규칙 : 객체의 두 관계 중 하나를 연관관계의 주인으로 지정
- 연관관계의 주인만이 외래 키를 관리
  - 주인이 아닌 쪽에만 값을 입력하는 실수가 자주 발생하므로 주의할 것
- 주인이 아닌 쪽은 읽기만 가능
  - JPA 입장에서는 mappedBy 쪽에 값을 입력해줄 필요가 없지만, 객체지향적으로 생각하면 양쪽 다 값을 넣어주는게 맞다.
  - 그렇지 않으면 같은 트랜잭션에서 em.find로 team을 조회했을 때 List<Member>가 비어있는 현상 발생 가능
  - 반면 값을 넣어주면 1차캐시에서 꺼내올 때는 객체의 동등성이 보장되므로 List<Member>에 값이 들어가 있다.
- 주인은 mappedBy 속성을 사용하지 않는다.
  - 그럼 일대다 양방향 매핑의 경우, 다 쪽을 연관관계의 주인으로 했을 때, 일 쪽에서 mappedBy를 쓰나?
- 외래 키가 있는 곳을 주인으로 정하자.
  - Team을 연관관계의 주인으로 정하면, Team의 값을 변경했는데 Member 테이블에 update 쿼리가 나가서 운영 환경에서 매우 헷갈린다.
- 양방향 매핑 시 무한 루프를 조심하자.
  - 양방향이 걸려있는 Entity를 JSON으로 바꾸는 순간 무한 루프에 빠진다.
  - 이는 Entity를 직접 JSON으로 바꾸지 말고, DTO를 JSON으로 바꾸면 쉽게 해결됨
  - 또한 lombok에서 toString을 만들어도 무한 루프에 빠질 수 있으므로, toString을 재정의할 때는 양방향 걸려있는 필드를 빼고 써야한다.

### 양방향 연관관계
- 이론상으로는 양방향 연관관계가 없어도 애플리케이션 개발을 모두 할 수 있다.
- 다만 실무에서 JPQL을 많이 사용하는데, 이 때 보통 Order를 조회할 때 List<OrderItem>도 한꺼번에 조회하고 싶다.
- 그러면 Order에 OrderItem 필드가 있어야 한다.
- 또한 객체지향적으로 개발하다보면 Order가 OrderItem을 필요로 할 때가 있다.
