
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
