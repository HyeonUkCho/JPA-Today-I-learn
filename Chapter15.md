15장 고급 주제와 성능 최적화
========================

## JPA 표준 예외처리

- 트랜잭션 롤백을 표시하는 예외 : 예외 발생하면 트랜잭션을 강제로 커밋해도 커밋되지 않고 대신에 Javax.persistence.RollbackException 예외 발생한다.

- 트랜잭션 롤백을 표시하지 않는 예외 : 심각한 예외가 아니다. 커밋할지 롤백할지 결정이 필요하다.

### 스프링 프레임워크의 JPA 예외 변환

- 서비스 계층에서 데이터 접근 계층의 구현 기술에 직접 의존하는 것은 좋은 설계가 아니다. 이에 스프링은 추상화해서 개발자에게 제공한다. 만약, 예외변환을 하고 싶지 않다면 throws JPA....Exception으로 하면 변환하지 않고 예외를 던진다.

### Transaction Rollback 주의사항

- 트랜잭션 롤백 >> DB 롤백 >> 수정 자바 객체 롤백 X(영속성 컨텍스트에 남아있음.)
- 스프링은 해당 현상을 미연에 방지하기 위해서 엔티티를 클리어한다. (clear() 함. doRollback 참고)
- OSIV : 영속성의 범위를 트랜잭션보다 넓게할 경우, 여러 트랜잭션이 하나의 영속성 컨텍스트 사용

- 같은 영속성 컨텍스트에서 조회, 저장 엔티티 완전히 같다.
![Persistence Context Scope](https://user-images.githubusercontent.com/18049575/184555036-365befbc-16fb-4f7a-bac8-4c95ed9763b8.png)

- 만약 테스트 클래스에서 @Transactional > 테스트 끝날 때 commit하지 않고 강제로 롤백 > 영속성 컨텍스트 flush하지 않음 > 쿼리 안보임 > 쿼리 보려면 em.flush() 해야함.

- __엔티티를 비교할 때는 비즈니스 키를 활용한 동등성 비교를 하자__

## 프록시 심화 주제
프록시는 원본 엔티티를 상속받아서 만들어진다. 이는 클라이언트가 구분없이 사용할 수 있는 편의를 제공한다. 그러나 기술적 한계 때문에 예상하지 목한 문제 들이 발생한다.

### 영속성 컨텍스트와 프록시
영속성 컨텍스트는 자신이 관리하는 영속 엔티티의 동일성을 보장한다. 그럼 프록시는 조회한 엔티티의 동일성도 보장할까? 답) 그렇다 보장한다.

### 프록시 타입 비교
프록시 엔티티를 상속 받아서 만들어지므로 프록시로 조회한 엔티티의 타입을 비교할 때는 == 를 비교하면 안되고 instanceof를 사용해야 한다

```java
@Test
public void proxy_type_compare {
  Member newMember = new Member(Member.class, "member1");
  em.persist(newMember);
  em.flush();
  em.clear();

  Member refMember = em.getReference(Member.class, "member1");
  
  System.out.println("refMember Type = " + refMember.getClass());

  Assert.assertFalse(Member.class == refMember.getClass()); // false
  Assert.assertTrue(refMember instanceof Member); // true
}

```

### 프록시 동등성 비교
엔티티의 동등성을 비교하려면 비즈니스 키를 사용해서 equals() 메소드를 오버라이딩하고 비교하면 된다. 그런데 IDE나 외부 라이브러리를 사용해서 구현한 equals 메소드로 엔티티를 비교할 때, 비교 대상이 원본 엔티티면 문제가 없지만 프록시면 문제가 발생할 수 있다.

```java
@Entity
@Getter
@Setter
public class Member {
  @Id
  private String id;
  private String name;

  @Override
  public boolean equals(Object obj) {
    if(this == obj) return true;
    if(obj == null) return false;

    if(this.getClass() != obj.getClass()) return false; // 프록시의 경우,
                                                        // instanceof로 비교하여야 한다.

    Member member = (Member) obj;

    if(name != null ? name.equals(member.name) :
    member.name != null) { // 프록시의 경우 null 반환 getName()으로 접근해야한다.
      return false;
    }

    @Override
    public int hashCode() {
      return name != null ? name.hashCode() : 0;
    }
  }
}

@Test
public void 프록시와_동등성_비교() {
  Member saveMember = new Member("member1", "회원");
  em.persist(saveMember);
  em.flush();
  em.clear();

  Member newMember = new Member("member2", "회원1");
  Member refMember = em.getReference(Member.class, "member1");

  Assert.assertTrue(newMember.equals(refMember));
}
```

### 상속관계와 프록시
```java
@Test
public void 부모타입으로_프록시조회() {
  // 테스트 데이터 준비
  Book saveBook = new Book();
  saveBook.setName("jpaBook")
  saveBook.setAuthor("kim");
  em.persist(saveBook);

  em.flush();
  em.clear();

  // 테스트 시작
  Item proxItem = em.getReference(Item.class, saveBook.getId()); // Item Base Proxy
  System.out.println("proxyItem = " + proxyItem.getClass());

  if(proxyItem instanceof Book) {
    System.out.println("proxyItem instanceof Book");
    Book book = (Book) proxyItem; // Item Base Proxy이기 때문에 다운캐스팅도 오류 발생
    System.out.println("Book Author = " + book.getAuthor());
  }

  // 결과검증
  Assert.assertFalse(proxyItem.getClass() == Book.class);
  Assert.assertFalse(proxyItem instanceof Book); // Item 기반 > ClassCastException 발생
  Assert.assertTrue(proxyItem instanceof Item);
}
```

### Visitor Pattern
프록시에 대한 걱정 없이 안전한게 원본 엔티티에 접근할 수 있고 instanceof나 타입캐스팅 없이 코드를 구현할 수 있는 장점이 있다.
- 장점
  1. 프록시에 대한 걱정 없이 안전하게 원본 엔티티에 접근할 수 있다.
  2. instanceof와 타입캐스팅 없이 코드를 구현할 수 있다.
  3. 알고리즘과 객체 구조를 분리해서 구조를 수정하지 않고 새로운 동작을 추가할 수 있다.
- 단점
  1. 너무 복잡하고 더블 디스패치를 사용하기 때문에 이해하기 어렵다.
  2. 객체 구조가 변경되면 모든 Visitor를 수정해야 한다.

