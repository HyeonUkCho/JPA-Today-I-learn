
5장 연관관계 맵핑 기초
==============

## 단방향 연관관계
- 다대일 연관관계
    1. Member와 Team의 연관 관계. 통상적으로 Member 여러명이 하나의 Team에 속할 수 있다.

```java
@Entity
public class Member {
    @Id
    @Column(name="MEMBER_ID")
    private String id;

    private String username;

    // 연관관계 맵핑
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;

    public void setTeam(Team team) {
        this.team = team;
    }

    // Getter, Setter...
}

@Entity
public class Team {
    @Id
    @Column(name="TEAM_ID")
    private String id;

    private String name;

    // Getter, Setter...
}
```

## @JoinColumn 주요 속성
    1. name : 매핑할 외래 키 이름 (default : 필드명 + _ + 참조하는 테이블의 기본 키 컬럼명)
    2. unique, nullable, inserable, updatable,,,

## @ManyToOne 주요 속성
    1. optional : false로 설정하면 연관된 Entity가 항상 있어야한다.
    2. fetch : 글로벌 패치 전략 사용 (8장)
    3. cascade : 영속성 전이 기능 (8장)
    4. targetEntity : 연관된 엔티티의 타입 정보를 설정한다. 거의 사용 X

## 위의 연관관계 설정한 부분을 사용하는 예제
```java

// 저장
public void testSave() {
    Team team1 = new Team("team1", "회원1");
    em.persist(team1);

    Member member1 = new Member("member1", "회원1");
    member1.setTeam(team1); // 연관관계 설정 member1 -> team1
    em.persist(member1);

    Member member2 = new Member("member1", "회원2");
    member1.setTeam(team1); // 연관관계 설정 member2 -> team1
    em.persist(member2);
    
}

// 객체 그래프 탐색 (자세한 내용 8장)
Member member = em.find(Member.class, "member1");
Team team = member.getTeam();

// JPQL 사용
private static List<Member> queryLogicJoin(EntityManager em) {
    String jpql = "select m from Member m join m.team t where " + "t.name=:teamName";
    List<Member> resultList = em.createQuery(jpql, Member.class)
                                .setParameter("teamName", "팀1");
                                .getResultList();
    
    return resultList;
}

// 수정
private static void updateRelation(EntityManager em) {
    // new Team
    Team team2 = new Team("team2", "팀2");
    em.persist(team2);

    // update team of member1
    Member member = em.find("Member.class", "member1");
    member.setTeam(team2);
}

// 연관관계 제거
private static void deleteRelation(EntityManager em) {
    
    Member member = em.find("Member.class", "member1");
    member.setTeam(null);
}

// 연관된 엔티티 삭제 : 기존에 연관관계를 모두 제거하고 엔티티를 삭제해야한다.
member1.setTeam(null);
member2.setTeam(null);
em.remove(team);

```


## 양방향 연관관계
- 단방향에서 멤버에서 팀의 관계를 알아봤다. 이제 반대인 팀에서 회원으로 접근하는 관계를 추가.

```java
@Entity
public class Member {
    @Id
    @Column(name="MEMBER_ID")
    private String id;

    private String username;

    // 연관관계 맵핑
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;

    public void setTeam(Team team) {
        this.team = team;
    }

    // Getter, Setter...
}

@Entity
public class Team {

    @Id
    @Column(name="TEAM_ID")
    private String id;

    private String name;

    //==추가==//
    @OneToMany(mappedBy = "team") // 연관관계 주인 설정
    private List<Member> members = new ArrayList<Member>();

    // Getter, Setter...
}

```

- 연관관계 주인을 설정하는 이유 : 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리한다. 그러나 엔티티를 단방향으로 매핑하면 참조를 하나만 사용하여 이 참조로 외래 키 하나만 관리하면 된다. 그러나 양방향으로 설정을 하게 되면 객체의 참조는 둘인데 외래 키는 하나가 된다. 이 점이 차이를 발생시킨다. 이러한 차이로 하나의 테이블을 정하여 외래 키를 관리하는 __연관관계 주인__ 설정을 한다.
- 연관관계 주인을 설정할 경우, mappedBy를 사용하면 되고 연관관계 주인만이 __외래 키 관리(등록, 수정, 삭제)__ 를 할 수 있다. 주인이 아닌 쪽은 읽기만 가능하다.
    * 주인은 mappedBy 사용 X
    * 주인이 아니면 mappedBy 속성을 사용해서 속성의 값으로 연관관계의 주인을 지정해야한다.

## 연관관계의 주인은 외래 키가 있는 곳!!

## 양방향 연관관계 저장 예제

```java
public void testSave() {
    // team1 save
    Team team1 = new Team("team1", "tteam1");
    em.persist(team1);

    // member1 save
    Member member1 = new Member("member1", "mmeber1");
    member1.setTeam(team1); // 연관관계 설정
    em.persist(member1);

    // member2 save
    Member member2 = new Member("member2", "mmeber2");
    member2.setTeam(team1); // 연관관계 설정
    em.persist(member2);

    /*
    *   아래의 코드가 있어야할 것 같지만 Team.members는 연관관계의 주인이 아니다.
    *   주인이 아닌 곳에 입력된 값은 외래 키에 영향을 주지 않는다.
    *   주인이 아닌 방향은 값을 설정하지 않아도 데이터베이스에 외래 키값이 정상 입력된다.
    */
    // team1.getMembers().add(member1);
    // team1.getMembers().add(member2);
}
```

## 양방향 연관관계 주의점 !!
- 주인인 곳에 값을 입력!, 주인이 아닌 곳은 입력을 해도 소용 X

## 순수한 객체까지 고려한다면?
- 주인인 곳에만 입력을 하지 않고 __객체관점__ 에서 양쪽 방향에 모두 값을 입력해주는 것이 안전한다.

## 연관관계 편의 메소드!! ##중요##
- 양방향 연관관계는 결국 양쪽 모두 설정을 해줘야한다. 실수로 둘중 하나만 설정한다면 양방향이 깨질 수 있다. 아래와 같이 리팩토링 해보자

```java
public class Member {
    private Team team;

    public void setTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}
```
- 그러나 위의 코드에서 Member가 이전에 또 다른 team과 연관관계가 있다면? 이전 연관관계를 삭제하고 설정을 해주어야 한다.
```java
public class Member {
    private Team team;

    public void setTeam(Team team) {
        // 기존 연관관계 제거
        if(this.team != null) {
            this.team.getMembers().remove(this);
        }

        this.team = team;
        team.getMembers().add(this);
    }
}
```

## 정리
- 단방향 매핑만으로 테이블과 객체의 연관관계 매핑은 이미 완료되었다.
- 단방향을 양방향으로 만들면 반대방향으로 객체 그래프 탐색이 가능한다.
- 양방향 연관관계를 매핑하려면 객체에서 양쪽 방향을 모두 관리해야 한다.
- __비즈니스 로직의 필요에 따라 다르겠지만 우선 단방향 매핑을 사용하고 반대 방향으로 객체 그래프 탐색 기능(JPQL 포함)이 필요할 때 양방향을 사용하도록 코드를 추가해도 된다.__
- __연관관계 주인을 정하는 기준 : 외래 키가 있는 곳!!__


## 무한 루프에 빠지지 않도록 주의 (추후 검색하여 이해 필요)
- 양방향 매핑 시에 Member.toString()에서 getTeam()을 호출하고 Team.toString()에서 getMember()를 호출하면 무한루프에 빠질 수 있다.