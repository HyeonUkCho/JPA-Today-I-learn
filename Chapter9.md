9장 값 타입
========================

## 기본값 타입
```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;
    private int age;

    /*
    Member 엔티티는 id라는 식별자 값도 가지고 생명주기도 있지만 값 타입인 name, age 속성은 식별자 값도 없고 생명주기도 회원 엔티티에 의존한다.
    따라서 회원 엔티티 인스턴스를 제거하면 name, age 값도 제거된다.
    그리고 값 타입은 공유하면 안된다. 예를 들어 다른 회원 엔티티의 이름을 변경한다고 해서 나의 이름까지 변경되는 것은 안되기 때문이다.
    */
}
```

## 임베디드 타입
- 새로운 값 타입을 직접 정의해서 사용할 수 있는데, JPA에서는 이것을 임베디드 타입이라 한다.
- 중요한 것은 직접 정의한 임베디드 타입도 int, String 처럼 값 타입이라는 것이다.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;
    
    //근무기간
    @Temporal(TemporalType.Date) java.util.Date startDate;
    @Temporal(TemporalType.Date) java.util.Date endDate;

    // 집주소 관련
    private String city;
    private String street;
    private String zipcode;
    
    //...
}
```

위 소스를 임베디드 타입을 이용하여 정의하면

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;
    
    //근무기간
    @Embedded Period workPeriod;
    // 집주소 관련
    @Embedded Address homeAddess;

    //...
}

@Embeddable
public class Period {
    @Temporal(TemporalType.Date) java.util.Date startDate;
    @Temporal(TemporalType.Date) java.util.Date endDate;

    public boolean isWork(Date date) {
        // .. 값 타입을 위한 메소드 정의 가능
    }
}

@Embeddable
public class Address {
    
    @Column(name="city") //매핑할 컬럼 정의 가능
    private String city;
    private String street;
    private String zipcode;
    //...
}
```


## 임베디드 타입과 연관관계
```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;
    
    //근무기간
    @Embedded Period workPeriod;
    // 집주소 관련
    @Embedded Address homeAddess;
    @Embedded PhoneNumber phoneNumber;

    //...
}

@Embeddable
public class Period {
    @Temporal(TemporalType.Date) java.util.Date startDate;
    @Temporal(TemporalType.Date) java.util.Date endDate;

    public boolean isWork(Date date) {
        // .. 값 타입을 위한 메소드 정의 가능
    }
}

@Embeddable
public class Address {
    
    @Column(name="city") //매핑할 컬럼 정의 가능
    private String city;
    private String street;
    private String zipcode;
    //...
}

@Embeddable
public class PhoneNumber {
    
    String areaCode;
    String localNumber;
    @ManyToOne PhoneServiceProvider provider; // 엔티티 참조
    //...
}

@Entity
public class PhoneServiceProvider {
    @Id String name;
    //...
}
```

## @AttributeOverride:속성 재정의
- 임베디드 타입에 정의된 매핑정보를 재정의하려면 엔티티에 @AttributeOverride를 사용하면 된다. 예를 들어 회원에게 주소가 하나 더 필요하면 어떻게?
```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;
    
    //근무기간
    @Embedded Period workPeriod;
    // 집주소 관련
    @Embedded Address homeAddess;
    //@Embedded Address companyAddress; // 이렇게 하면 테이블에 컬럼명이 중복이 일어남.

    @Embedded
    @AttributeOverride({
        @AttributeOverride(name="city", column=@Column(name="COMPANY_CITY"),
        @AttributeOverride(name="street", column=@Column(name="COMPANY_STREET"),
        @AttributeOverride(name="zipcode", column=@Column(name="COMPANY_ZIPCODE"))
    })
    Address companyAddress;

    //...
}
```

## 임베디드 타입과 null
- 임베디드 타입이 null이면 매핑한 컬럼 값은 모두 null이 된다.

## 값 타입과 불변 객체
- 값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다. 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.
## 값 타입 공유 참조
- 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험하다. 아래의 예를 보자
```java

member1.setHomeAddress(new Address("OldCity"));
Address address = member1.getHomeAddress();

address.setCity("NewCity"); // 회원1의 address 값을 공유해서 사용, 회원1과 회원2가 같은 Address를 참조하기 때문에 회원1도 NewCity가 되어 있다. 회원 둘다 update sql 실행된다.
member2.setHomeAddress(address);

```

