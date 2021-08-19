3장 영속성 관리
==============

# 엔티티 매니저 팩토리와 엔티티 매니저

## 엔티티 매니저 팩토리 - 이름 그대로 엔티티를 만드는 공장이다. 비용이 크다. 애플리케이션에서 하나를 만들어 공유한다. 여러 스레드에서 동시에 접근해도 안전하게 처리한다. 엔티티 매니저를 생성할 때 커넥션 풀도 생성한다.  

## 엔티티 매니저 - 엔티티 매니저는 엔티티를 관리하는 관리자이다. 저장, 수정, 삭제 등을 처리한다. 앤티티 매니저는 엔티티 매니저 팩토리에서 가져올 수 있다. 보통 하나의 데이터베이스를 사용하는 애플리케이션은 하나의 엔티티 팩토리 매니저를 생성한다. 엔티티 매니저를 공유하면 안된다. 트랜잭션을 시작할 때 커넥션을 흭득한다.  

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpatest");
EntityManager em = emf.createEntitymanager();  
```

## 엔티티의 생명주기 - 비영속, 영속, 준영속, 삭제로 분류됨. 아래의 예시로 정리

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

### 등록 및 수정 예제  

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpatest");
EntityManager em = emf.createEntitymanager(); 
EntityTransaction transaction = em.getTransacion();

Member m = new Member();
m.setId("member1");
m.setUsername("조현욱");    // 여기까지 비영속 상태

transaction.begin();      // 트랜잭션 시작

em.persist(m);            // 객체를 저장, 영속 상태, 1차 캐시에 저장

transaction.commit();     // 커밋하는 순간 영속성 컨텍스트에 모아둔 쿼리를 Database에 쿼리를 보냄. 이를 쓰기 지연이라고 한다.

Member findMember = em.find(Member.class, "member1"); // 1차 캐시에서 조회, 없으면 데이터베이스에서 조회, 조회한 값은 다시 1차캐시에 저장하고 결과 리턴

findMember.setUsername("변경이름");     // 영속 엔티티 수정

em.commit();        // 엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 변경 감지(Dirty checking)

transaction.end();

/*
 만약 1차 캐시에 없다면 Database에서 조회
 1차 캐시에서는 식별자(엔티티의 Id)가 키이다.
*/
```

### 삭제 예제 - 삭제된 엔티티는 재사용하지 말고 자연스럽게 GC의 대상이 되도록 하는 것이 좋다.
```java
Member memberA = em.find(Member.class, "member1");
em.remove(memberA); // 호출하는 순간 영속성 컨텍스트에서 삭제된다. 데이터베이스에서 바로 삭제하지는 않고 삭제 쿼리를 SQL저장소에 저장하고 커밋으로 플러시가 실행되면 실제 데이터베이스에 쿼리를 전달한다.
```

# 영속성 컨텍스트 - 굳이 번역하자면 '엔티티를 영구 저장하는 환경', 엔티티 매니저로 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관함. 영속성 컨텍스트는 엔티티 매니저를 생성할 때 하나 만들어진다. 그리고 엔티티 매니저를 통해서 영속성 컨텍스트에 접근할 수 있고, 관리할 수 있다.

## 영속성 컨텍스트 특징(p95)
1. 영속성 컨텍스트와 식별자 값 : 영속성 컨텍스트는 엔티티를 식별자 값(@Id)으로 구분한다. 따라서 영속 상태는 식별자 값이 반드시 있어야 한다.
2. 영속성 컨텍스트와 데이터베이스 저장 : 영속성 컨텍스트에 엔티티를 저장하면 이 엔티티를 언제 데이터베이스에 보통 커밋하는 순간 영속성 컨텍스트에 새로 저장된 엔티티를 데이터베이스에 반영하는데 이를 플러시(flush)라고 한다.
3. 영속성 컨텍스트가 엔티티를 관리하면 다음과 같은 장점이 있다.
    * 1차 캐시
    * 동일성 보장
    * 트랜잭션을 지원하는 쓰기 지연
    * 변경 감지 Dirty cheking - 플러시 시점에 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해서 저장하는 스냅샷하고 비교하여 변경된 엔티티를 찾는다. Dirty checking은 영속성 컨텍스트에 의해 관리되는 엔티티에만 적용된다. 변경감지로 인해 실행된 UPDATE 쿼리는 모든 필드를 업데이트한다. 전송량이 증가하는 단점이 있지만 모든 필드를 사용하면 쿼리가 항상 같아 애플리케이션 로딩 시에 수정 쿼리를 미리 생성해두고 사용할 수 있다. 또 데이터베이스는 이전에 한번 파싱된 쿼리를 재사용할 수 있다. 필드가 30개 이상이면 @DynamicUpdate 어노테이션을 사용하여 동적으로 UPDATE 쿼리를 생성하도록 하자.
    * 지연 로딩

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
query = em.createQuery("select m from Member m", Member.class); // JPQL을 사용하면 자동으로 플러시가 실행된다.
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


