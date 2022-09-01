---
title: "⑤. 다양한 연관관계 매핑"
layout: single
categories: JPA
typora-root-url: ..\..\images\
toc: true
---

이 포스트 시리즈는 inflearn의 ''자바 ORM 표준 JPA 프로그래밍 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------





# 5. 다양한 연관관계 매핑





## 1) 연관관계 매핑 시 고려사항

- 다중성 : 일대일 / 일대다 / 다대일 / 다대다
- 단방향과 양방향
  - 테이블 : DB 테이블은 방향이라는 개념이 필요 없음. FK로 테이블 조인이 가능함.
  - 객체 : 참조용 필드가 있는 쪽으로만 참조 가능. 객체도 양방향은 사실 없고, 정확하게 말하자면 서로 단방향으로 바라보고 있기 때문에 양방향으로 참조가 가능해지는 것.

- 연관관계의 주인
  - 테이블 : FK 하나로 조인이 가능하기 때문에 연관관계 주인이랄 것이 없음.
  - 객체 : FK를 관리할 곳을 정해야 함. FK를 관리하는 곳이 주인.




## 2) 다대일 연관관계

- ### 다대일 단방향

![image-20220902030858068](..\..\images\image-20220902030858068.png)

- Member.java	

  ```java
  @ManyToOne // 관계가 어떻게 되는지
  @JoinColumn(name = "TEAM_ID") // Join 시 어떤 컬럼을 키값(FK)으로 사용할 것인지. DB의 컬럼 값.
  private Team team;
  ```

-  하나의 팀에 여러 맴버가 속할 수 있음. 즉 팀(하나) - 맴버(여럿)

- 맴버 입장에서는 `다대일`, 팀 입장에서는 `일대다`



- ### 다대일 양방향

  ![image-20220902031210721](..\..\images\image-20220902031210721.png)

- Team.java

  ```java
  @OneToMany(mappedBy = "team") // Member 클래스의 매핑이 되어있는 변수명
  private List<Member> members = new ArrayList<>();
  ```

- Member.java는 단방향과 동일함.
- Member가 Team의 외래키를 가지고 있기 때문에 연관관계의 주인.
- 보통 `다대일`연관관계에서는 `다` 쪽이 FK를 갖고 있기 때문에 연관관계의 주인이 됨.



## 3) 일대다 연관관계

- ### 일대다 단방향

![image-20220902031757627](..\..\images\image-20220902031757627.png)

- Team.java

  ```java
  @OneToMany
  @JoinColumn(name = "TEAM_ID")
  private List<Member> members = new ArrayList<>();
  ```

  - 권장하지 않음. 왜냐하면, 실제로 DB에 값이 들어가려면 결국 Member에 쿼리를 날려주어야 하기 때문. (실제로 Update쿼리가 날아감)
  - 나는 Team에 메서드작업을 했는데, 쿼리가 Member로 날아간다?



- ### 일대다 양방향

![image-20220902032319715](..\..\images\image-20220902032319715.png)

- 굳이 이러지 맙시다. 그냥 다대일 양방향을 사용하자.





## 4) 일대일 연관관계

- 일대일은 서로 일대일. 외래키를 어디에 두느냐(즉, 연관관계의 주인이 누구냐)에 따라 매핑방법이 반대가 됨.



- ### 일대일 단방향(주 테이블에 외래키)

  ![image-20220902032443954](..\..\images\image-20220902032443954.png)

  - 한 회원당 라커 하나!

- Member.java

  ```java
  @OneToOne
  @JoinColumn(name = "LOCKER_ID")
  private Locker locker;
  ```





- ### 일대일 양방향(주 테이블에 외래키)

  ![image-20220902032909817](..\..\images\image-20220902032909817.png)

- Locker.java

  ```java
  @Entity
  public class Locker {
  
      @Id @GeneratedValue
      private Long id;
  
      private String name;
  
      @OneToOne(mappedBy = "locker") // 양방향이 되면 반대 테이블에 이렇게 mappedBy 적용해주어야 함.
      private Member member;
  }
  ```



