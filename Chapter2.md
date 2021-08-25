2장 JPA 스따뜨
==============

# 객체 매핑 스따뜨

@Entity - 이 클래스를 테이블과 매핑한다고 JPA에게 알려준다. 이렇게 @Entity가 사용된 클래스를 엔티티 클래스라고 한다.  
@Table - 엔티티 클래스에 매핑할 테이블 정보를 알려준다. 이 어노테이션을 생략하면 클래스 이름을 테이블 이름으로 매핑한다.  
@Id - PK를 매핑한다. @Id를 사용한 필드를 식별자 필드라고 한다.  
@Column - 필드를 컬럼에 매핑한다.  
매핑정보가 없다면? - 매핑 어노테이션을 생략하면 필드명을 컬럼명으로 매핑한다. (대소문자 구분하지 않는 DB라고 가정)  

```java
@Entity
@Table(name="MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    private Integer age;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

DATABASE Dialect - SQL은 다음과 같이 표준 SQL인 ANSI SQL이 있으며, ANSI SQL 이외에 각 DBMS Vendor(벤더, 공급업체)인 MS-SQL, Oracle, MySQL, PostgreSQL 에서 자신만의 기능을 추가한 SQL이 있습니다. ANSI SQL이 모든 DBMS에서 공통적으로 사용가능한 핵심 표준 SQL이지만, 여러 제품의 DBMS에서는 자신만의 독자적인 기능을 위해서 추가적인 SQL을 만들었습니다. JPA는 어플리케이션이 직접 JDBC 레벨에서 SQL을 작성하는 것이 아닌 JPA가 직접 SQL을 작성하고 실행하는 형태입니다. 그런데, DBMS 종류별로 사용하는 SQL 언어가 조금씩 다르다면, 다른 부분에 대한 대처가 필요할 것입니다. 예를 들어 JPA로 게시판을 개발할 때 DBMS마다 다른 페이징 방법을 처리할 필요가 있습니다.  
그리고 제품(어플리케이션)을 개발하는데 있어서, 각각 DBMS 벤더별로 다른 모듈을 개발해 주어야 합니다. 만약 고객의 요구에 따라 오라클 DB를 기준으로 작성했던 게시판 프로그램을 MS-SQL에 맞게 추가적으로 개발하려면 엄청난 비용이 들어갈 것입니다.  
그러나 개발자는 JPA를 이용함에 있어서 쿼리를 작성할 필요도 없고, JPA를 사용하더라도 각 DBMS별로 조금씩 다른 SQL 방언을 걱정할 필요도 없습니다. JPA에서는 이를 Dialect라는 추상화된 방언 클래스를 제공하고 각 벤더에 맞는 구현체를 제공하고 있습니다.  
출처: https://dololak.tistory.com/465 [코끼리를 냉장고에 넣는 방법]

# 사용 예제

```java
public class JpaMain {

    public static void main(String[] args) {

        //엔티티 매니저 팩토리 생성
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook"); //jpabook 은 persistence-unit을 설정
        EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성
        /*
        엔티티 매니저는 내부에 데이터베이스 커넥션을 유지하면서 데이터베이스와 통신한다.
        엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드간에 공유하거나 재사용하면 안된다.
        */

        EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득
        /*
        항상 트랜잭션 안에서 데이터 변경을 해야한다. 트랜잭션이 없으면 에러가 발생한다.
        */

        try {

            tx.begin(); //트랜잭션 시작
            logic(em);  //비즈니스 로직
            tx.commit();//트랜잭션 커밋

        } catch (Exception e) {
            e.printStackTrace();
            tx.rollback(); //트랜잭션 롤백
        } finally {
            em.close(); //엔티티 매니저 종료, 엔티티 매니저의 사용이 끝났을 때.
        }

        emf.close(); //엔티티 매니저 팩토리 종료, 애플리케이션이 종료할 때.
    }

