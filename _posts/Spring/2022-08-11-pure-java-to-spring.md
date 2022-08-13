---
title: "③. Java에서 Spring으로"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true
---

이 포스트 시리즈는 inflearn의 ''스프링 핵심 원리 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------

# 3. Java에서 Spring으로

[ 정리 한 문장: 객체를 외부(AppConfig -> Spring Container)로 부터 주입받아서 의존성과 다형성을 확보시킨다! ]

------

Java에서 Spring으로 넘어가는 단계. 순서는 Pure-java로 만들어보고, 이를 Spring으로 넘겨보자.





## 1. 할인 정책에 따른 주문 클래스 만들어보기

: 1장에서 배웠던 다형성과 DIP(의존관계 역전 원칙)에 의거해 인터페이스와 구현체를 분리해서 만들어보기.

- **주문 인터페이스: OrderService**

  ```java
  public interface OrderService {
      Order createOrder(Long memberId, String itemName, int itemPrice);
  }
  ```

- **주문 클래스: OrderServiceImpl**

  ```java
  public class OrderServiceImpl implements OrderService {
  
      private final MemberRepository memberRepository = new MemoryMemberRepository();
      private final DiscountPolicy discountPolicy = new FixDiscountPolicy(); // 고정 할인 정책
      
      @Override
      public Order createOrder(Long memberId, String itemName, int itemPrice) {
          Member member = memberRepository.findById(memberId); // MemoryMemberRepository 객체 사용
          int discountPrice = discountPolicy.discount(member, itemPrice); //FixDiscountPolicy 객체 사용
  
          return new Order(memberId, itemName, itemPrice, discountPrice);
      }
  }
  ```

