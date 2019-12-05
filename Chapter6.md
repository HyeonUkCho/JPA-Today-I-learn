
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
