
6장 다양한 연관관계 매핑
==============
## 연관관계 매핑에 있어 다음의 3가지를 고려하여야 한다.
1. 다중성 - 다대일, 일대다, 일대일, 다대다. 다중성을 판단하기 어려울 때는 반대방향을 생각해보면 된다.
2. 단방향, 양방향 - 테이블은 양방향이라는 개념이 없지만, 객체는 참조용 필드를 통해 단방향, 양방향 둘다 가능한다.
3. 연관관계 주인 - mappedBy 사용하지 않는다. 연관관계 주인이 아니면 mappedBy 속성을 사용하고 연관관계의 주인 필드 이름을 값으로 입력해야 한다.

## 다대일 [N:1]
syntax: [다대일 예제] (https://github.com/HyeonwookCho/JPA-Today-I-learn/blob/master/Chapter5.md#%EB%8B%A8%EB%B0%A9%ED%96%A5-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84)

## 다대일 [N:1, 1:N]
syntax: [다대일 예제] (https://github.com/HyeonwookCho/JPA-Today-I-learn/blob/master/Chapter5.md#%EC%96%91%EB%B0%A9%ED%96%A5-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84)

## 일대다
- 일대다 관계는 다대일 관계의 반대방향이다. 일대다 관계는 엔티티를 하나 이상 참조할 수 있으므로 자바 컬렉션인 Collection, List, Set, Map 중에 하나를 사용해야 한다.

    1. 일대다 단방향 [1:N]
    - 하나의 팀은 여러 회권을 참조할 수 있는 관계, 반대로 회원은 팀을 참조하지 않으면 단방향.

    ```java
    @Entity
    public class Team {

        @Id
        @GeneratedValue
        private Long id;

        private String name;

        @OneToMany
        @JoinColumn(name="TEAM_ID") // Member 테이블의 TEAM_ID(FK)
        private List<Member> members = new ArrayList<Member>();

        // Getter, Setter ...

    }

    @Entity
    public class Member {

        @Id @GeneratedValue
        @Column(name = "MEMBER_ID")
        private Long id;

        private String username;

        //Getter, Setter...
    }
    ```

    - __일대다 단방향 매핑의 단점__ : 외래 키가 다른 테이블에 있기 때문에 INSERT 한번으로 끝날 것을 UPDATE를 추가로 진행해야한다.
    - __일대다 매핑보다는 다대일 매핑을 하자__



## 일대다 양방향
- 일대다 양방향 매핑은 존재하지 않고 다대일 양방향 매핑을 사용해야 한다. 양방향 매핑에서 @OneToMany는 연관관계의 주인이 될 수 없다. 왜냐하면 관계형 데이터베이스의 특성상 일대다, 다대일 관계는 항상 다 쪽에 외래 키가 있다. 따라서 @OneToMany, @ManyToOne 둘 중에 연관관계의 주인은 항상 다 쪽인 @ManyToOne을 사용한 곳이다. @ManyToOne에는 mappedBy 속성이 없다. 그렇다고 완전 불가능은 아니지만 굳이...

__일대다 보다는 다대일을 사용하자__

## 일대일[1:1]
- 일대일 관계는 양쪽이 서로 하나의 관계만 가진다. 예를 들어 회원은 하나의 사물함만 이용하고 사물함도 하나의 회원에 의해서 사용된다.
- 일대일 관계는 주 테이블이나 대상 테이블 중에 누가 외래 키를 가질지 선택해야 한다. 왜냐하면 보통 다(N) 쪽에 외래 키를 가졌지만 일대일 관계에서는 다 쪽이 없기 때문이다.
    1. 주 테이블의 외래 키 : 주 객체가 대상 객체를 참조하는 것처럼 주 테이블에 외래 키를 두고 대상 테이블을 참조한다. 외래 키를 객체 참조화 비슷하게 사용할 수 있어서 객체지향 개발자들이 선호한다.
    2. 대상 테이블의 외래 키 : 전통적인 데이터베이스 개발자들은 보통 대상 테이블에 외래 키를 두는 것을 선호한다. 이 방법의 장점은 테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있다.

## 주 테이블의 외래 키

- 단방향
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name="LOCKER_ID") // 데이터베이스에서는 unique 조건 추가
    private Locker locker;
    ...
}

@Entity
public class Locker {
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;
    ...
}
```

-양방향
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name="LOCKER_ID") // 데이터베이스에서는 unique 조건 추가
    private Locker locker;
    ...
}

@Entity
public class Locker {
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;
    
    @OneToOne(mappedBy="locker")  // 양방향으로 연관관계 주인이 아니라고 설정
    private Member member;
    ...
}
```

## 대상 테이블의 외래 키
- 단방향 : X

- 양방향
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne(meppedby="member")
    private Locker locker;
    ...
}

@Entity
public class Locker {
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;
    