- **실행 클래스: OrderApp**

  ```java
  public static void main(String[] args) {
  	
      OrderService orderService = new OrderServiceImpl();
  
      Long memberId = 1L;
      
      Order order = orderService.createOrder(memberId, "itemA", 15000);
  
      System.out.println(order);
      System.out.println(order.calculatePrice());
  }
  ```

  1. **OrderServiceImpl(클래스)**은 MemberRepository(인터페이스)와 DiscountPolicy(인터페이스)라는 역할을 의존하는 것으로 보인다. 하지만 실제로는 클래스 안에서 각 인터페이스(역할)에 어떤 객체를 넣어서 사용할 것인지를 **직접 선택하고 객체를 생성**하고 있다. *-> DIP 위반*

  2. 할인정책에 바뀌면 **OrderServiceImpl(클래스)**에서 객체를 바꿔주어야 한다. *-> OCP 위반*

     ```java
     // private final DiscountPolicy discountPolicy = new FixDiscountPolicy(); // 고정 할인 정책
     private final DiscountPolicy discountPolicy = new RateDiscountPolicy(); // 정률 할인 정책
     ```

  ![orderService](..\..\images\orderService.png)

  

  

  

  ## 2. 관심사와 역할 나누기

  - Service 클래스는 Service로직을 실행하는 것에만 역할이 있고 어떤 객체를 생성할지는 관심사가 아니며 몰라야 한다. 즉, 객체에 의존적이지 않고 역할(인터페이스)에만 의존해야 한다.

    => 객체를 결정하고 생성하는 누군가가 필요하다!! => AppConfig

    

  - **어플리케이션 설정 클래스: AppConfig**

    ```java
    public class AppConfig {
        // 여기서 주입시키는 객체가 -> OrderService의 생성자로 들어가서 객체로 주입됨.
        public OrderService orderService() {                              // 할인정책 객체 생성
            return new OrderServiceImpl(new MemoryMemberRepository(), new RateDiscountPolicy());
        }
    }
    ```

    ```java
    public class OrderServiceImpl implements OrderService {
    
        private final MemberRepository memberRepository; // 변수만 선언하고
        private final DiscountPolicy discountPolicy; 
        // 객체는 생성자를 통해 주입받는다.
        public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
            this.memberRepository = memberRepository;
            this.discountPolicy = discountPolicy;
        }
    
        @Override
        public Order createOrder(Long memberId, String itemName, int itemPrice) {
            Member member = memberRepository.findById(memberId);
            int discountPrice = discountPolicy.discount(member, itemPrice);
    
            return new Order(memberId, itemName, itemPrice, discountPrice);
        }
    }
    ```

    ```java
    public static void main(String[] args) {
        AppConfig appConfig = new AppConfig(); // AppConfig 객체를 생성해서 객체 불러오기
        OrderService orderService = appConfig.orderService();
    
        Long memberId = 1L;
        
        Order order = orderService.createOrder(memberId, "itemA", 15000);
    
        System.out.println(order);
        System.out.println(order.calculatePrice());
    }
    ```

    

    1. 정책에 따라 리턴할 객체를 바꿔주어 OrderService의 클래스 생성자로 해당 객체를 넘기면 된다.<br>즉, 이 어플리케이션에서 구현할 **객체 생성은 이제 AppConfig가 맡아서** 하는 것! Service클래스는 이제 서비스하는 것만 집중하면 된다. 역할분담.

    2. AppConfig는 설정파일이다 보니, 해당 어플리케이션의 구조를 한눈에 볼 수 있으면 좋다. 비즈니스 전체에 대한 역할들이 보여야 한다. 따라서, **모든 역할(인터페이스)이 보이도록 구현**해주자.

       ```java
       public class AppConfig {
           // MemberRepository 역할    
           public static MemberRepository getMemberRepository() {
               return new MemoryMemberRepository();
           }
           // DiscountPolicy 역할   
           public static DiscountPolicy discountPolicy() {
               return new FixDiscountPolicy(); // 고정 할인 정책
               return new RateDiscountPolicy(); // 정률 할인 정책
           }
       
           // 여기서 주입시키는 객체가 -> OrderService의 생성자로 들어가서 객체로 주입됨.
           public OrderService orderService() {
               return new OrderServiceImpl(getMemberRepository(), discountPolicy());
           }
       }
       ```

       => 이제는 **사용영역의 코드**(Service, Repository 등)의 코드는 정책이나 생성해야하는 객체가 바뀌어도 **변경할 필요가 없다!** AppConfig가 있는 **구성영역의 코드만 수정**하면 된다. 객체 생성은 정적인 의존관계가 아니라 실행 시점(런타임)시 생성되는 것이기 때문에 동적으로 변경이 가능하다.

  ![orderService&AppConfig](..\..\images\orderService&AppConfig.png)

  

  

  

  ## 3. Spring으로 넘어가기

  - @어노테이션을 활용하여 Spring Container를 사용해보자. (간단하게만)
  - **@ApplicationContext**: 스프링 컨테이너. 이 컨테이너에 객체를 넣어서 관리하고, Service에서 컨테이너에 있는 객체를 꺼내어 사용함.
  - **@Configuration**: 스프링 컨테이너가 이 어노테이션이 붙은 클래스를 설정(구성)정보로 사용함.
  - **@Bean**: 컨테이너에 등록된 객체. 객체의 이름은 기본적으로 메서드 이름으로 등록됨.

  ```java
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  @Configuration
  public class AppConfig {
     
      @Bean
      public static MemberRepository getMemberRepository() {
          return new MemoryMemberRepository();
      }
      
      @Bean
      public static DiscountPolicy discountPolicy() {
          return new FixDiscountPolicy(); // 고정 할인 정책
          return new RateDiscountPolicy(); // 정률 할인 정책
      }
  
      @Bean
      public MemberService memberService() {
          return new MemberServiceImpl(getMemberRepository());
      }
      
      @Bean
      public OrderService orderService() {
          return new OrderServiceImpl(getMemberRepository(), discountPolicy());
      }
  }
  ```
  
  ```java
  public static void main(String[] args) {
  //    AppConfig appConfig = new AppConfig();
  //    OrderService orderService = appConfig.orderService();
  
      ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
      MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
      OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
  
      Long memberId = 1L;
  
      Order order = orderService.createOrder(memberId, "itemA", 15000);
  
      System.out.println(order);
      System.out.println(order.calculatePrice());
  }
  ```
  
  

------

> 마지막 수정일시: 2022-08-11 01:11