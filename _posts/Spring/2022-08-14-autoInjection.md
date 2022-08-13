---
title: "⑦. 의존관계 주입방법과 자동주입"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true

---

이 포스트 시리즈는 inflearn의 ''스프링 핵심 원리 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------



# 7. 의존관계 주입방법과 자동주입

[ 정리 한 문장: 편리한 Autowired는 업무로직 빈에서, 생성자 객체주입은 Config등의 기술 지원 빈에서 사용하자. ]

------

스프링에서 의존관계를 주입하는 여러가지 방법과, 어떤 상황에서 어떤 방법을 사용하는 것이 좋을지 알아보자.





## 1. 의존관계 주입 방법

### 1) 생성자 주입(constructor)

- 이때까지 해왔던 거의 대부분의 방법.
- 생성자를 통해 의존관계를 주입하다 보니, 생성자의 특성을 그대로 가져감. (컴포넌트 스캔 시 생성자 호출을 딱 1번만 하게 되고, 이후에는 객체를 생성하지 못하기 때문에 거의 변경되지 않는다)

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
    
    @Autowired // 생성자가 딱 하나면 생략도 가능함.
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```



### 2) 수정자 주입(setter)

- Java의 컨벤션인 setter메서드를 사용하여 의존관계를 주입함.
- setter 메서드의 특성인 수정이 가능함.

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
    
    @Autowired 
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
    
    @Autowired 
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```



### 3) 필드 주입

- private 필드에 바로 주입하는 방법.
- 코드는 간결해서 좋으나, 외부에서 변경이 전혀 불가능해 테스트가 어려움.
- 웬만하면 쓰지 말자.

```java
@Component
public class OrderServiceImpl implements OrderService {
    @Autowired private final MemberRepository memberRepository;
    @Autowired private final DiscountPolicy discountPolicy;
}
```



### 4) 일반 메서드 주입

- 일반 메서드에도 `@Autowired`를 사용하여 의존관계를 주입할 수 있음.
- 생성자처럼 여러 필드를 한번에 주입 받을 수 있으나, 역시 굳이 사용하지 않는 방법.

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
    
    @Autowired
    public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```



### 5) 그래서 뭐 쓰나?

- 기본적으로 생성자 주입 권장.
- 실무에서 대부분은 의존관계 주입이 한번 일어나면 애플리케이션 종료 시점까지 변경할 일이 거의 없음. 오히려 변경되지 않도록 관리하는게 더 중요한 경우가 많음.
- 참고로, 생성자 주입을 제외한 나머지는 생성자가 호출된 후 호출되므로 필드에 `final` 사용이 불가하다. 생성자 주입만 필드에 `final`사용 가능. *-> 나중에 `lombok` 라이브러리와 아주 찰떡 궁합이다.*

```java
// Lombok 활용한 깔끔코드
@Component
@RequiredArgsConstructor // final 필드를 파라미터로 갖는 생성자 자동생성
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
}
```

### **Lombok*

: 어노테이션을 활용해 개발에 많이 사용되는 여러가지 메서드(Getter, Setter, Constructor 등)들을 편하게 생성해주는 라이브러리. 나중에 더 자세하게 공부해보자.





## 2. NoUniqueBeanDefinitionException

- 빈이 2개 이상 조회 될 때 생기는 오류.
- `@Autowired`로 객체를 주입할 때, 스프링 컨테이너에서는 동일한 '타입(클래스)'을 찾아봄.

```java
@Autowired
private DiscountPolicy discountPolicy;
```

-> 이 경우 컨테이너에서 `ac.getBean(DiscountPolicy.class);`처럼 스프링 빈을 조회.

-> 빈이 없으면 객체를 주입하지만, 있으면 `NoUniqueBeanDefinitionException` 오류가 발생함.

```java
@Component
public class FixDiscountPolicy implements DiscountPolicy {}

