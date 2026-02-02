### JPA란?
자바 ORM 기술 표준(인터페이스 모음). 대표적인 구현체는 Hibernate.
- ORM(Object Relational Mapping) - 객체와 RDB 테이블의 데이터 매핑시켜 SQL문 없이 객체를 사용해 객체 지향적인 방식으로 데이터를 조작할 수 있도록 도와주는 기술.

### Spring Data JPA란?
Spring에서 제공해주는 모듈 중 하나. JPA를 더 쉽게 사용할 수 있도록 도와준다.

**Spring Data JPA 사용 vs 순수 JPA(Hibernate)사용 예시**
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

Spring Data JPA는 save와 findById같은 기본적인 CRUD는 제공. (SimpleJpaRepository)
추가적인 메소드는 메소드명을 규칙에 맞게 작성하면 메서드 이름을 분석해 JPQL을 자동 생성.

### 영속성 컨텍스트
//TODO
