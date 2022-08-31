---
title: "③. 엔티티 매핑"
layout: single
categories: JPA
typora-root-url: ..\..\images\
toc: true
---

이 포스트 시리즈는 inflearn의 ''자바 ORM 표준 JPA 프로그래밍 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------





# 3. 엔티티 매핑



## 1) 객체와 테이블 매핑

### 1. @Entity

- 엔티티로 사용할 클래스에 붙이는 어노테이션.
- 해당 어노테이션이 붙으면 JPA가 관리하게 되고, 테이블과 매핑이 됨.
- `기본 생성자 필수` : JPA같은 내부 구현을 하는 라이브러리는 동적으로 어떠한 작업을 할 때(ex, reflection 등) 기본 생성자를 필요로 함.
- `@Entity(name = "엔티티명")` : 매핑할 엔티티 이름을 직접 지정. 설정하지 않으면 기본값은 엔티티 클래스명과 동일하게 사용함.(대소문자는 구별하지 않음)

### 2. @Table

- 엔티티로 사용할 클래스에 붙이는 어노테이션.
- `@Table(name = "테이블명")` : 매핑할 테이블이름을 직접 지정. 설정하지 않으면 기본값은 엔티티명과 동일하게 사용함.(대소문자는 구별하지 않음)
- `@Table(catalog = "catalog명")` , `@Table(schema = "schema명")` : 데이터베이스 catalog, schema 매핑.
- `@Table(unmiqueConstraints = )` : DDL 생성 시에 유니크 제약 조건 생성.

```java
@Entity(name="member")
@Table(name="MEMBER")
public class Member {
    
}
```



**DB 스키마 자동생성?*

- 애플리케이션이 실행될 때 스키마를 어떻게 할 것인지에 대한 옵션 세팅이 가능함.
- 이를 통해 테이블 중심에서 객체 중심 개발이 더욱 가능해짐.
- 단, 속성값으로 자동으로 실행되다 보니 중요한 DB를 다룰 때는 정말 조심해서 사용해야 함. `특히 실제 운영서버의 DB는 정말 중요함`

```xml
<properties>
	<property name="hibernate.hbm2ddl.auto" value="update" />
</properties>
```

- 위 코드에서 value값 세팅이 가능함. (애플리케이션 실행 시 어떻게 할지)
  1. create : 기존 테이블 삭제 후 다시 생성 (DROP + CREATE)
  2. create-drop : create + 종료시점에 drop까지 진행
  3. update : 변경분만 반영
  4. validate : 정상적으로 엔티티와 테이블이 매핑이 되었는지만 확인
  5. none : 사용하지 않음





## 2) 필드와 컬럼 매핑

### 1. @ Column

- `name` : 필드와 매핑할 테이블의 컬럼 이름. 기본값은 `객체의 필드 이름`
- `insertable, updatable` : 등록 및 변경 가능여부. 기본값은 `TRUE`
- `nullable(DDL)` : null값 허용 여부. `false`설정 시 NOT NULL 제약조건 붙음.
- `length(DDL)` : 문자 길이 제약조건, String 타입에만 사용. 기본값 `255`

### 2. @Enumerated

- Enum 타입 매핑.

  - `EnumType.ORDINAL` : enum 순서를 DB에 저장.<br>ex) enum 클래스에 'USER', 'ADMIN' 순으로 적혀있다면, USER는 '0', ADMIN은 '1' 값을 가지게 됨.
  - `EnumType.STRING` : enum 이름 그대로 DB에 저장.

  => 기본값은 `ORDINAL`인데, `ORDINAL`은 enum클래스에서 데이터가 추가되어 순서가 바뀌게 되면 기존 데이터들에 문제가 생기기 때문에 쓰지말자! 

  => `@Enumerated(EnumType.STRING)` 전체가 세트라고 생각하자.

### 3. @Temporal

- 날짜 타입 매핑.
- 요즘은 LocalDate, LocalDateTime 클래스를 사용하기 때문에 거의 안씀.
- `TemporalType.DATE` : 날짜 매핑
- `TemporalType.TIME` : 시간 매핑
- `TemporalType.TIMESTAMP` : 날짜 및 시간 매핑

### 4. @Lob

- 엄청나게 큰 데이터 매핑
- 따로 속성이 없으며, 필드 타입이 문자면 `CLOB`, 나머지는 `BLOB`으로 자동 설정 됨.

### 5. @Transient

- 매핑하지 않음.



## 3) 기본키 매핑(Primary key)

### 1. @Id

- 해당 필드를 Primary Key로 지정.

### 2. @GeneratedValue

- Primary Key 생성 전략.

- `strategy = "전략이름"`