    public static void logic(EntityManager em) {

        String id = "id1";
        Member member = new Member();
        member.setId(id);
        member.setUsername("지한");
        member.setAge(2);

        //등록
        em.persist(member);

        //수정
        member.setAge(20); // em.update를 날려야할 것 같으나 JPA가 어떤 엔티티가 변경되었는지 추적하는 기능을 가지고 있다. 알아서 UPDATE SQL을 날린다.

        //한 건 조회
        Member findMember = em.find(Member.class, id);
        System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

        //목록 조회, JPQL (Java Persistence Query Language)
        //JPQL : 엔티티 객체를 대상으로 쿼리를 실행한다. 클래스와 필드로
        //SQL : 데이터베이스 테이블을 대상으로 쿼리를 실행한다.
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
        System.out.println("members.size=" + members.size());

        //삭제
        em.remove(member);

    }
}
```

```java
Member m = new Member();
m.setId("member1");
m.setUsername("조현욱");    // 여기까지 비영속 상태
em.persist(m);            // 객체를 저장, 영속 상태, 1차 캐시에 저장

Member findMember = em.find(Member.class, "member1"); // 1차 캐시에서 조회
/*
 만약 1차 캐시에 없다면 Database에서 조회
 1차 캐시에서는 식별자(엔티티의 Id)가 키이다.
*/
```

# 등록 및 수정 예제  

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpatest");
EntityManager em = emf.createEntitymanager(); 
EntityTransaction transaction = em.getTransacion();

Member m = new Member();
m.setId("member1");
m.setUsername("조현욱");    // 여기까지 비영속 상태

transaction.begin();      // 트랜잭션 시작

em.persist(m);            // 객체를 저장, 영속 상태, 1차 캐시에 저장

transaction.commit();     // 커밋하는 순간 Database에 Insert 쿼리를 보냄

Member findMember = em.find(Member.class, "member1"); // 1차 캐시에서 조회

findMember.setUsername("변경이름");     // 영속 엔티티 수정

em.commit();        // 엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 변경 감지(Dirty checking)

transaction.end();

/*
 만약 1차 캐시에 없다면 Database에서 조회
 1차 캐시에서는 식별자(엔티티의 Id)가 키이다.
*/
```

# 삭제 예제 - 삭제된 엔티티는 재사용하지 말고 자연스럽게 GC의 대상이 되도록 하는 것이 좋다.
```java
Member memberA = em.find(Member.class, "member1");
em.remove(memberA);
```

# Flush - 영속성 컨테스트의 변셩 내용을 데이터베이스에 반영한다.
변경 감지(Dirty Checking)이 동작하여 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해서  
수정된 엔티티를 찾는다. 수정된 엔티티는 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.  
쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송한다.  

        Flush 호출 방법
        1. em.flush() 호출
        2. 트랜잭션 커밋 시 플러시가 자동으로 호출된다.
        3. JPQL 쿼리 실행 시 플러시가 자동 호출된다.

```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);

// 중간에 쿼리 실행
query = em.createQuery("select m from Member m", Member.class); // MemberA, MemberB, MemberC 조회되지 않는다.
List<Member> members = query.getResultList();
```

Flush모드 옵션
em.setFlushMode(FlushModeType.COMMIT); // 플러시 모드 직접설정 (커밋할때만 플러시)
em.setFlushMode(FlushModeType.AUTO); // 커밋이나 쿼리를 실행할 때 플러시(기본값)


준영속은 스킵..

# 영속성 컨텍스트 종료 : close()

```java
public void closeEntityManager() {
    EntityManagerFactory emf = Persistence.createEntityManager("jpabook");
    EntityManager em = emf.createEntityManager();
    EntityTransaction transaction = em.getTransaction();

    transaction.begin();

    MemberA memberA = em.find(Member.class, "memberA");
    MemberB memberB = em.find(Member.class, "memberB");
    
    transaction.commit();

    em.close();
}
```

# 병합 Merge
준영속 상태에서 영속 상태로 바꿀 때 사용.