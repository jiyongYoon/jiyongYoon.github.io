---
title: "④. 연관관계 매핑"
layout: single
categories: JPA
typora-root-url: ..\..\images\
toc: true
---

이 포스트 시리즈는 inflearn의 ''자바 ORM 표준 JPA 프로그래밍 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------





# 4. 연관관계 매핑

이 강의를 제대로 이해하기 위해 관계형 DB에 대해서 더 공부하고 왔다.

------



## 1) 연관관계가 없이 테이블에 맞춘 모델링

![image-20220901024303588](..\..\images\image-20220901024303588.png)

- Member.java

  ```java
  @Entity // JPA가 관리할 객체
  public class Member {
  
      @Id @GeneratedValue
      @Column(name = "MEMBER_ID") // 생략 가능
      private Long id;
      
      //  DB 테이블 기준 매핑
      @Column(name = "USERNAME") // DB 컬럼명
      private String username; // 객체 변수명    
      @Column(name = "TEAM_ID")
      private Long teamId;
  ```

- Team.java

  ```java
  @Entity
  public class Team {
  
      @Id @GeneratedValue
      @Column(name = "TEAM_ID") // 생략 가능
      private Long id;
  
      private String name;
  }
  ```

- Main.java

  ```java
  // 팀 저장
  Team team = new Team();
  team.setName("TeamA");
  em.persist(team);
  // 맴버 저장
  Member member = new Member();
  member.setUsername("member1");
  member.setTeamId(team.getId());
  em.persist(member);
  // 조회
  Member findMember = em.find(Member.class, member.getId());
  Long findTeamId = findMember.getTeamId();
  Team findTeam = em.find(Team.class, findTeamId);
  ```

- 팀까지 가려면 식별자로 다시 재조회를 해야 함.





## 2) 단방향 연관관계

![image-20220901025056734](..\..\images\image-20220901025056734.png)

- Member 테이블에 Team 객체가 추가됨.

  

- Member.java

  ```java
  @Entity // JPA가 관리할 객체
  public class Member {
  
      @Id @GeneratedValue
      @Column(name = "MEMBER_ID") // 생략 가능
      private Long id;
      
      //  DB 테이블 기준 매핑
      @Column(name = "USERNAME") // DB 컬럼명
      private String username; // 객체 변수명
      /*
      @Column(name = "TEAM_ID")
      private Long teamId;
      */
      
      // 객체를 직접 주입
      @ManyToOne // 연관관계. Member 객체가 Many, 연결되는 객체가 One이라는 뜻.
      @JoinColumn(name = "TEAM_ID") // Join 컬럼은 FK 키 컬럼이 되겠지. 그걸로 Team 테이블에 가서 찾을테니.
      private Team team;
  }
  ```

- Team.java

  ```java
  @Entity
  public class Team {
  
      @Id @GeneratedValue
      @Column(name = "TEAM_ID") // 생략 가능
      private Long id;
  
      private String name;
  }
  ```

- Main.java

  ```java
  // 팀 저장
  Team team = new Team();
  team.setName("TeamA");
  em.persist(team);
  // 맴버 저장
  Member member = new Member();
  member.setUsername("member1");
  // member.setTeamId(team.getId());
  member.setTeam(team); // 드디어 team을 저장할 수 있게 됨
  em.persist(member);
  // 조회
  Member findMember = em.find(Member.class, member.getId());
  Team findTeam = findMember.getTeam(); // 참조를 사용하여 팀 바로 조회
  // 팀 변경
  Team team2 = new Team();
  team.setName("TeamB");
  em.persist(team2);
  findMember.setTeam(team2);
  ```





## 3) 양방향 연관관계

![image-20220901030007245](..\..\images\image-20220901030007245.png)

- 각 테이블에 필요한 객체가 포함됨. (Member는 Team, Team은 Member 리스트)



