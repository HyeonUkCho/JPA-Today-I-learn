7장 고급 매핑
==============

- 상속 관계 매핑 (일단 패스..필요할 때 찾아보고 정리하기...)
- @MappedSuperclass
- 복합 키와 식별 관계 매핑
- 조인 테이블 (일단 패스..필요할 때 찾아보고 정리하기...)
- 엔티티 하나에 여러 테이블 매핑하기 (일단 패스..필요할 때 찾아보고 정리하기...)

## 상속 관계 매핑

- 조인 전략 : 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 기본 키 + 외래 키로 사용하는 전략이다. 따라서 조회할 때, 자주 사용한다. 이 전략을 주의할 점은 객체는 타입으로 구분할 수 있지만 테이블은 타입의 개념이 없다. 따라서 타입을 구분하는 컬럼을 추가해야 한다.
    * 장점 : 테이블이 정규화된다. 외래 키 참조 무결성 제약조건을 활용할 수 있다. 저장공간을 효율적으로 사용한다.  
    * 단점 : 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있다. 조회 쿼리가 복잡하다. 데이터를 등록할 INSERT SQL을 두 번 실행한다.  

```java

@Entity
@Inheritance(strategy = InheritanceType.JOINED) // 매핑 전략 지정
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;        //이름
    private int price;          //가격
    private int stockQuantity;  //재고수량

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<Category>();

    //Getter, Setter
    
}

@Entity
@DiscriminatorValue("A") // A가 저장됨.
public class Album extends Item {

    private String artist;
    private String etc;


    //Getter, Setter
}

@Entity
@DiscriminatorValue("M") // M이 저장됨.
public class Movie extends Item {

    private String director;
    private String actor;

    //Getter, Setter
}

```

- 단일 테이블 전략 : 테이블 하나만 사용한다. 그리고 구분 컬럼으로 어떤 자식 데이터가 저장되었는지 구분한다. 조회할 때 조인을 사용하지 않으므로 일반적으로 가장 빠르다. 이 전략을 사용할 때 주의점은 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다는 점이다.
    * 장점 : 조인 필요 없으므로 일반적으로 조회 성능이 빠르다. 조회 쿼리가 단순하다.  
    * 단점 : 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다. 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다. 오히려 조회 성능이 느려질 수 있다.

```java

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE) // 매핑 전략 지정
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;        //이름
    private int price;          //가격
    private int stockQuantity;  //재고수량

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<Category>();

    //Getter, Setter
    
}

@Entity
@DiscriminatorValue("A") // A가 저장됨.
public class Album extends Item {

    private String artist;
    private String etc;


    //Getter, Setter
}

@Entity
@DiscriminatorValue("M") // M이 저장됨.
public class Movie extends Item {

    private String director;
    private String actor;

    //Getter, Setter
}

```

- 구현 클래스마다 테이블 전략 : 자식 엔티티마다 테이블을 만든다. 그리고 자식 테이블 각각에 필요한 컬럼이 모두 있다. (일반적으로 추천 X)
    * 장점 : 서브 타입을 구분해서 처리할 때 효과적이다. not null 제약 조건을 사용할 수 있다.
    * 단점 : 여러 자식 테이블을 함께 조회할 때 성능이 느리다. (SQL에 UNION을 사용해야 한다.) 자식 테이블을 통합해서 쿼리하기 어렵다.

```java

@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS) // 매핑 전략 지정
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;        //이름
    private int price;          //가격
    private int stockQuantity;  //재고수량

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<Category>();

    //Getter, Setter
    
}

@Entity
public class Album extends Item {

    private String artist;
    private String etc;


    //Getter, Setter
}

@Entity
public class Movie extends Item {

    private String director;
    private String actor;

    //Getter, Setter
}

```

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
- __@MappedSuperclass 는 테이블과는 관계가 없고 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모아주는 역할을 한 뿐이다. ORM 에서 이야기하는 진정한 상속 매핑은 이전에 학습한 객체 상속을 데이터베이스의 슈퍼타입 서브타입 관계와 매핑하는 것이다. 등록일자, 수정일자, 등록자, 수정자와 같은 여러 엔티티에서 공통으로 사용하는 속성을 효과적으로 관리할 수 있다. 해당 클래스는 실제로 사용하지 않기 때문에 추상 클래스로 만드는 것을 추천한다.__