@Component
public class RateDiscountPolicy implements DiscountPolicy {}
```

 이 후 @Autowired를 통해 객체를 주입하려고 보니까?

```java
@Autowired
private DiscountPolicy discountPolicy;
```

스프링 컨테이너에(){빈네임 : 빈객체}) `{discountPolicy : fixDiscountPolicy, discountPolicy : rateDiscountPolicy}` 이렇게 빈이 두 쌍이 있네? 

-> *스프링: 빈이 1개 있어야 의존성 주입을 하는데, 빈이 2개입니다. 어떤 객체를 주입할까요??*



### @Autowired 필드 명 매칭 방법

```java
@Autowired
private DiscountPolicy fixDiscountPolicy;
```

- 필드 명인 fixDiscountPolicy 빈을 추가로 조회 후 빈 객체를 찾아서 주입.

### @Qualifier("이름")

- 쉽게 말해 객체에 tag를 달아 놓는다고 생각하면 됨.

```java
@Component
public class FixDiscountPolicy implements DiscountPolicy {}

@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {}
```

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
    
    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, 
        @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
        	this.memberRepository = memberRepository;
        	this.discountPolicy = discountPolicy;
    }
}
```



### @Primary

- 여러 빈이 매칭될 때 이 어노테이션 빈이 우선권을 가지도록 함.

```java
@Component
@Primary
public class FixDiscountPolicy implements DiscountPolicy {}

@Component
public class RateDiscountPolicy implements DiscountPolicy {}
```

** `@Qualifier` vs  `@Primary`* - @Qualifier는 디테일하게 이름을 정해주다보니, 스프링에서는 이 어노테이션이 우선권을 가지는 것으로 처리함.



### @나만의어노테이션

- annotation 클래스를 만들어서 사용 가능. (@Qualifier와 비슷하게 작동)

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {
}
```

- 이렇게 어노테이션 클래스를 만들면, `"mainDiscountPolicy"` 이 어노테이션을 사용할 수 있게 됨. (마치 `@Component` 처럼)







## 3. 조회한 빈이 모두 필요한 경우

- Map에 { 빈이름 : 빈객체 } 형대로 담아두었다가 로직을 통해 동적으로 객체를 꺼내서 주입할 수 있음.

```java
static class DiscountService {
    private final Map<String, DiscountPolicy> policyMap;

    @Autowired
    public DiscountService(Map<String, DiscountPolicy> policyMap) {
        this.policyMap = policyMap;
    }

    public int discount(Member member, int price, String discountCode) {
        // discountCode를 파라미터로 받아서, 해당하는 빈객체를 Map에서 get.
        DiscountPolicy discountPolicy = policyMap.get(discountCode);
        // 해당 객체를 주입받아 할인금액 계산.
        return discountPolicy.discount(member, price);
    }
}

@Test
void findAllBea() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);

    DiscountService discountService = ac.getBean(DiscountService.class);
    Member member = new Member(1L, "userA", Grade.VIP);

    int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");
	// 여기는 fix 할인정책 적용됨
    assertThat(discountService).isInstanceOf(DiscountService.class);
    assertThat(discountPrice).isEqualTo(1000);
	// 여기는 rate 할인정책 적용됨
    int rateDiscountPrice = discountService.discount(member, 20000, "rateDiscountPolicy");
    assertThat(rateDiscountPrice).isEqualTo(2000);
}
```







## 4. 자동 빈 등록. 무조건 사용인가?

- 업무로직(Service, Controller, Repository 등)에 필요한 객체들은 객체가 워낙 많고 유사한 패턴이 있기 때문에 자동으로 빈을 등록하고 객체를 주입받는 것이 매우 편함.
- 그러나, 기술 지원 빈(AOP, Config 등)은 애플리케이션 전반에 걸쳐 영향을 주며, 한 눈에 파악하는 것이 중요하기 때문에 수동으로 빈을 등록해서 사용하는 것이 유지보수 등에 이점이 많음!!



------

> 마지막 수정일시: 2022-08-14 03:49