- Member.java

  ```java
  @Entity
  public class Member {
  
      @Id @GeneratedValue
      @Column(name = "MEMBER_ID") // 생략 가능
      private Long id;
      
      @Column(name = "USERNAME") // DB 컬럼명
      private String username; // 객체 변수명
      /*
      @Column(name = "TEAM_ID")
      private Long teamId;
      */
      
      // 객체를 직접 주입
      @ManyToOne // 연관관계. Member 객체가 Many, 연결되는 객체가 One이라는 뜻.
      @JoinColumn(name = "TEAM_ID") // Join 컬럼은 FK 키 컬럼이 되겠지. 그걸로 Team 테이블에 가서 찾을테니.
      private Team team;
  }
  ```

- Team.java

  ```java
  @Entity
  public class Team {
  
      @Id @GeneratedValue
      @Column(name = "TEAM_ID") // 생략 가능
      private Long id;
  
      private String name;
      
      @OneToMany(mappedBy = "team") // Member 클래스의 매핑이 되어있는 변수명
      private List<Member> members = new ArrayList<>(); // 초기화하여 null값이 되지 않게 하기 위한 컨벤션
  }
  ```

- Main.java

  ```java
  Team team2 = new Team();
  team2.setName("Team2");
  em.persist(team2);
  
  Member member2 = new Member();
  member2.setUsername("member2");
  member.setTeam(team2);
  em.persist(member);
  
  em.flush(); // 쓰기 지연 SQL 저장소에 있는 쿼리를 보내 DB에 insert 하기
  em.clear(); 
  // 그래야지 get 할 때 PK값을 받아올 수 있음.
  
  Member findMember2 = em.find(Member.class, member.getId());
  Team findTeam2 = findMember2.getTeam();
  
  // 역방향으로 객체 조회도 가능함.
  List<Member> members = findMember.getTeam().getMembers();
  for (Member m : members) {
      System.out.println("m = " + m.getUsername());
  }
  ```



### **(중요) 연관관계의 주인과 `mappedBy`*

- 테이블의 연관관계는 연관이 되는 순간 PK - FK 로 양쪽 테이블을 Join하여 데이터 조회가 가능.

- 객체는 객체를 담고있는 곳만 참조를 사용하여 접근 가능.

- 이러한 특성 때문에 객체에서의 양방향 연관관계는 사실상 단방향 연관관계를 서로 맺고 있다고 생각해야함.

  - `Member` -> `Team`

  - `Team` -> `List<Member>`

    ![image-20220901031220825](..\..\images\image-20220901031220825.png)

- 그렇다면 이때, member중 한 사람의 Team이 변경되었다면, 어느 값을 바꾸어야 하나? 그리고 DB입장에서는 어디값을 매핑받아와야 하나? **<u>*==> 둘 중 하나로 외래키의 주인을 정해주어야 한다!!!*</u>**



### **(중요) 외래키가 있는 곳을 주인으로!*

- `Member`가 `Team`의 외래키를 가지고 있음. 그러므로 `Member`가 연관관계의 주인이다. (1:N 관계에서 N 테이블)
- 그렇기 때문에 `Team.java`에서 `List<Member>` 위에 `@OneToMany(mappedBy = "team")`가 선언되는 것. 이 뜻은, 이 연관관계의 주인은 `Member.java`의 변수 중 변수명이 `team`인 곳에서 가지고 온다는 뜻. (Member.java의 FK값)
- `mappedBy = "team"` : "team"에 의해 매핑되었다!



#### **그러면 team 리스트는?*

- 추가하지 않아도 됨. 그러나, JPA 동작에 문제가 발생할 수 있음.
- JPA는 1차캐시를 사용하여 영속성 관리를 하게 되는데, 그러다보니 SQL 쿼리가 동작하지 않은 상태에서 값을 가져오게 되면 null값이 될 수 있음.

![image-20220901032526805](..\..\images\image-20220901032526805.png)

- `team.getMembers().add(member);`를 하지 않는다면?
  - `em.flush()` 호출 시 문제 없음. 쿼리가 날아가 DB에 insert 되기 때문.
  - `em.flush()` 미 호출시 문제 생길 수 있음.
    1. `persist()`를 하면 1차 캐시(영속 관리)에 들어가게 됨.
    2. 그러나 실제로 컬랙션 Team team에는 값이 들어가지 않음.
    3. `em.find()`를 해도 찾아지는 member가 없음. (DB에 값이 없기 때문)
    4. 나중에 테스트코드시에도 이러한 결의 문제가 발생할 수 있음.