## 복합 키와 식별 관계 매핑
- 데이터베이스 테이블 사이에 관계는 외래 키가 기본 키에 포함되는지 여부에 따라 식별 관계와 비식별 관계로 구분한다.

1. 식별 관계 : 부모 테이블의 기본 키를 내려받아서 자식 테이블의 기본 키 + 외래 키로 사용하는 관계다.

2. 비식별 관계 : 부모 테이블의 기본 키를 받아서 자식 테이블의 외래 키로만 사용하는 관계다. 또 비식별 관계에는 외래 키를 NULL을 허용하는지에 따라 필수적 비식별 관계와 선택적 비식별 관계로 나눈다. (주로 비식별 관계를 사용하고 꼭 필요한 곳에만 식별 관계를 사용하는 추세.)

## 복합 키 : 비식별 관계 매핑
- JPA에서 식별자를 둘 이상 사용하려면 별도의 식별자 클래스를 만들어야 한다. JPA는 영속성 컨텍스틍 엔티티를 보관할 때 엔티티의 식별자를 키로 사용한다. 그리고 식별자를 구분하기 위해 equals와 hashCode를 사용해서 동등성 비교를 한다. 그런데 식별자 필드가 하나일 때는 보통 자바의 기본 타입을 사용하므로 문제가 없지만, 식별자가 2개 이상이면 별도의 식별자 클래스를 만들고 그곳에 __equals와 hashCode를 구현해야 한다.__ @IdClass(데이터베이스에 가까운 방법)와 @EmbeddedId(객체지향에 가까운 방법) 2가지 방법이 있다.

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
자식에 매핑
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
    public boolean equals(Object o) { ... } // 5. equals가 구현되어 있어야 한다. 보통 모든 필드를 사용한다.

    @Override
    public boolean hashCode() { ... } // 6. hashCode가 구현되어 있어야 한다. 보통 모든 필드를 사용한다.
    ...
}

```

- 엔티티 저장
```java

Parent parent = new Parent();
ParentId parentId = new ParentId("id1", "id2");
parent.setParentId(parentId); // 실제로 저장
parent.setName("parentName");
em.persist(Parent);

```

## 복합 키 구성 시에 equals와 hashCode 구현해야하는 이유
- 자바의 모든 클래스는 기본으로 Object를 상속하면서 equals의 참조 값 비교인 == 비교 (동일성)을 하기 때문에 식별자로 비교를 하기 위해 꼭 구현해야 한다.

## 복합 키 : 식별 관계 매핑
- 식별 관계에서 자식 테이블은 부모 테이블의 기본 키를 포함해서 복합 키를 구성해야 하므로 @IdClass나 @EmbeddedId를 사용해서 식별자를 매핑해야 한다.

## 부모, 자식, 손자로 계속 기본 키를 전달하는 식별 관계 IdClass 사용 예제

```java

// Parent
@Entity
public class Parent {

    @Id @Column(name="PARENT_ID")
    private String id;

    private String name;
    ...
}

// Child
@Entity
@IdClass(ChildId.class)
public class Child {

    @Id
    @ManyToOne(name="PARENT_ID")
    public Parent parent;

    @Id @Column(name="CHILD_ID")
    private String childId;

    private string name;
    ...
}

//ChildId
public class ChildId implements Serializable {

    private String parent; // Child.parent 매핑
    private String childId; //Child.childId 매핑

    //equals, hashCode
    ...
}

// GrandChild
@Entity
@IdClass(GrandChildId.class)
public class GrandChild {

    @Id
    @ManyToOne
    @JoinColumns ({
        @JoinColumn(name="PARENT_ID"),
        @JoinColumn(name="CHILD_ID")
    })
    private Child child;

    @Id @Column(name="GRANDCHILD_ID")
    private String id;

    private String name;
    ...

}

// GrandChildId
public class GrandChildId implements Serializable {

    private ChildId child; // GrandChild.child 매핑
    private String id; // GrandChild.id 매핑

    // equals, hashCode
    ...
}
```

## 부모, 자식, 손자로 계속 기본 키를 전달하는 식별 관계 @EmbeddedClass 사용 예제

```java

// Parent
@Entity
public class Parent {

    @Id @Column(name="PARENT_ID")
    private String id;

