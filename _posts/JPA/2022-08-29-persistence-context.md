---
title: "2. 영속성 관리와 컨텍스트(PersistenceContext)"
layout: single
categories: JPA
typora-root-url: ..\..\images\
toc: true
---

이 포스트 시리즈는 inflearn의 ''자바 ORM 표준 JPA 프로그래밍 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------





# 2. 영속성 관리와 컨텍스트(PersistenceContext)



## 1) 영속성 관리

- JPA의 기능과 장점을 활용하기 위한 개념.
- 영속성 컨텍스트(PersistenceContext)를 사용하여 JPA가 데이터를 바라보고 관리하는 상태로 만듦. (마치 SpringBoot의 컨테이너처럼)
- 이 영속성 관리를 통해 Batch, 지연로딩, DB 쿼리 효율화 등이 가능해짐.



## 2) 영속성 컨텍스트(PersisteenceContext)

- JPA가 바라보고 관리하는 곳.
- `EntityManager`를 통해서 접근할 수 있음. 
- `EntityManagerFactory`에서 `EntityManager`를 만들고, `EntityManager`는 DB의 커넥션풀을 사용하여 DB에 접근함.
- 실제로 보이지는 않고, 논리적인 개념.

**J2SE 환경 - 엔티티매니저와 영속성 컨텍스트가 1:1 매칭 / J2EE, 스프링 프레임워크와 같은 컨테이너 환경 - 엔티티매니저와 영속성 컨텍스트가 N:1 매칭* (이 부분은 추가 학습 필요)



## 3) 영속성 컨텍스트의 기능 및 이점

- Member.java

```java
import javax.persistence.Entity;
import javax.persistence.Id;

@Entity // JPA가 관리할 객체
public class Member {

    @Id // DB pk와 매핑
    private Long id;
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

- JpaMain.java

```java
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.util.List;

public class JpaMain {

    public static void main(String[] args) {

        EntityManagerFactory emf =                  // persistence.xml의 설정이름
                Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager(); //DB connection 객체랑 비슷

        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            // 등록
            Member member = new Member();
            member.setId(3L);
            member.setName("Name3");
            em.persist(member);

            // 영속성 확인
            System.out.println("=== find 3L ===");
            Member findMember = em.find(Member.class, 3L);
            System.out.println("findId: " + findMember.getId());
            System.out.println("findName: " + findMember.getName());

            System.out.println("=== find 3L twice ===");
            Member findMember2 = em.find(Member.class, 3L);
            System.out.println("findId: " + findMember2.getId());
            System.out.println("findName: " + findMember2.getName());

            // 조회 및 수정
            Member findMember = em.find(Member.class, 1L);
            findMember.setName("1L");

            // 검색(조회)
            List<Member> result = em.createQuery
                // Member 테이블이 아니라 Member 객체라고 생각하면 됨.
                ("select m from Member as m", Member.class)
                .getResultList();
            for (Member member : result) {
                System.out.println("member.name = " + member.getName());
            }

            // 준영속 상태 만들기 (JPA가 더이상 관리 하지 않는 상태)
            Member findMember = em.find(Member.class, 3L);
            findMember.setName("findMember3");
            em.detach(findMember);

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }
        emf.close();
    }
}

```



### 1. 1차 캐시

- 영속성 컨텍스트를 이야기함. 
- 트랜잭션 단위로 만들게 됨. 따라서 굉장히 짧은 시간이고, 잠깐의 효율정도를 가짐.(JPA의 동작원리를 이해하는 것에 필요한 편)
- `.persist()`나 `.find()` 등 DB 작업을 위한 세팅을 하게 되면 데이터가 영속성 컨텍스트인 1차 캐시에 들어가게 됨.

![context_1](..\..\images\context_1.PNG)

![context_2](..\..\images\context_2.PNG)

- 같은 트랜잭션 안의 영속 컨텍스트에서 가져온 내용은 '==' 비교를 했을 때 true! (java 컬랙션에서 가져오는 것처럼 참조값이 같음) => <u>영속 엔티티의 동일성 보장</u>



### 2. SQL 쓰기 지연

- `transaction.begin()`이 되고 나서 영속성 컨텍스트에 반영되는 내용들은 JPA가 관리를 하고 있음.

- 바로 SQL 쿼리를 날려 DB에 반영하지 않고, 쓰기 지연 SQL 저장소에 보관을 해 둔다. (JDBC Batch)

  ```xml
  <property name="hibernate.jdbc.batch_size" value="10"/> <!-- 10개만큼 모아서 쏘세요 -->
  ```

- `transaction.commit()`을 통해 영속성 컨텍스트의 내용들이 모두 정리된 후 쿼리를 생성하고 날린다. (`flush`라고 하며, 필요에 따라서는 개발자가 직접 flush하거나 영속성 컨텍스트를 비울 수 있음.)



### 3. 변경 감지

- update쿼리를 위한 메서드가 필요 없음.
- `flush()` 작업이 이뤄지게 되면 영속성 컨텍스트(1차 캐시)에 등록될 때의 값(스냅샷)과 현재 값을 비교함.
- 비교해서 달라진 부분이 있으면 JPA가 UPDATE SQL도 생성하여 `flush()`작업을 하게 됨. 
- `스냅샷` - 최초로 컨텍스트에 등록이 되었을 때의 값.



## 4) 플러시(flush)

- 쓰기 지연 SQL 저장소에 모여있는 쿼리를 보내는 것.
- 실제로 실무에서 사용할 일은 거의 없음. (데이터의 신뢰성이 중요한데, 트랜잭션 작업단위에서는 기본적으로 영속성 관리가 되는 것이 유리하기 때문)
- 영속성 컨텍스트 내용에는 아무런 영향이 없음. (즉, 비워지거나 하지 않는다는 뜻)
- 트랜잭션이 `commit()`될 때 호출되는 것이 기본값.
- `flush()`메서드로 직접 호출 가능.
- `JPQL` 쿼리 사용시에도 자동으로 호출 됨.

```java
[JPQL쿼리 예시]
query = em.createQuery("select m from Member m", Member.class);
List<Member> members = query.getResultList();
```





## 5) 준영속 상태

- 영속 상태에서 벗어난 상태. 즉, JPA가 더 이상 관리를 안한다는 것.
  - `.detach(entity이름)` - 특정 엔티티를 준영속 상태로 전환.
  - `.clear()` - 영속성 컨텍스트 초기화.
  - `.close()` - 영속성 컨텍스트 종료.

![detach](..\..\images\detach.PNG)

- `find();` - DB에서 값 조회 시 select 쿼리가 전송. (왜? 1차 캐시에 없기 때문에) -> 값을 조회함과 동시에 영속 컨텍스트(1차 캐시)에 관리가 됨. --> 스냅샷을 찍어놓음.
- `detach();` - 준영속 상태. 즉, 더이상 JPA에서 관리하지 않는 상태로 만듦. -> 영속상태였으면 setName 으로 데이터가 변경되었다는 것을 스냅샷을 통해 확인하지만, 준영속상태이기 때문에 더이상 관리하지 않아 비교 불가. --> commint을 했어도 어떠한 쿼리도 전송되지 않음.

------

> 마지막 수정일시: 2022-08-29 13:43