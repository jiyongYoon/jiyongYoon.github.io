---
title: "⑥. 고급 매핑 - 상속관계와 MappedSuperclass"
layout: single
categories: JPA
typora-root-url: ..\..\images\
toc: true
---

이 포스트 시리즈는 inflearn의 ''자바 ORM 표준 JPA 프로그래밍 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------





# 6. 고급 매핑 - 상속관계와 MappedSuperclass





## 1) 상속관계 매핑

- 객체는 상속관계가 있지만, 데이터베이스 테이블은 상속관계가 없음(기본적으로)
- 슈퍼타입 - 서브타입 관계가 객체 상속과 유사하기 때문에 이를 활용하여 매핑을 함.

![image-20220902131936505](..\..\images\image-20220902131936505.png)

### 1. 조인전략(`@Inheritance(strategy = InheritanceType.JOINED)`)

![image-20220902132035734](..\..\images\image-20220902132035734.png)

- Item.java

  ```java
  @Entity
  @Inheritance(strategy = InheritanceType.JOINED)
  public class Item {
  
      @Id @GeneratedValue
      private Long id;
  
      private String name;
      private int price;
  
  }
  ```

- Album.java

  ```java
  @Entity
  public class Album extends Item {
      private String artist;
  }
  ```

- Movie.java

  ```java
  @Entity
  public class Movie extends Item {
      private String director;
      private String actor;
  }
  ```

- Book.java

  ```java
  @Entity
  public class Book extends Item {
      private String author;
      private String isbn;
  }
  ```

- 객체 패러다임과 잘 맞고 테이블을 정규화 할 수 있음.

- 저장 시 - Insert 쿼리가 2번 호출됨. 상속하는 테이블과 상속받는 테이블에 각각 저장해야 하기 때문. (각 테이블의 id는 PK-FK값으로 동일하게 매핑되고, 나머지 정보는 필드가 있는 테이블에 넣어야 하기 때문.)

- 조회 시 - 조인을 사용.

- 두 번째 전략인 단일테이블에 비해서 테이블을 많이 사용하기 때문에 나타나는 단점이 있음. (결국 Trade-off 관계임)



### 2. 단일 테이블 전략(`@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`)

![image-20220902132558253](..\..\images\image-20220902132558253.png)

- 테이블을 하나만 만들고 테이블 내에서 DTYPE으로 구분하는 것.

- Item.java

  ```java
  @Entity
  @Inheritance(strategy = InheritanceType.SINGLE_TABLE) // 변하는 부분
  @DiscriminatorColumn(name = "DTYPE") // DTYPE이 기본값. ITEM 테이블에 DTYPE 컬럼이 추가되며, 이 컬럼에 자식엔티티명이 들어가서 구분지음.
  public class Item {
  
      @Id @GeneratedValue
      private Long id;
  
      private String name;
      private int price;
  
  }
  ```

- Album.java

  ```java
  @Entity
  public class Album extends Item {
      private String artist;
  }
  ```

- Movie.java

  ```java
  @Entity
  public class Movie extends Item {
      private String director;
      private String actor;
  }
  ```

- Book.java

  ```java
  @Entity
  public class Book extends Item {
      private String author;
      private String isbn;
  }
  ```

- 저장 및 조회에서 다른 테이블을 사용할 필요가 없음.

- 하나에 뭉쳐서 쓰기 때문에 테이블 크기가 커질 수 있음.

- 단, 자식 엔티티가 매핑한 컬럼은 null이 허용될 수 밖에 없음.

- **`@DiscriminatorColumn` & `@DiscriminatorValue("대체할 이름")`**
  - 단일테이블 전략 사용 시, 한 테이블에 정보가 다 들어가기 떄문에 어떤 자식엔티티인지 구분할 컬럼이 필요함.
  - DTYPE 컬럼을 사용하여 구분. **=> 조인전략도 결국 있는게 좋음. 조인전략 테이블을 까보면 ITEM 테이블 값만 보면 어떤 자식 엔티티 값인지 구분이 안됨. *
  - 부모엔티티에 `@DiscriminatorColumn`, DTYPE이 아니라 다른 컬럼명도 (name = "컬럼명") 사용 가능.
  - DTYPE 값을 다른걸 사용하고 싶다면 자식엔티티에 `@DiscriminatorValue("대체할 이름")`



