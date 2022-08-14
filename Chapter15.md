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

![Persistence Context Scopeß](/Users/hyunwook/Pictures/persistence_context_scope.png)