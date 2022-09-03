---
title: "⑦. 프록시와 연관관계 관리"
layout: single
categories: JPA
typora-root-url: ..\..\images\
toc: true
---

이 포스트 시리즈는 inflearn의 ''자바 ORM 표준 JPA 프로그래밍 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------



# 7. 프록시와 연관관계 관리





## 1) 프록시

- 단어 뜻: 대리인. 즉, 본체가 아니라 본체를 대신할 객체. (여기서는 Team 엔티티의 역할을 대신하는 더미 엔티티라고 생각하면 됨)

  ![image-20220902232832268](..\..\images\image-20220902232832268.png)

- Member가 연관관계 주인이고 다대일로 연관관계가 맺어져있다고 가정.

- Member를 find하면 Team을 조인해서 항상 데이터들을 가져와야 할까?

  - 비즈니스 로직이 멤버와 팀을 함께 사용하는 경우가 대다수라면 그렇게 하는게 좋을 수 있음.

  - 그러나, 대부분의 경우는 Member의 정보가 필요해서 불러올 것.

    => JPA는 이런 경우 `프록시` 객체를 사용하여 `지연로딩`을 하여 문제를 해결하고 성능을 향상시킴.

- 프록시 객체는 실제 객체의 참조(target)값을 보관하고 있음. 

- 프록시 동작원리

  - Member를 호출하면 프록시(더미, 껍데기)를 통해서 Team 객체도 반환한 척 하고 있음. 
  - 실제 Team정보를 필요로 할 때 프록시 객체에서 영속성 컨텍스트에 자료를 요청함.(초기화 요청)
  - 이후 영속성 컨텍스트에서 값을 반환하기 위해 DB에 접근하고 이 때 쿼리가 날아감.
  - 실제 엔티티 생성.
  - 프록시 객체의 target 참조값이 생성된 진짜 엔티티를 바라보게 됨.(연결, 초기화 작업 완료)

  ```java
  Member member = em.getReference(Member.class, "id1"); // 프록시 엔티티 반환
  String name = member.getName(); // 어? 진짜 데이터가 필요하네? 초기화 요청
  
  ======= [ 다른 트랜젝션 ] ========
      
  Member member = em.find(Member.class, "id1"); // Member는 실제 엔티티 반환(find이기 때문)
  Team team = member.getTeam(); // Member 내부에 있는 Team 객체는 프록시 엔티티 반환
  String teamName = team.getName(); // 어? 진짜 데이터가 필요하네? 초기화 요청
  ```

  - `em.find()` : 데이터베이스를 통해서 실제 엔티티 객체를 조회함. => 쿼리가 날아감.

  - `em.getReference()` : 데이터베이스 조회를 미루고 더미 엔티티를 조회해놓음.



### - 프록시의 특징

- 프록시 객체는 처음 사용할 때 한 번만 초기화. 이후는 영속성 컨텍스트에서 값을 가져옴.

- 프록시 객체를 초기화 할 때, 프록시 객체 대신 진짜 객체가 들어오는게 아니라 프록시 객체의 target 참조값에 실제 엔티티 위치가 들어가게 되는 것임.

- 프록시와 실제 객체가 언제 어떻게 반환될 지 모르기 때문에, 타입 체크 시 == 비교는 조심해야 함.

- 만약, 1차 캐시에 찾는 엔티티가 이미 있다면 프록시가 필요없기 때문에 `getReference()`를 해도 실제 엔티티 반환.

- `detach()`나 `clear()` 등으로 준영속 상태가 되면 초기화 시 오류 발생. (영속성 컨텍스트의 도움을 받을 수 없는데, 값을 어떻게 가져오라고???) => `LazyInitializationException` 예외

  

### - 프록시 정리

- ① 당장 필요한 값이 아니고 ② 영속성 컨텍스트에 값이 없다면 => 프록시 생성.
- ① 당장 필요한 값이거나 ② 영속성 컨텍스트에 값이 있다면 => 실제 엔티티 반환.
- 프록시가 있다면? => 같은 트랜젝션 안에서는 프록시 반환.
- 즉, 괜한 짓은 안함.

=> 실무에서 프록시 자체의 메서드를 활용할 일은 거의 없음. 그러나, 이 개념이 지연로딩과 즉시로딩에 사용됨.





## 2) 지연로딩과 즉시로딩

- 앞선 예제에서, 우리는 Member를 조회할 때 Team의 데이터까지 직접적으로 하지 않다면 가짜로 불러온 척 할 수 있음을 알게 됨. => 지연로딩

  ```java
  @Entity
  public class Member {
      
      @Id @GeneratedValue
      private Long id;
      
      @ManyToOne(fetch = FetchType.LAZY) // 해당 옵션을 사용하면 지연로딩이 적용됨.
      @JoinColumn(name = "TEAM_ID")
      private Team team;
      ...
  }
  ```

  - `fetch = FetchType.LAZY` : 지연로딩
  - `fetch = FetchType.EAGER` : 즉시로딩

- 지연로딩과 즉시로딩의 실무

  - 결론: 지연로딩만 사용할 것.
  - 즉시로딩을 설정해두었다가, 해당 객체가 다른 테이블에 또 물려있으면 예상치 못한 쿼리들이 계속 발생하게 됨.
  - JPQL에서는 JPA가 최적화 해주는게 아니라 SQL 쿼리를 날려주는 것이기 때문에 Member을 로딩한 후에 Team을 한번 더 찾는 쿼리가 생기게 됨.
  - <u>즉시로딩이 필요한 경우 `EAGER`이 아니라 JPQL의 fetch 조인, 혹은 엔티티 그래프 기능을 사용하여 해결 가능!!</u>

- **`@ManyToOne` `@OneToOne`은 `fetch` 기본 설정이 `EAGER`이기 때문에 반드시 `LAZY`로 바꿔주어야 함.**
- `@OneToMany` `@ManyToMany`는 기본 설정이 `LAZY`



## 3) CASCADE (영속성 전이)

- 특정 엔티티를 영속 상태로 만들 때, 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용. (예: 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장하도록)

- Parent.java

  ```java
  @Entity
  public class Parent {
  
      @Id @GeneratedValue
      private Long id;
      private String name;
  
      @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST) // 영속성 전이 설정
      private List<Child> childList = new ArrayList<>();
  }
  ```

- Child.java

  ```java
  @Entity
  public class Child {
  
      @Id
      @GeneratedValue
      private Long id;
  
      private String name;
  
      @ManyToOne
      @JoinColumn(name = "parent_id")
      private Parent parent;
  }
  ```

- 연관관계와는 상관 없음. 그저, `em.persist();` 이런것들을 안하게 해주는 것일 뿐.

- 종류: `ALL - 모두` `PERSIST - 영속` `REMOVE - 삭제` 등...

- **<u>영속성 전이는 자식 엔티티의 연관관계가 오직 하나일 때 사용 고려.</u>** 이럴 땐 보통 부모 엔티티와 자식 엔티티의 라이프사이클이 거의 동일하기 때문에 사용하면 이점을 가져갈 수 있음. (ex, 이 게시판에서만 해당 글을 관리하고 해당 글은 다른 게시판이나 다른 곳에서 사용될 일이 전혀 없을때)

- 그러나 다른 쪽에 연관관계가 맺어져 있는 경우에는 괜히 사용했다가 큰일날 수 있음.





## 4) 고아 객체

- 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제해주는 기능.
- 연관관계 뒤에 `orphanRemoval = true`로 설정하면 활성화 됨.
- 참조하는 곳이 하나일 때만 사용.
- 부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있음.



------

> 마지막 수정일시: 2022-09-03 00:10