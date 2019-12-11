7장 고급 매핑
==============

- 상속 관계 매핑 : 일단 패스
- @MappedSuperclass
- 복합 키와 식별 관계 매핑
- 조인 테이블
- 엔티티 하나에 여러 테이블 매핑하기


## @MappedSuperclass
- 부모 클래스는 테이블과 매핑하지 않고 부모 클래스를 상속받는 자식 클래스에게 매핑 정보만 제공하고 싶으면 해당 어노테이션을 사용하면 된다.
- 추상 클래스와 비슷한데 @Entity는 실제 테이블과 매핑되지만 해당 어노테이션은 매핑되지 않는다. 단순히 매핑 정보를 상속할 목적으로만 사용된다.

```java

@MappedSuperclass
public abstract class BaseEntity { // 이 클래스를 직접 생성해서 사용할 일은 없으므로 추상 클래스로 만드는 것을 권장
    @Id
    @GeneratedValue
    private Long id;

    private String name;
    ...
}

@Entity
public class Member extends BaseEntity {
    // ID 상속
    // NAME 상속
    private String email
    ...
}

@Entity
public class Seller extends BaseEntity {
    // ID 상속
    // NAME 상속
    private String shopName;
    ...
}

@Entity
// 부모에게 상속받은 id 속성의 컬럼명을 MEMBER_ID로 재정의
@AttributeOveride(name = "id", column = @Column(name = "MEMBER_ID"))
public class Member extends BaseEntity { ... }

@Entity
@AttributeOverrides({
    // 부모에게 상속받은 id 속성의 컬럼명을 MEMBER_ID 로 재정의
    @AttributeOveride(name = "id", column = @Column(name = "MEMBER_ID")) 
    // 부모에게 상속받은 id 속성의 컬럼명을 MEMBER_NAME 으로 재정의
    @AttributeOveride(name = "name", column = @Column(name = "MEMBER_NAME"))
})
```
- __@MappedSuperclass 는 테이블과는 관계가 없고 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모아주는 역할을 한 뿐이다. ORM 에서 이야기하는 진정한 상속 매핑은 이전에 학습한 객체 상속을 데이터베이스의 슈퍼타입 서브타입 관계와 매핑하는 것이다. 등록일자, 수정일자, 등록자, 수정자와 같은 여러 엔티티에서 공통으로 사용하는 속성을 효과적으로 관리할 수 있다.__



## 복합 키와 식별 관계 매핑
- 데이터베이스 테이블 사이에 관계는 외래 키가 기본 키에 포함되는지 여부에 따라 식별 관계와 비식별 관계로 구분한다.

1. 식별 관계 : 부모 테이블의 기본 키를 내려받아서 자식 테이블의 기본 키 + 외래 키로 사용하는 관계다.

2. 비식별 관계 : 부모 테이블의 기본 키를 받아서 자식 테이블의 외래 키로만 사용하는 관계다. 또 비식별 관계에는 외래 키를 NULL을 허용하는지에 따라 필수적 비식별 관계와 선택적 비식별 관계로 나눈다. (주로 비식별 관계를 사용하고 꼭 필요한 곳에만 식별 관계를 사용하는 추세.)

## 복합 키 : 비식별 관계 매핑
- JPA에서 식별자를 둘 이상 사용하려면 별도의 식별자 클래스를 만들어야 한다. JPA는 영속성 컨텍스틍 엔티티를 보관할 때 엔티티의 식별자를 키로 사용한다. 그리고 식별자를 구분하기 위해 equals와 hashCode를 사용해서 동등성 비교를 한다. 그런데 식별자 필드가 하나일 때는 보통 자바의 기본 타입을 사용하므로 문제가 없지만, 식별자가 2개 이상이면 별도의 식별자 클래스를 만들고 그곳에 equals와 hashCode를 구현해야 한다. @IdClass(데이터베이스에 가까운 방법)와 @EmbeddedId(객체지향에 가까운 방법) 2가지 방법이 있다.

## @IdClass
```java
@Entity
@IdClass(ParentId.class)
public class Parent {
    @Id
    @Column(name = "PARENT_ID1")
    private String id1; // ParentId.id1과 연결

    @Id
    @Column(name = "PARENT_ID2")
    private String id2; // ParentId.id2과 연결

    private String name;
    ...
}

// @IdClass를 사용할 때 식별자 클래스는 다음 조건을 만족해야 한다.
// 1. 식별자 클래스는 public 이어야 한다.
public class ParentId implements Serialize { // 2. Serializable 인터페이스를 구현해야 한다.
    
    // 3. 식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야 한다.
    private String id1;
    private String id2;

    public ParentId() { // 4. 기본 생성자가 있어야 한다.

    }

    public ParentId(String id1, String id2) {
        this.id1 = id1;
        this.id2 = id2;
    }

    @Override
    public boolean equals(Object o) { ... } // 5. equals가 구현되어 있어야 한다.

    @Override
    public boolean hashCode() { ... } // 6. hashCode가 구현되어 있어야 한다.

}
```

- 엔티티 저장
```java

Parent parent = new Parent();
parent.setId1("id1");
parent.setId2("id2");
parent.setName("parentName");
em.persist(Parent);

```


- 비식별 관계의 경우 자식 클래스
```java

@Entity
public class Child {
    @Id
    private String id;

    // 부모 테이블의 기본 키 컬럼이 복합 키이므로 자식 테이블의 외래 키도 복합 키다.
    // 따라서 외래 키 매핑 시 여러 컬럼을 매핑해야 하므로 @JoinColumn 사용하고
    // 각각의 외래 키 컬럼을 @JoinColumn으로 매핑한다.
    @ManyToOne
    @JoinColumns({
        @JoinColumn(name="PARENT_ID1", referencedColumnName = "PARENT_ID1"),
        @JoinColumn(name="PARENT_ID2", referencedColumnName = "PARENT_ID2")
    }) 
    private Parent parent;
}

```

## @EmbeddedId
```java

@Entity
public class Parent {
    @EmbeddedId
    private ParentId id; // 식별자를 직접 사용하고 어노테이션 추가

    private String name;
    ...
}

// 아래의 조건 필요
@Embeddable // 1. @Embeddable 어노테이션 추가
// 2. 식별자 클래스는 public 이어야 한다.
public class ParentId implements Serializable { // 3. Serializable 구현

    @Column(name = "PARENT_ID1")
    private String id1;

    @Column(name = "PARENT_ID2")
    private String id2;

    private String name;

    public ParentId() { // 4. 기본 생성자가 있어야 한다.

    }

    public ParentId(String id1, String id2) {
        this.id1 = id1;
        this.id2 = id2;
    }

    @Override
    public boolean equals(Object o) { ... } // 5. equals가 구현되어 있어야 한다.

    @Override
    public boolean hashCode() { ... } // 6. hashCode가 구현되어 있어야 한다.
    ...
}

```

- 엔티티 저장
```java

Parent parent = new Parent();
ParentId parentId = new ParentId("id1", "id2");
parent.setParentId(parentId);
parent.setName("parentName");
em.persist(Parent);

```

## 복합 키 구성 시에 equals와 hashCode 구현해야하는 이유
- 자바의 모든 클래스는 기본으로 Object를 상속하면서 equals의 참조 값 비교인 == 비교 (동일성)을 하기 때문에 식별자로 비교를 하기 위해 꼭 구현해야 한다.

## 복합 키 : 식별 관계 매핑

