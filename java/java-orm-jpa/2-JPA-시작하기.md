## 2. JPA 시작하기
- JPA에서 `hello world`를 간단하게 해볼 것임

### 프로젝트 생성
- 

### JPA EntityManager 기본 메서드
- `tx.begin()`으로 트랜잭션을 시작함
    - tx: `EntityTransaction tx = em.getTransaction()`
- `tx.commit()`으로 트랜잭션을 닫음
    - 만약 rollback이 필요한 상황이면 `tx.rollback()` 
- **INSERT**
    ```java
        Member member = Member.createMember(1L, "memberA");
        em.persist(member);
    ```
    - `em.persist()` 메서드로 영속성을 부여한 후에 트랜잭션을 닫아 적용시킴
- **UPDATE**
    ```java
        Member member = em.find(Member.class, 1L); // { class, PK }
        member.setName("memberB");
    ```
    - **자바 객체에서 값만 바꿔도** JPA를 통해 엔티티를 가져오면 JPA가 관리함
        - 값에 변화가 생기면 update 쿼리를 만들어 커밋 시점에 보냄
- 쿼리 사용(JPQL)
    - 예시. 나이가 18살 이상인 회원을 모두 검색하고 싶으면?
    - JPA는 **SQL을 추상화한 JPQL**이라는 객체 지향 쿼리 언어를 제공함
        - JPQL은 **엔티티 객체를 대상으로 쿼리**
        - SQL은 DB 테이블을 대상으로 쿼리
        - **"객체 지향 SQL"**
    - 전체 조회
        ```java
            List<Member> members = em.createQuery("select m from Member m", Member.class)
                    .getResultList();
            members.forEach(System.out::println); // @ToString in Member.class
        ```

---

## 다음 글 

### 3. [영속성 관리 - 내부 동작 방식](3-영속성-관리-내부-동작-방식.md)