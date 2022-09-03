---
title: "⑧. 값 타입 매핑"
layout: single
categories: JPA
typora-root-url: ..\..\images\
toc: true
---

이 포스트 시리즈는 inflearn의 ''자바 ORM 표준 JPA 프로그래밍 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------



# 8. 값 타입 매핑





## 1) JPA의 데이터 타입 분류

### - 엔티티 타입

- `@Entity`로 정의하는 객체

- 식별자가 있어 추적 가능

  


### - 값 타입

- 단순히 값으로 사용하는 자바 기본 타입이나 객체

- 식별자가 없어 추적 불가능

  - #### 기본값 타입

    - int, double, Integer, Long, String 등..

  - #### 임베디드 타입

    - 복합 값 타입. 기본값 타입을 조합하여 사용자가 만드는 타입

    ```java
    @Entity
    public class Member {
        
    	@Id @GeneratedValue
        private Long id;
        
        /* 이렇게 값을 나누어 사용할수도 있지만
        private String city;
    	private String street;
    	private String zipcode;
    	*/
        @Embedded
        private Address address; // 이런식으로 임베디드 타입을 사용.
    }
    ```

    ```java
    @Embeddable
    public class Address {
    
        private String city;
        private String street;
        @Column(length = 5)
        private String zipcode;
        
        // 장점: 이런식으로 메서드를 만들어 사용할 수 있음.
        public String fullAddress() {
            return getCity() + " " + getStreet() + " " + getZipcode();
        }
    }
    ```

    - 재사용이 가능해지며, 높은 응집도를 가지게 됨. -> 더욱 객체지향적인 설계가 가능.

    - 의미있는 메서드를 만들어서 사용 가능.

    - 임베디드 타입을 포함한 모든 값 타입의 생명주기는 값 타입을 소유한 엔티티를 의존하게 됨. (Member가 없어지면 Address도 없어짐)

    - `@Embeddable` : 값 타입을 정의한 곳에 표시

    - `@Embedded` : 값 타입을 사용하는 곳에 표시

    - **기본 생성자가 필수!**

    - **매핑하는 테이블은 같다.** (테이블에는 `ADDRESS`가 아니라 `CITY` `STREET` `ZIPCODE` 로 매핑 됨.)

    - 임베디드 타입도 필드값으로 엔티티를 가질 수 있음.

    - 한 엔티티에서 같은 값 타입을 사용하게 되면? => 오버라이딩 사용 가능.

      ```java
      @Embedded
      private Address homeAddress;
      
      @Embedded
      @AttributeOverrides(
          @AttributeOverride(name="city", column = @Column(name = "WORK_CITY")),
          @AttributeOverride(name="street", column = @Column(name = "WORK_STREET")),
          @AttributeOverride(name="zipcode", column = @Column(name = "WORK_ZIPCODE"))
      )
      private Address workAddress;
      ```

      - 이렇게 되면 매핑테이블에 다른 컬럼으로 매핑이 됨.

    

  - #### 컬렉션 값 타입

    - List, Set 등의 컬렉션.

    - 컬렉션 값 타입에서는 Java의 인스턴스 개념이 그대로 적용됨. 즉, 객체의 참조값을 바라보고 있을 때 수정이 되면 여기저기 값이 다 수정될 수 있음을 인지해야 함.

      ```java
      Address address = new Address("city", "street", "10000");
      
      Member member = new Member();
      member.setUsername("member1");
      member.setHomeAddress(address); // address 인스턴스 값 넣음
      em.persist(member);
      
      Member member2 = new Member();
      member2.setUsername("member2");
      member2.setHomeAddress(address); // 같은 인스턴스를 넣음
      em.persist(member);
      
      
      member.getHomeAddress().setCity("city2"); // 이렇게 바꾸게 되면 두 값이 다 바뀜...!
      ```

      - 이렇게 바꾸게 되면 두 값이 다 바뀜. 같은 인스턴스를 바라보기 때문.

      ```java
      Address copyAddress = new Address("city2", address.getStreed(), address.getZipcode());
      ```

      - 이런식으로 넣어야 안전함.

    - 이러한 동작은 실무에서 심각한 오류가 되나 컬렉션 값 타입에서 참조값을 전달하는 것은 컴파일단계에서 막을 수 없음. 

    - 때문에 수정자(setter)를 만들지 않거나 private로 사용하는 등의 `불변객체`를 만들어주어야 부작용이 없음.






## 2) 실무 사용 팁

- 사실 실무에서는 컬렉션 값 타입을 거의 사용하지 않음. (부작용이 훨씬 많기 때문)

- 리스트 등의 컬렉션으로 관리하기 보다는 **엔티티로 한번 Wrapping해서 그 안에서 클래스를 사용하는 것**이 더 바람직함. 이렇게 되면 <u>**식별자를 갖게**</u> 되기 때문에 추적변경 등이 용이해짐.

  ```java
  @Entity
  public class AddressEntity {
  
      @Id @GeneratedValue
      private Long id;
  
      private Address address;
  }
  ```

  ```java
  @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
  @JoinColumn(name = "USER_ID")
  private List<AddressEntity> addressHistory = new ArrayList<>();
  ```

- 지속해서 값을 추적 및 변경해야 한다면, 그것은 값 타입이 아니라 `엔티티`여야 함!!

  

------

> 마지막 수정일시: 2022-09-03 19:36