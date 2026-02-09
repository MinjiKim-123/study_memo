### JPA란?
자바 ORM 기술 표준(인터페이스 모음). 대표적인 구현체는 Hibernate.
- ORM(Object Relational Mapping) - 객체와 RDB 테이블의 데이터 매핑시켜 SQL문 없이 객체를 사용해 객체 지향적인 방식으로 데이터를 조작할 수 있도록 도와주는 기술.

### Spring Data JPA란?
Spring에서 제공해주는 모듈 중 하나. JPA를 더 쉽게 사용할 수 있도록 도와준다.

**Spring Data JPA 사용 vs JPA(Hibernate)사용 예시**
```
@Repository
public class UserRepository {
  @PersistenceContext
  private EntityManager em;

  public void save(User user) {
    em.persist(user);
  }

  public void findById(Long id) {
    return em.find(User.class, id);
  }

  public void findByName(Long id) {
     return em.createQuery(
            "select u from User u where u.name = :name", User.class)
            .setParameter("name", name)
            .getSingleResult();
  }
}
```
<sub>JPA 예시</sub>

```
public interface UserRepository extends JpaRepository<User, Long> {

    User findByName(String name);
}
```
<sub>Spring Data JPA 예시</sub>

Spring Data JPA는 save와 findById같은 기본적인 CRUD는 제공. (SimpleJpaRepository) <br/>
추가적인 메소드는 메소드명을 규칙에 맞게 작성하면 메서드 이름을 분석해 JPQL을 자동 생성.

### 영속성 컨텍스트
엔티티를 보관하는 논리적인 공간. EntityManager(엔티티를 관리하는 역할)를 통해 엔티티를 관리하며, <br/>
대부분 엔티티매니저 생성시 영속성 컨텍스트가 1:1대로 생성된다.<br/>
영속성 컨텍스트는 트랜잭션 단위로 생성 및 관리됨.

**엔티티 LifeCycle**
총 4가지의 상태가 존재.
1. 비영속(new): 영속성 컨텍스트와 관련이 없는 상태. 
어플리케이션 내에서 entity를 생성하고 엔티티 매니저를 통해 조작하지는 않은 순수한 자바 객체 상태.
2. 영속(managed): entity가 영속성 컨텍스트에 의해 관리되는 상태.
persist, find 메소드를 통해 entity를 영속화.
3. 준영속(detached): 영속성 컨텍스트에 의해 관리되다가 분리되어 현재는 더 이상 관리를 받지 않는 상태.
4. 삭제(removed): DB삭제를 예약한 상태. COMMIT시 DELETE 쿼리를 실행하여 데이터 삭제.

**1차캐시**<br/>
영속성 컨텍스트의 주요 기능 중 하나로, 영속성 컨텍스트에서 엔티티를 보관하는 저장소.<br/>
JPA find시 바로 DB조회를 하지 않고, 1차캐시에서 키 값을 기준으로 조회 후 캐시에 존재하지 않을 경우 DB에서 조회.<br/>
JPQL,QueryDsl,findBy다른컬럼시에는 1차캐시 조회 거치지 않고 바로 DB조회 -> 1차 캐시에 동일 데이터 존재시 캐시 데이터 반환 & 미존재시 조회 데이터 캐싱 처리 및 반환 하며,<br/>
캐시 데이터는 key(entity타입&PK), value(Entity) 형태로 저장되어 관리됨.

**Dirty Checking**<br/>
영속된 Entity의 상태 변경 감지를 뜻하며, <br/>
entity 영속시 최초 상태를 스냅샷으로 만들고 flush 시점에<br/>
최초 상태와 현재 상태를 비교하여 변경 사항이 있을 경우 Update Query를 실행시킴.<br/>
(트랜잭션 필수. 트랜잭션이 없을 경우 동작하지 않음)<br/><br/>
-장점
1. 변경된 컬럼만 UPDATE
2. 
-단점
