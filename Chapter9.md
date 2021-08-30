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
- 값 타입은 부작용 걱정 없이 사용할 수 있어야 한다. 부작용이 일어나면 값 타입이라 찰 수 없다. 객체를 불변하게 만들면 값을 수정할 수 없으므로 부작용을 원천 차단할 수 있다. 따라서 값 타입은 될 수 있으면 불변 객체로 설계해야 한다.
```java

@Embeddable
public class Address {
    private String city;
    protected Address() { //JPA에서 기본 생성자는 필수다.

    }
    // 생성자로 초기 값을 설정한다.
    public Address(String city) {
        this.city = city;
    }

    public String getCity() {
        return city;
    }

    // Setter는 만들지 않는다.
}
```

## 값 타입의 비교
- 동일성 비교 : 인스턴스의 참조 값을 비교, == 사용
- 동등성 비교 : 인스턴스의 값을 비교, equals 사용