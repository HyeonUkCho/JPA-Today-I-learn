8장 프록시와 연관관계 관리
========================
- 프록시와 즉시로딩, 지연로딩 : 객체는 객체 그래프로 연관된 객체들을 탐색한다. 그런데 객체가 데이터베이스에 저장되어 있으므로 연관된 객체를 마음껏 탐색하기는 어렵다. JPA 구현체들은 이 문제를 해결하려고 프록시라는 기술을 사용한다. 프록시를 사용하면 연관된 객체를 처음부터 데이터베이스에서 조회하는 것이 아니라, 실제 사용하는 시점에 데이터베이스를 조회할 수 있다. 하지만 자주 함께 사용하는 객체들은 조인을 사용해서 함께 조회하는 것이 효과적이다.
- 영속성 전이와 고아 객체 : JPA는 연관된 객체를 함께 저장하거나 함께 삭제할 수 있는 영속성 전이와 고아 객체 제거라는 편리한 기능을 제공한다.

## 프록시
- 지연로딩 기능을 사용하기 위한 가짜 객체를 프록시 객체라고 한다.

## 프록시 기초
- EntityManager.find()를 사용하면 일단 영속성 컨텍스트를 확인하고 없으면 데이터베이스를 조회한다. 이렇게 엔티티를 직접 조회하면 조회한 엔티티를 실제 사용하든 사용하지 않던 데이터베이스를 조회한게 된다. 엔티티를 실제 사용하는 시점까지 데이터베이스 조회를 미루고 싶으면 EntityManager.getReference() 메소드를 사용하면 된다.  

```java
Member member = em.find(Member.class, "member1");  
Member member = em.getReference(Member.class, "member1"); // 이 메소드를 호출할 떄 JPA는 데이터베이스를 조회하지 않고 실제 엔티티 객체도 생성하지 않는다. 대신에 데이터베이스 접근을 위임한 프록시 객체를 반환한다.

```

## 프록시의 특징
- 프록시 클래스를 실제 클래스를 상속 받아서 만들어지므로 실제 클래스와 겉 모양이 같다. 따라서 사용하는 입장에서는 이것이 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 된다.
- 프록시 객체는 실제 객체에 대한 참조(target)을 보관한다. 그리고 프록시 객체의 메소르를 호출하면 프록시 객체는 실제 객체의 메소드를 호출한다.

## 프록시 객체의 초기화
- 프록시 객체는 member.getName()처럼 실제 사용될 떄 데이터베이스를 조회해서 실제 엔티티 객체를 생성하는데 이것을 프록시 객체의 초기화라 한다.
```java
// MemberProxy 반환
Member member = em.getReference(Member.class, "id1");
member.getName(); // 1. getName() 실제 데이터 조회

```
```java
// MemberProxy 반환
class MemberProxy extends Member {

    Member target = null;

    public String getName() {
        if(target == null) {
            // 2. 초가화 요청, 실제 엔티티가 생성되어 있지 않으면 영속성 컨텍스트에 실제 엔티티 생성을 요청하는데 이것을 초기화라고 한다.
            // 3. DB 조회, 영속성 컨텍스트는 데이터베이스를 조회해서 실제 엔티티 객체를 생성한다.
            // 4. 실제 엔티티 생성 및 참조 보관
            this.target = ....;
        }
        // 5. target.getName(); 실제 반환
        return target.getName();
    }

}

```

## 프록시의 특징
- 프록시 객체는 처음 사용할 때 한 번만 초기화된다.
- 프록시 객체를 초기화한다고 프록시 객체가 실제 엔티티로 바뀌는 것은 아니다. 프록시 객체가 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근할 수 있다.
- 프록시 객체는 원본 엔티티를 상복받은 객체이므로 타입 체크 시에 주의해서 사용해야 한다.
- __영속성 컨텍스트에 찾는 엔티티가 이미 있으면 데이터베이스를 조회할 필요가 없으므로 em.getRefernce()를 호출해도 프록시가 아닌 식제 엔티티를 반환한다.__
- 초기화는 영속성 컨텍스트의 도움을 받아야 가능한다. 따라서 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태의 프록시를 초기화하면 문제가 발생한다. 하이버네이트는 LazyInitianlizationException 예외를 발생시킨다.


- 준영속 상태와 초기화 예제
```java

//MemberProxy 반환
Member member = em.getReference(Member.class, "id1");
transaction.commit();
em.close();

member.getName(); // LazyInitianlizationException 발생

```

## 프록시와 식별자
```java

//MemberProxy 반환
Member member = em.getReference(Member.class, "id1"); // 식별자 보관
member.getId(); // 초기화되지 않음. 
// @Access(AccessType.PROPERTY) : 해당 어노테이션을 사용하면 초기화 X
// @Access(AccessType.FIELD) : getId()가 id만을 조회하는 메소드인지 다른 필드까지 활용해서 어떤 일을 하는 메소드인지 알지 못하므로 프록시 객체를 초기화한다.

// 프록시는 다음 코드처럼 연관관계를 설정할 때 유용하게 사용할 수 있다.
Member member = em.find(Member.class, "member1");
Team team = em.getReference(Team.class, "team1");
member.setTeam(team); // 연관관계를 설정할 때는 식별자 값만 사용하므로 프록시를 사용하면 데이터베이스 접근 횟수를 줄일 수 있다. 참고로 연관관계를 설정할 때는 엔티티 접근 방식을 필드로 설정해도 프록시를 초기화하지 않는다.

```

## 프록시 확인
```java

boolean isLoad = em.getEntityManegerFactory()
                    .getPersistenceUnitUtil().isLoaded(entity);
// 또는 boolean isLoad = emf.getPersistenceUnitUtil().isLoaded(entity);
System.out.println("isLoad = " + isLoad);

System.out.println("memberProxy = " + member.getClass().getName());
// result : memberProxy = jpaboot.domain.Member_$$_javassis_0 <-- 프록시임


```


