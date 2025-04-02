### DDL 생성 기능
- 런타임에 JPA에 영향을 주지 않고 DDL을 생성할 때만 옵션의 효과가 발휘
- ex. @Column(nullable = false, length = 10)

### @Column
- (insertable = false)는 런타임에 JPA에 영향을 준다.
  - DB에 직접 SQL을 날리면 이 옵션은 효력이 없지만, JPA를 통해서 DB에 접근할 때는 효력이 있다.
- (nullable = false)는 DDL 생성 시에 not null 제약 조건을 붙인다.
  - 근데 이런 옵션보다 실제로는 schema.sql을 쓰니까, 개발자들이 보기 편하라고 있는 정도일 듯

### @Enumerated
- 자바 enum 타입을 매핑할 때 사용
- EnumType.STRING을 사용해서 enum 이름을 DB에 저장하자.

### @Transient
- 필드 매핑X, 데이터베이스에 저장 및 조회 X
- 주로 메모리 상에서만 임시로 어떤 값을 보관하고 싶을 때 사용

### 기본 키 매핑 방법
- 직접 할당하는 경우 @Id만 사용. 일반적으로 자동 생성
- @GeneratedValue
  - IDENTITY: DB에 위임
    - MYSQL의 경우 AUTO_INCREMENT
    - 영속성컨텍스트에서 관리되는 엔티티들은 무조건 PK를 가지고 있어야 하므로 em.persist() 시점에 insert 쿼리를 날려서 id를 가져와야 한다.
    - 즉 commit() 시점까지 insert 쿼리를 미룰 수 없음 
  - SEQUENCE: DB 시퀀스 오브젝트 사용, ORACLE
    - DB 시퀀스는 유일한 값을 순서대로 생성하는 특별한 DB Object
    - id만 따로 가져오므로 엔티티에 대한 insert 쿼리를 commit() 시점까지 미룰 수 있다.
    - 하나의 트랜잭션에서 여러 번의 insert 작업이 필요할 때, 동시성 문제를 피하면서 성능도 챙길 수 있다.
  - TABLE : 키 생성용 테이블 사용. 모든 DB에서 사용 가능
  - AUTO : DB 방언에 따라 자동 지정
