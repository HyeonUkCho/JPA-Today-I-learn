# 엔티티 매니저 팩토리와 엔티티 매니저

엔티티 매니저 - 엔티티 매니저는 엔티티를 관리하는 관리자이다. 저장, 수정, 삭제 등을 처리한다.
            앤티티 매니저는 엔티티 매니저 팩토리에서 가져올 수 있다. 보통 하나의 데이터베이스를 사용하는 애플리케이션은 하나의 엔티티 팩토리 매니저를 생성한다.  

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpatest");
EntityManager em = emf.createEntitymanager();  
```

영속성 컨텍스트 - 굳이 번역하자면 '엔티티를 영구 저장하는 환경', 엔티티 매니저로 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관함.  
              영속성 컨텍스트를 엔티티 매니저를 생성할 때 하나 만들어진다. 그리고 엔티티 매니저를 통해서 영속성 컨텍스트에 접근할 수 있고, 관리할 수 있다.

# 엔티티의 생명주기 - 비영속, 영속, 준영속, 삭제로 분류됨. 아래의 예시로 정리

```java
Member m = new Member();
m.setId("member1");
m.setUsername("조현욱");    // 여기까지 비영속 상태
em.persist(m);            // 객체를 저장, 영속 상태

em.detach(m);
em.close();
em.clear();  // 이 3가지는 영속성 컨텍스트에서 관리하지 않는 준영속 상태

em.remove(member); // 객체를 삭제한 상태, 삭제 상태
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