### 3. 구현 클래스마다 테이블 전략(`@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`)

![image-20220902132849548](..\..\images\image-20220902132849548.png)

- 상속 테이블이 없음.
- 각자 도생.
- 조회 시 UNION 쿼리 필요. (자식 테이블을 다 뒤져봐야 함.)
- 개발자와 DB담당자 모두 선호하지 않음!!





### 4. 그럼 어떤 전략을 써야 하나?

- 기본적으로 조인전략. 정석적이며 객체 패러다임과 잘 맞기 때문.
- 조인전략 - 단일 테이블 전략의 Trade-off를 생각하여 선택.
  - 확장 가능성이 낮고 단순한 편이면 단일 테이블 전략을 가져가는 것이 단순하고 좋음.
  - 비즈니스적으로 굉장히 중요한 테이블이며, 확장 가능성이 있따면 조인전략을 사용하는 것이 추후 유연한 확장에 용이함.
- 다만, 현실세계처럼 복잡해지기 때문에 데이터가 많아질수록 관리 및 유지보수가 어려워질 수 있음. 인지의  범위를 넘어서는 경우가 생길 수 있음.
- 때문에 처음에는 조인전략으로 가되, 데이터가 많아져서 조인전략의 장점보다 단점이 커지는 시점이 되는 경우 시스템을 개편하고 설계를 변경하는 등의 방식으로 유지보수를 할 것을 추천.







## 2) MappedSuperclass

- 공통 매핑정보가 필요할 때 사용. (마치 Java의 인터페이스와 비슷)
- 예를들어, 우리 어플리케이션에는 모든 테이블에 항상 있어야 하는 정보들이 있다면 (createDt, createBy, modifiedDt, modifiedBy 등등) 이러한 속성만 상속받아서 사용 가능.
- 따라서 추상클래스로 만드는 것이 좋음.
- 엔티티가 아님. 테이블도 생성이 안되기 떄문에, 해당 클래스를 사용하여 조회, 저장 등은 불가능.

![image-20220902134339501](..\..\images\image-20220902134339501.png)

- BaseEntity.java

  ```java
  @MappedSuperclass
  public abstract class BaseEntity {
      
      @Id @GeneratedValue
      private Long id;
      
      private String name;
      
      // 아래는 이런식의 정보들이 보통 MappedSuperclass에 많이 활용된다는 예시
      private String createBy;
      private LocalDateTime createDt;
      private String modifiedBy;
      private LocalDateTime modifiedDt;
  
   }
  ```

- Member.java

  ```java
  @Entity
  public class Member extends BaseEntity {   
      private String email;
  }
  ```

- Seller.java

  ```java
  @Entity
  public class Seller extends BaseEntity {   
      private String shopName;
  }
  ```

  





## 3) 연관관계 기본 매핑 예시

(Getter, Setter 코드는 생략)

- **Item의 자식클래스, 모든 클래스에 등록일과 수정일 추가!**

![image-20220902134757404](..\..\images\image-20220902134757404.png)

![image-20220902134846768](..\..\images\image-20220902134846768.png)

### - Item.java

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE) // 단일 테이블 전략
@DiscriminatorColumn // DTYPE 사용
public class Item extends BaseEntity { // MappedSuperclass 사용

    @Id
    @GeneratedValue
    @Column(name="ITEM_ID")
    private Long id;
    
    @OneToMany
    @JoinColumn(name = "CATEGORY_ID")
    private List<Category> categories = new ArrayList<>();

    private String name;
    private int price;
    private int stockQuantity;
}
```

### - Album.java

```java
import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Album extends Item {

    private String artist;
    private String etc;
}
```

### - Book.java

```java
import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Book extends Item {

    private String author;
    private String isbn;
}
```

### - Movie.java

```java
import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Movie extends Item {

    private String director;
    private String actor;
}
```

### - BaseEntity.java (MappedSuperclass)

```java
import java.time.LocalDateTime;

@MappedSuperclass
public abstract class BaseEntity {

    private String createBy;
    private LocalDateTime createDt;
    private String modifiedBy;
    private LocalDateTime modifiedDt;

}
```



------

> 마지막 수정일시: 2022-09-02 13:53