- `결국 양쪽 값을 다 세팅하는 것이 바람직함`
  - 객체지향 관점에서 봐도, 영속성 컨텍스트 관점에서 봐도 `setTeam()`과 `getMembers().add()`를 다 해주는 것이 좋음.
  - 그러나, 사람인지라 빠트릴수도 있지 않을까?





## 4) 양방향 연관관계에서의 이론과 실무

- 양방향 매핑 시 위 문제점을 방지하기 위해 `setter`나 또는 `add()`와 같은 메서드에 미리 추가 세팅을 해두는 것이 좋음. (`setter`가 아닌 `add`와 같이 메서드를 따로 빼면 코드를 보고 명시적으로 어떤 작업을 하는 지 알 수 있어서 좋음)

  ```java
  [Member.java]
  
  public void setTeam(Team team) {
      this.team = team;
      team.getMembers().add(this); // 이렇게 아예 추가를 해버리면 실수가 줄어듬.
  }
  ```

- 양방향 매핑 시 무한 루프가 생길 수 있음.
  - `lombok - toString()` : toString() 안에 상대 객체를 서로 호출하게 되면 DB값이 세팅되어 있지 않은 상태에서는 스택오버플로우 가능성 있음.
  - JSON 생성 라이브러리 : `Entity`를 직접 반환하게 되면 생길 수 있음. 정석적으로 필요한 값을 담은 `Dto`를 반환하여 해결하자!







## 5) 연관관계 기본 매핑 예시

(Getter, Setter 코드는 생략)

![image-20220831044319033](..\..\images\image-20220831044319033.png)

### - Member.java

```java
import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Member {

    public List<Order> getOrders() {
        return orders;
    }

    public void setOrders(List<Order> orders) {
        this.orders = orders;
    }

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name="MEMBER_ID")
    private Long id;

    private String name;
    private String city;
    private String street;
    private String zipcode;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

    // 주문 메서드에서 객체 양방향에 값이 들어가도록 세팅
    public void addOrder(Order order) {
        orders.add(order);
        order.setMember(this);
    }
}
```

### - Order.java

```java
import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name="ORDERS")
public class Order {
    @Id
    @GeneratedValue
    @Column(name="ORDER_ID")
    private Long id;

//    @Column(name="MEMBER_ID")
//    private Long memberId;
    
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    // 메서드에서 객체 양방향에 값이 들어가도록 세팅
    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }
}
```

### - OrderStatus.java

```java
public enum OrderStatus {
    ORDER, CANCEL
}
```

### - OrderItem.java

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class OrderItem {

    @Id
    @GeneratedValue
    @Column(name = "ORDER_ITEM_ID")
    private Long id;

//    @Column(name="ORDER_ID")
//    private Long orderId;
    
    @ManyToOne
    @JoinColumn(name = "ORDER_ID")
    private Order order;

//    @Column(name="ITEM_ID")
//    private Long itemId;
    
    @ManyToOne
    @JoinColumn(name = "ITEM_ID")
    private Item item;

    private int orderPrice;
    private int count;
}
```

### - Item.java

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Item {

    @Id
    @GeneratedValue
    @Column(name="ITEM_ID")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;
}
```

### -  양방향 매핑 정리

- 단방향 매핑만으로도 이미 어플리캐이션은 작동준비 완료.

- 양방향 매핑은 단방향 매핑 + 객체 그래프탐색 기능이 추가된 것.

- JPQL에서는 역방향 탐색할 일이 많이 생김.

  **<u>=> 단방향 매핑을 잘 해서 설계를 한 후, 양방향은 필요할 때 추가하면 됨. 양방향 매핑은 테이블에 영향을 주지 않기 때문에 설계단계에서 신경쓰지 않아도 됨.</u>**<br>(위 예시에서도 Team클래스에 List<Member> 객체만 넣어주면 됐다.)

------

> 마지막 수정일시: 2022-09-01 03:45