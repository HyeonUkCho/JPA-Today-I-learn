15장 고급 주제와 성능 최적화
========================

## JPA 표준 예외처리

- 트랜잭션 롤백을 표시하는 예외 : 예외 발생하면 트랜잭션을 강제로 커밋해도 커밋되지 않고 대신에 Javax.persistence.RollbackException 예외 발생한다.

- 트랜잭션 롤백을 표시하지 않는 예외 : 심각한 예외가 아니다. 커밋할지 롤백할지 결정이 필요하다.

## 스프링 프레임워크의 JPA 예외 변환

- 서비스 계층에서 데이터 접근 계층의 구현 기술에 직접 의존하는 것은 좋은 설계가 아니다. 이에 스프링은 추상화해서 개발자에게 제공한다. 만약, 예외변환을 하고 싶지 않다면 throws JPA....Exception으로 하면 변환하지 않고 예외를 던진다.

## Transaction Rollback 주의사항

- 트랜잭션 롤백 >> DB 롤백 >> 수정 자바 객체 롤백 X(영속성 컨텍스트에 남아있음.)
- 스프링은 해당 현상을 미연에 방지하기 위해서 엔티티를 클리어한다. (clear() 함. doRollback 참고)
- OSIV : 영속성의 범위를 트랜잭션보다 넓게할 경우, 여러 트랜잭션이 하나의 영속성 컨텍스트 사용

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

## 더욱 더 자세히...책을 보자...쓰기에는 너무 많다... 그래도 아래에 정리를 조금씩 해보겠습니다

## JPQL

- SELECT : SELECT m FROM Member AS m where m.username = 'Hello';
  - 대소문자 구분, 엔티티 이름, 별칭은 필수
  - TypeQuery : 반환 타입을 명확하게 지정할 수 있을 때
  - Query : 반환 타입을 명확하게 지정할 수 없을 때
  - 파라미터 바인딩

    ```java
    String usernameParam = "User1";
    TypeQuery query = em.createQuery("SELECT m FROM Member AS m where m.username = :username", Member.class);

    query.setParameter("username", usernameParam);
    List<Member> resultList = query.getResultList();
    ```

- 프로젝션 : SELECT 절에 조회할 대상을 지정하는 것을 프로젝션이라고 한다.
  - 엔티티 프로젝션 - SELECT m FROM Member m, SELECT m.team FROM Member m
  - 임베디드 타입 프로젝션 -
