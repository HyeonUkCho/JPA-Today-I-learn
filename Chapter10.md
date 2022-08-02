10장 객체지향 쿼리 언어
========================

## 객체지향 쿼리 소개
- JPQL
- Criteria 쿼리
- 네이티브 SQL
- Query DSL
- JDBC, Mybatis


## JPQL
- 엔티티 객체를 조회하는 객체지향 쿼리이다. JPQL은 SQL을 추상화해서 특정 데이터베이스에 의존하지 않는다. 데이터베이스 Dialect만 변경하면 JPQL을 수정하지 않아도 자연스럽게 데이터베이스를 변경할 수 있다.
- JPQL은 SQL보다 간결하다. 엔티티 직접 조회, 묵시적 조인, 다형성 지원으로 SQL보다 코드가 간결하다.
```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;
    private int age;
    
    //...
}

//쿼리 생성
String jpql = "select m from Member as m where m.name = 'kim'";
List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();
// em.createQuery를 하면 SQL로 변환되어 DB 조회 후 엔티티를 리턴한다.
```

## Criteria 쿼리 소개
- Criteria는 JPQL을 생성하는 빌더 클래스다. Criteria의 장점은 문자가 아닌 query.select(m).where(...)처럼 프로그래밍 코드로 JPQL을 작성할 수 있다는 점이다. 등등...그렇게 좋아보이지 않아서 정리는 이만..끄...ㅌ

## Query DSL (오픈소스 프로젝트)
- Query DLS도 Criteria처럼 JPQL 빌더 역할을 한다. Query DLS의 장점은 코드 기반이면서 단순하고 사용하기 쉽다. 그리고 작성한 코드도 JPQL과 비슷해서 한눈에 들어온다.
```java
//Ready
JPAQuery query = new JPAQuery(em);
QMember member = QMember.member;

//Query, Select
List<Member> members = query.from(member).where(member.usename.eq("kim")).list(member);
```
- Query DSL도 어노테이션 프로세서를 사용해서 쿼리 전용 클래스를 만들어야 한다. QMember는 Member 엔티티 클래스를 기반으로 생성한 Query DSL 쿼리 전용 클래스다.  

__네이티브 SQL, JDBC, Mybatis 등등이 있다...__

## 더욱 더 자세히...책을 보자...쓰기에는 너무 많다... 그래도 아래에 정리를 조금씩 해보겠습니다...



## JPQL
- SELECT : SELECT m FROM Member AS m where m.username = 'Hello';
    * 대소문자 구분, 엔티티 이름, 별칭은 필수
    * TypeQuery : 반환 타입을 명확하게 지정할 수 있을 때
    * Query : 반환 타입을 명확하게 지정할 수 없을 때
    * 파라미터 바인딩
    ```java
    String usernameParam = "User1";
    TypeQuery query = em.createQuery("SELECT m FROM Member AS m where m.username = :username", Member.class);

    query.setParameter("username", usernameParam);
    List<Member> resultList = query.getResultList();
    ```
- 프로젝션 : SELECT 절에 조회할 대상을 지정하는 것을 프로젝션이라고 한다.
    * 엔티티 프로젝션 - SELECT m FROM Member m, SELECT m.team FROM Member m
    * 임베디드 타입 프로젝션 - 