- #### `GenerationType.IDENTITY`

  - DB에 위임하는 전략.
  - 자동으로 1씩 증가시켜 pk값을 관리함.

  - 주로 MySQL, PostgreSQL, SQL Server, DB2에 사용.

  - ##### 사용 이슈

    - JPA는 트랜잭션이 커밋할 때 Insert SQL을 실행한다고 배웠다.
    - 자동증가하는 pk값은 DB에 Insert가 되어야 알 수 있다.
    - 영속성 관리를 위해서는 pk값을 알아야(1차 캐시에 key - value값으로 저장될 때 key가 바로 pk값이므로) 관리가 되는데, 아직 트랜잭선.commit()을 안했다면 어떡하지?

  - ##### 해결책

    - `IDENTITY` 전략은 `em.persist()`시점에 즉시 INSERT SQL을 실행하여 DB에 pk 값을 조회할 수 있게 함.

- #### `GenerationType.SEQUENCE` 

  - DB 시퀀스 오브젝트(DB에 유일한 값을 순서대로 생성하는 DB 오브젝트)를 사용하여 키를 생성. @SequenceGenerator 필요. 

  - Oracle이 대표적이며, PostgreSQL, DB2, H2 DB에서 사용.

  - `IDENTITY` 전략과 다르게 SQL을 실행하지 않고 `call next`하여 SEQUENCE만 먼저 받아서 사용함.

  - 기본값은 1씩 증가하는 세팅이지만, @SequenceGenerator에 옵션값 세팅 가능.

    ```java
    @Entity
    @SequenceGenerator(
    	name = "MEMBER_SEQ_GENERATOR", // Generator 이름
    	sequenceName = "MEMBER_SEQ", // 매핑할 DB 시퀀스 이름
    	initialValue = 1, allocationSize = 1) // 초기값 / call next 마다의 증가값
    public class Member {
        
        @Id
        @GeneratedValue(strategy = GenerationType.SEQUENCE,
                       generator = "MEMBER_SEQ_GENERATOR")
        private Long id;
        
    }
    ```

    **allocationSize* - 매 pk값이 필요할때 시퀀스 한 번 호출에 증가하는 수. 이를 통해 성능 최적화가 가능해짐. 기본값은 50. (DB 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 이 값을 반드시 1로 설정해야 함.)

    **작동 원리* - start with 1 increase by 50으로 call next value를 하면 50씩 시퀀스를 미리 세팅함. 그리고 나서 매 1씩 call next하지 않고 세팅한걸 다 쓸때까지는 메모리에서만 작동함. -> 네트워크를 타지 않아서 성능이 상승함. -> 미리 값을 올려놓는 방식이기 때문에 동시성 이슈도 없음. (자기가 미리 확보를 해놓기 때문에)

- #### `GenerationType.TABLE` 

  - 시퀀스 대신 TABLE을 사용하는 전략.
  - 모든 DB에 사용이 가능하지만, 실제 TABLE을 사용하기 때문에 성능이 안좋아짐.



### 3. 그래서 기본키는 어떻게 관리하는데?

- 기본 키의 제약 조건을 먼저 따져보자.
- Null이면 안됨! 유일해야 함! 변하면 안됨! => 이러한 조건을 만족하는 키를 찾는 것이 쉽지는 않음.
- 특히, 비즈니스와 연관된 값을 키값으로 사용하는 것은 매우 위험함(예를 들어 핸드폰번호, 주민번호, 주문번호 등등)
- **`GeneratedValue`를 사용하며, 타입은 `Long`을 사용하는 것이 바람직할 것!**



## 4) 사용 예시

```java
@Entity // JPA가 관리할 객체
public class Member {

    @Id // DB pk와 매핑
    @GeneratedValue
    private Long id;
    @Column(name = "name") // DB 컬럼명
    private String username; // 객체 변수명

    private Integer age;

    @Enumerated(EnumType.STRING) // DB는 Enum타입이 없어서 이렇게 어노테이션 해줌
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP) //Data, Time, TimeStamp
    private Date createDate; // -> 요즘은 Date 말고 LocalDate, LocalDateTime 하면 됨.

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob //C Lob(문자), B Lob(나머지) (큰 컨텐츠)
    private String description;

    public Member(){
    }

    public Member(Long id, String name) {
        this.id = id;
        this.username = name;
    }
}
```





## 5) 기본 매핑 예시

(Getter, Setter 코드는 생략)

![image-20220831044319033](..\..\images\image-20220831044319033.png)

### - Member.java

```java
import javax.persistence.*;

@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name="MEMBER_ID")
    private Long id;

    private String name;
    private String city;
    private String street;
    private String zipcode;

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

    @Column(name="MEMBER_ID")
    private Long memberId;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

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

    @Column(name="ORDER_ID")
    private Long orderId;

    @Column(name="ITEM_ID")
    private Long itemId;

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

### - 해당 설계들의 문제점

- 엔티티 코드는 잘 작성되었지만, Order클래스에서 Member클래스로 바로 접근이 불가능함.
- 즉, 현재 방식은 `DB Table` 설계에 맞춘 방식.
- `Order -> MemberId -> Member` 이렇게 한 단계를 더 거쳐아 함.

<u>*=> 연관관계 매핑이 필요한 시점!*</u>

------

> 마지막 수정일시: 2022-08-31 04:43