- ### 일대일 단방향(대상 테이블 외래키)

  ![image-20220902033431279](..\..\images\image-20220902033431279.png)

  - 일대일 단방향 관계는 JPA에서 지원하지 않음.
  - 일대일은 양방향 관계로 기본적으로 설정할 것.



- ### 일대일 양방향(대상 테이블 외래키)

  ![image-20220902033511577](..\..\images\image-20220902033511577.png)

  - 사실상 말장난임. 이건 결국 `Locker`가 연관관계의 주인이 되는거랑 동일함.

- ### 일대일 정리

  - 주 테이블에 외래 키
    - 객체지향 개발자가 선호하는 편. JPA 매핑이 편리하고 주 테이블만 조회해도 대상 테이블의 데이터 확인 가능.
    - 값이 없으면 외래키에도 null이 가능해짐. (사실 좀 말이 안되긴 함)
  - 대상 테이블에 외래 키
    - DB관점에서 보면 맞긴 함.

  => <u>결국, 나중에 어떤 테이블이 더 확장가능성이 많은지를 따져서 해당 테이블을 연관관계 주인으로 가져가는 것이 유리할 가능성이 있음!!</u>





## 5) 다대다 연관관계

- 관계형 DB는 다대다 연관관계를 테이블로 표현하는 것이 불가능함.
- 테이블 관점에서 결국, 다대다는 가운데 조인테이블을 두어 다대일 - 일대다 관계로 한단계 거쳐야 함.
- 객체는 JPA에서 매핑이 가능하지만, JPA에게 맡겨 추상적으로 관리하기 보다는 DB 테이블과의 관계를 위해서라도 조인테이블을 `엔티티`로 승격시켜 관리하면 좋음!

![image-20220902034034863](..\..\images\image-20220902034034863.png)

- 이렇게 조인테이블에도 다른 정보들이 들어갈 가능성이 생김!

  ![image-20220902034104532](..\..\images\image-20220902034104532.png)







## 6) 연관관계 기본 매핑 예시

(Getter, Setter 코드는 생략)

- **DELIVERY, CATEGORY 추가!**

![image-20220902034131760](..\..\images\image-20220902034131760.png)

![image-20220902034148412](..\..\images\image-20220902034148412.png)

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
    
    // 추가된 내용
    @OneToOne // 일대일이며 FK갖고 있기 때문에 연관관계의 주인으로 사용
    @JoinColumn(name = "DELIVERY_ID") // 따라서 조인컬럼 사용
    private Delivery delivery;

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

### - Delivery.java

```java
package jpabook.jpashop.domain;


import javax.persistence.*;

@Entity
public class Delivery {

    @Id @GeneratedValue
    private Long id;

    @OneToOne(mappedBy = "delivery") // 일대일 연관관계에서 상대테이블이 주인이기 때문에 mappedBy
    private Order order;

    private String city;
    private String street;
    private String zipcode;

    @Enumerated(EnumType.STRING)
    private DeleveryStatus status;
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
    
    // 추가부분
    @OneToMany
    @JoinColumn(name = "CATEGORY_ID")
    private List<Category> categories = new ArrayList<>();

    private String name;
    private int price;
    private int stockQuantity;
}
```

### - Category.java

```java
import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Category {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    /* 이 부분은 나중에 추가학습
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Category parent;

    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();
    */

    // 추가부분
    @OneToMany(mappedBy = "item")
    private List<Item> items = new ArrayList<>();

}
```

### - Category_Item.java (조인테이블의 엔티티화)

```java
import javax.persistence.*;

@Entity
public class Category_Item {

    @Id @GeneratedValue // 새로 테이블의 Id 생성
    private Long id;

    @ManyToOne
    @JoinColumn(name = "CATEGORY_ID")
    private Category category;

    @ManyToOne
    @JoinColumn(name = "ITEM_ID")
    private Item item;
}
```



- 다대다 연관관계를 가운데 조인테이블을 엔티티로 만들어 일대다 - 다대일 관계로 풀어냄.



------

> 마지막 수정일시: 2022-09-02 03:53