# 3.1 엔티티 매니저 팩토리와 엔티티 매니저

엔티티 매니저 - 엔티티 매니저는 엔티티를 관리하는 관리자이다. 저장, 수정, 삭제 등을 처리한다.
            앤티티 매니저는 엔티티 매니저 팩토리에서 가져올 수 있다. 보통 하나의 데이터베이스를 사용하는 애플리케이션은 하나의 엔티티 팩토리 매니저를 생성한다.  

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpatest");
EntityManager em = emf.createEntitymanager();  
```