4장 엔티티 맵핑
==============

# @Entity
- 테이블과 맵핑될 클래스에 필수적으로 붙여야한다.  
- 기본생성자는 필수!  
- final, enum, interface, inner 클래스는 사용 불가!  
- 저장할 필드에 final 사용 불가!  

```java

// 자바는 생성자가 없으면 아래와 같이 기본 생성자를 자동으로 만든다.
public Member() {} 

// 아래와 같이 생성자를 하나 이상 만들면 자바는 기본생성자를 자동으로 만들지 않는다. 아때는 기본생성자를 만들어야한다.
public Member(String name) {
    this.name = name;
}

```

# @Table
- 엔티티와 맵핑할 테이블을 지정할 경우에 사용  
- 생략하면 엔티티 이름을 테이블 이름으로 사용한다.  
- 속성 정리  
    1. name : 맵핑할 테이블 이름.
    2. catalog : catalog 기능이 있는 데이터베이스에서 catalog 맵핑.
    3. schema : schema 기능이 있는 데이터베이스에서 schema 맵핑.
    4. uniqueConstraints : DDL 생성 시에 유니크 제약조건을 만든다. 2개 이상의 복합 유니크 제약조건을 만들 수 있다. 참고로 이 기능은 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용.


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

    @Enumerate(EnumType.STRING) // 회원의 타입을 enum 으로 구분.
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP) // 날짜 타입을 @Temporal 사용
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob // CLOB, BLOB 사용 가능
    private String description;

    //Getter, Setter....
}

public enum RoleType {
    ADMIN, USER
}
```

# 데이터베이스 스키마 자동생성
- JPA는 데이터베이스 스키마 자동생성 지원.  
- 데이터베이스 방언을 통해 자동생성.  
- 속성값으로 애플리케이션 생성 시에 자동생성 옵션 줄 수 있음.  
- 책, P127 참조  
- 주의사항 : 개발 초기 단계에는 create나 update, 테스트 서버는 update, validate, 운영서버는 validate, none 권장.  


# DDL 생성 기능

```java
@Entity
@Table(name="MEMBER")
public class Member {
    @Id
    @Column(name="ID")
    private String id;

    @Column(name="NAME", nullable=false, length=10) // null 여부 및 길이 지정
    String username;

    ....
}
```

# 유니크 제약조건 추가

```java
@Entity
@Table(name="MEMBER", uniqueConstraints = {@UniqueConstraint(
    name = "NAME_AGE_UNIQUE",
    columnName = {"NAME", "AGE"} // unique 조건 추가.
)})
public class Member {
    @Id
    @Column(name="ID")
    private String id;

    @Column(name="NAME", nullable=false, length=10) // null 여부 및 길이 지정
    String username;

    ....
}
```


# 기본 키 맵핑

- 직접할당 : 기본 키 생성을 데이터베이스에 위임
- 자동생성 : 대리키 사용방식
    - IDENTITY : 기본 키 생성을 데이터베이스에 위임  
    - SEQUENCE : 데이터베이스 시퀀스를 사용해서 기본 키를 할당.  
    - TABLE    : 키 생성 테이블 사용.  

1. 기본 키 직접 할당 전략

```java
@Id
@Column(name = "id")
private String id;
```
@Id 적용 가능 자바 타입 - 자바 기본형, 자바 Wrapper형, String, ....  
em.persist() 저장 전, 기본 키 할당 필요~  

2. IDENTITY 전략 - AUTO_INCREMENT, SEQUENCE와 같이 데이터베이스의 기능으로 사용. INSERT 한 후에 기본 키 조회 가능.

```java
@Entity
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}

3. SEQUENCE 전략 - 오라클, PostgreSQL, DB2, H2 에서 사용 가능. 시퀀스 사용코드는 IDENTITY 전략과 같지만 내부 동작 방식은 다름. 시퀀스는 선조회후커밋이고 IDENTITY 전략은 선커밋후조회.  
```java
@Entity
@SequenceGenerator(
    name = "BOARD_SEQ_GENERATOR", // 식별자 생성기 이름
    sequenceName = "BOARD_SEQ", // 맵핑 시퀀스 이름
    initialValue = 1, // 초기값 (default = 1)
    allocationSize = 1 // 호출될 때, 증가하는 수 (default = 50)
)
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUNCE,
                    generator = "BOARD_SEQ_GENERATOR" )
    private Long id;
}
```

4. 테이블 전략 - P139 확인 하기.

5. AUTO 전략 - GenerationType.AUTO 사용.  @GeneratedValue의 default 값.  


# 필드와 컬럽 맵핑 : 레퍼런스

1. @Column : 객체 필드와 테이블 컬럼에 맵핑. name과 nullable 주로 사용
    - name : 필드와 맵핑할 테이블의 컬럼 이름
    - nullable : null 허용 여부 (default : true)
    - length : 길이 Set
    - unique : @Table의 uniqueConstraint와 같지만 한 컬럼의 제약조건 간단히 걸 때, 사용

2. @Enumerated : enum 사용할 때.
    - value : EnumType.ORDINAL --> enum의 순서를 데이터베이스에 저장. (default)
            : EnumType.STRING  --> enum의 이름을 데이터베이스에 저장. (권장)

3. @Temporal : 날짜 타입 java.util.Date, java.util.Calendar 맵핑.

4. @Lob : 데이터베이스에 BLOB과 CLOB 맵핑.

5. @Transient : 저장, 조회 하지 않는 값. 객체에 임시로 저장할 때 사용.  