__이러한 문제는 임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본 타입이 아니라 객체 타입이라는 점이다. 자바 객체에 값을 대입하면 항상 참조 값을 전달한다.__
__기본타입은 값을 복사! __객체면 참조를 넘긴다! 값을 수정할 수 없게 set함수를 없애는 것도 하나의 방법이다.__
```java

member1.setHomeAddress(new Address("OldCity"));
Address address = member1.getHomeAddress();

// 이렇게 해야한다.
Address newAddress = address.clone();
newAddress.setCity("NewCity"); 
member2.setHomeAddress(address);

```

## 불변 객체

- 즉시 로딩 : 즉시 로딩을 사용하려면 @ManyToOne의 fetch 속성을 FetchType.EAGER로 지정
```java

@Entity
public class Member {
    //...
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    //...
}

Member member = em.find(Member.class, "member1");
Team team = member.getTeam(); // 객체 그래프 탐색

```
조인 쿼리로 한방에 조회함.
```sql

SELECT 
    M.MEMBER_ID AS MEMBER_ID,
    M.TEAM_ID AS TEAM_ID,
    M.USERNAME AS USERNAME,
    T.TEAM_ID AS TEAM ID,
    T.NAME AS NAME,
FROM 
    MEMBER M LEFT OUTER JOIN TEAM T
    ON M.TEAM_ID = T.TEAM_ID
WHERE
    M.MEMBER_ID = 'member1'

```

## __중요__ NULL 제약조건과 JPA 조인 전략 
- 위 예제에서 LEFT OUTER JOIN을 한 부분을 잘 봐야한다. 현재 히ㅗ원 테이블에 TEAM_ID 외래 키는 NULL 값을 허용하고 있다. 따라서 팀에 소속되지 않은 회원이 있을 가능성이 있다. 팀에 소속하지 않은 회원과 팀을 내부 조인하면 팀은 물론이고 회원 데이터도 조회할 수 없다. JPA는 이런 상황을 고려하여 외부 조인을 한다. 하지만 외부 조인보다 내부 조인이 성능과 최적화에서 유리한다. 그럼 내부 조인을 사용하려면 어떻게 해야할까?
외래 키에 NOT NULL 제약조건을 설정하면 값이 있는 것을 보장한다. 따라서 이때는 내부 조인만 사용해도 된다.
JPA에게도 이런 사실을 알려줘야하기 때문에 @JoinColumn에 nullable = false을 설정해서 이 외래 키는 NULL 값을 허용하지 않는다고 알려주면 JPA는 외부 조인 대신 내부 조인을 사용한다. 
- JoinColumn(nullable = true) : NULL 허용(default), 외부 조인 사용 
- JoinColumn(nullable = false) :  NULL 허용하지 않음, 내부 조인 사용


- 지연 로딩 : 지연 로딩을 사용하려면 @ManyToOne의 fetch 속성을 FetchType.LAZY로 지정
```java

@Entity
public class Member {
    //...
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    //...
}

Member member = em.find(Member.class, "member1");
Team team = member.getTeam(); // 객체 그래프 탐색, 프록시 객체를 team에 넣어둔다.
team,getName(); // 팀 객체 실제 사용

```

## 상황에 따라 즉시 로딩, 지연 로딩을 사용하자. 책에서는 모두 지연 로딩으로 개발을 하고 나중에 자주 같이 많이 사용되는 부분은 즉시 로딩으로 바꾸는 것을 권한다.

## 기본 패치 전략
- @ManyToOne, @OneToOne : 즉시로딩
- @OneToMay, @ManyToMany : 지연로딩
- 컬렉션을 가지면 지연 로딩을 기본값으로 갖는다.

## 영속성 전이 : CASCADE
- 영속성 전이 : 쉽게 말해 연관관계를 가진 엔티티끼리 저장될 때 같이 저장되고 삭제될 때 같이 삭제되게 해주는 방법이다.
- 저장은 CascadeType.PERSIST, 삭제는 CascadeType.REMOVE

## 고아 객체 : 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능을 제공.