    @OneToOne
    @JoinColumn(name="MEMBER_ID")
    private Member member;
    ...
}
```

## 다대다 [N:N]
- 회원과 상품의 관계를 예로 들어 회원은 상품을 주문, 상품들은 회원에 의해 주문되어 지는 다대다의 관계이다. 이는 회원 테이블과 상품 테이블만으로는 이 관계를 표현할 수 없다. 이에 Member, Member_Product, Product 이렇게 중간에 연결 테이블을 만들어 풀어낼 수 있다. 그러나 객체는 컬렉션을 이용하여 다대다 관계를 매핑할 수 있다.

- 단방향
```java
@Entity
public class Member {
    @Id @Column(name="MEMBER_ID")
    private String id;

    private String username;

    @ManyToMany
    @JoinTable(name="MEMBER_PRODUCT",                        // 연결 테이블 지정. Member_Product 엔티티 없이 매핑 완료 가능.
               joinColumns = @JoinColumn(name="MEMBER_ID"),  // 현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정한다.
               inverseJoinColumns = @JoinColumn(name="PRODUCT_ID") // 반대 방향인 상품과 매핑할 조인 컬럼 정보를 지정한다.
               )
    private List<Product> products = new ArrayList<Product>();
    ...
}

@Entity
public class Product {
    @Id @GeneratedValue
    private String id;

    private String name;
    ...
}
```

- 다대다 테이블 저장 예제
```java
public void save() {
    Product productA = new Product();
    productA.setId("ProductA");
    productA.setName("ProductAA");
    em.persist(productA);

    Member member1 = new Member();
    member1.setId("member1");
    member1.setUsername("member11");
    member1.getProducts().add(productA); // 연관관계 설정
    em.persist(member1);
}
```

- 양방향
```java
@Entity
public class Product {
    @Id @GeneratedValue
    private String id;

    private String name;

    @ManyToMany(mappedBy="products") // 역방향 추가
    private List<Member> members;

    ...
}
```


## 다대다: 매핑의 한계와 극복, 연결 엔티티 사용
- 위와 같이 다대다 관계를 처리하면 __실무에서는 사용하기 어렵다.__
- 이유는 보통 회원이 상품을 주문하면 회원 아이디와 상품 아이디만 담고 끝나지 않는다. 주문 수량, 주문한 날짜와 같은 컬럼을 추가 했을 때 매핑할 수 없다. 결국 연결 테이블을 매핑하는 연결 엔티티를 만들고 이곳에 추가한 컬럼들을 매핑해야 한다. 그리고 엔티티의 관계로 다대다에서 일대다, 다대일의 관계로 풀어야 한다.

```java
@Entity
public class Member {
    @Id @Column(name="MEMBER_ID")
    private String id;

    // 역방향 회원상품 엔티티가 ID값을 가지고 있기 때문에
    @OneToMany(mappedBy="member")
    private List<MemberProduct> memberProducts;
    ...
}

@Entity
public class Product { // 상품 엔티티에서 회원상품 엔티티로 객체 그래프 탐색 기능이 필요하지 않다고 판단.
    @Id @Column(name="PRODUCT_ID")
    private String id;

    private String name;
}

@Entity
@IdClass(MemberProductId.class)
public class MemberProduct {

    @Id
    @ManyToOne
    @JoinColumn(name="MEMBER_ID")  // @Id와 @JoinColumn을 동시에 사용하여 기본 키 + 외래 키를 한번에 매핑.
    private Member member;         // MemberProductId.member 와 연결.

    @Id
    @ManyToOne
    @JoinColumn(name="PRODUCT_ID")
    private Product product;       // MemberProductId.product 와 연결.    
}

public class MemberProductId implements Serializable {
    private String member  // MemberProduct.member 와 연결
    private String product // MemberProduct.product 와 연결

    @Override
    public boolean equals(Object o) {...}

    @Override
    public int hashCode() {...}
}
```

## 복합 기본 키
- JPA에서 복합키를 사용하려면 별도의 식별자 클래스를 만들어야한다. 그리고 @IdClass를 사용해서 식별자 클래스를 지정하면 된다.
    1. 복합 키는 별도의 식별자 클래스를 만들어야 한다.
    2. Serializable을 구현해야한다. 
    3. equals와 hashCode 메소드를 구현해야 한다
    4. 기본 생성자가 있어야 한다.
    5. 식별자 클래스는 public 이어야 한다.
    6. @IdClass를 사용하는 방법 외에 @EmbeddedId를 사용하는 방법도 있다


## 다대다 : 새로운 기본 기 사용
- 데이터베이스에서 생성해주는 대리 키를 Long 값으로 사용하는 것. 장점은 영구히 사용이 가능하고 비즈니스에 의존하지 않는다.

```java
@Entity
public class Order {
    @Id
    @GeneratedValue
    @Column(name="ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name="MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name="PRODUCT_ID")
    private Product product;

    private int orderAmount;
    ...
}
```

## 다대다 연관관계 정리
- 식별 관계 : 받아온 식별자를 기본 키 + 외래 키로 사용한다.
- 비식별 관계 : 받아온 식별자는 외래 키로만 사용하고 새로운 식별자를 추가한다. __책에서는 비식별 관계 추천__