    private String name;
    ...
}

// Child
@Entity
public class Child {

    @EmbeddedId
    private Child child;

    @MapsId("parentId") // ChildId.parentId 매핑.
    @ManyToOne
    @JoinColumn(name="PARENT_ID")
    public Parent parent;

    private string name;
    ...
}

//ChildId
@Embeddable
public class ChildId implements Serializable {

    private String parentId; // MapsId("parentId") 매핑
    
    @Column(name="CHILD_ID")
    private string childId; //Child.childId 매핑

    //equals, hashCode
    ...
}

// GrandChild
@Entity
public class GrandChild {

    @EmbeddedId
    private GrandChildId id;

    @MapsId("childId")  // GrandChildId.childId 매핑
    @ManyToOne
    @JoinColumns ({
        @JoinColumn(name="PARENT_ID"),
        @JoinColumn(name="CHILD_ID")
    })
    private Child child;

    private String name;
    ...

}

// GrandChildId
@Embeddable
public class GrandChildId implements Serializable {

    private ChildId childId; // @MapsId("childId") 매핑
    
    @Column(name="GRANDCHILD_ID")
    private String id;

    // equals, hashCode
    ...
}
```

## 부모, 자식, 손자로 계속 기본 키를 전달하는 비식별 관계 @EmbeddedClass 사용 예제

```java

@Entity
public class Parent {

    @Id @GeneratedValue
    @Column(name="PARENT_ID")
    private Long id;

    private String name;
    ...
}

@Entity
public class Child {

    @Id @GeneratedValue
    @Column(name="CHILD_ID")
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name="PARENT_ID")
    private Parent parent;

    ...

}

@Entity
public class GrandChild {

    @Id @GeneratedValue
    @Column(name="GRANDCHILD_ID")
    private Long id;

    private String name

    @ManyToOne
    @JoinColumn(name="CHILD_ID")
    private Child child;

    ...
}

```

## 비식별 관계 선호...하는 이유
- 데이터베이스 관점에서 아래와 같은 이유로 비식별 관계를 선호한다.
    1. 식별 관계는 부모 테이블의 기본 키를 자식 테이블로 전파하면서 자식 테이블의 기본 키 컬럼이 점점 늘어난다. 결국 조인할 때, SQL이 복잡해지고 기본 키 인덱스가 불필요하게 커질 수 있다.
    2. 식별 관계는 2개 이상의 컬럼을 합해서 복합 기본 키를 만들어야 하는 경우가 많다.
    3. 식별 관계를 사용할 때 키본 키로 비즈니스 의미가 있는 자연 키 컬럼을 조합하는 경우가 많다. 반면에 비식별 관계의 기본 키는 비즈니스와 전혀 관계없는 대리 키를 주로 사용한다. 비즈니스 요구사항은 시간이 지남에 따라 언젠가는 변한다. 식별 관계의 자연 키 컬럼들이 자식에 손자까지 전파되면 변경하기 힘들다.
    4. 식별 관계는 부모의 테이블의 기본 키를 자식 테이블의 기본 키로 사용하므로 비식별 관계보다 테이블의 구조가 유연하지 못하다.

- 객체 관계 매핑의 관점에서는 아래와 같은 이유로 비식별 관계를 선호한다.
    1. 일대일 관례를 제외하고 식별 관계는 2개 이상의 컬럼을 묶은 복합 기본 키를 사용한다. JPA에서 복합 키는 별도의 복합 키 클래스를 만들어서 사용해야 한다. 따라서 컬럼이 하나인 기본 키를 매핑하는 것보다 많은 노력이 필요하다.
    2. 비식별 관계의 기본 키는 주로 대리 키를 사용하는데 JPA는 @GeneratedValue처럼 대리 키를 생성하기 위한 편리한 방법을 제공한다. 


## 조인 테이블
- 조인 컬럼 사용 (외래 키)
- 조인 테이블 사용 (테이블 사용)
- 객체와 테이블을 매핑할 때 조인 컬럼은 @JoinColumn으로 매핑하고 조인 테이블은 @JoinTable로 매핑한다.
- 조인 테이블은 주로 다대다 관계를 일대다, 다대일 관계로 풀어내기 위해 사용한다. 그렇지만 일대일, 일대다, 다대일 관계에서도 사용한다,


































