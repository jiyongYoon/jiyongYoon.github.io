---
title: "⑥. ComponentScan과 Autowired"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true

---

이 포스트 시리즈는 inflearn의 ''스프링 핵심 원리 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------



# 6. ComponentScan과 Autowired

[ 정리 한 문장: 스프링 컨테이너에 등록할 클래스는 @Component시리즈, 주입할 객체는 @Autowired ! ]

------

메서드는 @Bean으로 스프링 컨테이너에 등록했다.

그러나, 모든 메서드를 다 @Bean으로 등록해주어야 사용할 수 있을까?





## 1. ComponentScan

- ComponentScan은 구성(설정) 정보 클래스에 사용되는 어노테이션.
- 구성(설정) 정보 클래스 내부는 아무것도 없음. Component를 훑으며 스프링 빈으로 등록해줌.

```java
@Configuration
@ComponentScan (
    excludeFilters = @ComponentScan.Filter (
        type = FilterType.ANNOTATION, classes = Configuration.class
    )
)
public class AutoAppConfig {
	// 아무 내용도 없는게 맞음.
}
```

**@ComponentScan의 추가 설정내용<br>     - excludeFilters는 스캔 대상에서 제외하는 내용. 타입과 클래스를 기준으로 제외시킬 수 있다.





## 2. Autowired

- @Component 클래스에 사용되며, 의존관계를 자동으로 주입해주기 위해 사용하는 어노테이션.

```java
@Service
public class MemberServiceImpl implements MemberService{
    private final MemberRepository memberRepository;

    @Autowired // 생성자에 붙여주면, ComponentScan 할 때 여기 와서 자동으로 객체 주입하게 됨.
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```





### - @ComponentScan 동작 순서

1. AppConfig 클래스에 @ComponentScan을 붙여준다.

2. 해당 Config클래스로 스프링컨테이너를 만든다. <br>`ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);`

3. `AppConfig`가 위치한 패키지부터 그 하위 패키지까지 @Component를 찾는다.

   **< @Component에 포함되는 여러가지 어노테이션 >* <br>

   - `@Component` - 일반 컴포넌트.
   - `@Controller` - MVC 패턴의 컨트롤러에 사용되는 어노테이션. 스프링 컨테이너에서 컨트롤러로 인식.
   - `@Service` - 비즈니스 주요 로직에 사용되는 어노테이션. 특별한 처리는 없음. (컴포넌트와 구별 정도)
   - `@Repository` - 데이터 접근계층에 사용되는 어노테이션. 데이터 계층의 예외를 스프링 예외로 변환. (후에 추가학습)
   - `@Configuration` - 스프링 구성(설정)정보에 사용되는 어노테이션. 이 스프링 빈들은 컨테이너에서 싱글톤을 유지하도록 관리함.

4. @Component 클래스를 빈으로 등록한다. (빈 이름은 클래스명이며, 첫 글자가 대문자인 경우 소문자로 저장)

5. 클래스 내부에 @Autowired 된 객체도 호출하여 의존관계를 주입한다.





## 3. ComponentScan 탐색 위치

- `@ComponentScan(basePackages = "패키지명")`으로 최우선 위치를 잡아줄 수 있음. 해당 패키지부터 해당 패키지의 하위 패키지 모두 스캔
- Default 값은 `@ComponentScan`이 있는 패키지 부터 그 하위 패키지 모두를 스캔하는 것. -> 요즘 권장사항, 추세.
- 때문에, Config클래스는 프로젝트의 최 상단 패키지에 두고 Scan을 하도록 유도한다.





## 4. Filter

- Scan 할 때 조건이 되는 것.
- `includeFilters` : Scan에 추가할 대상
- `excludeFilters` : Scan에 제외할 대상

```java
@Configuration
@ComponentScan(
    includeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
)
static class ComponentFilterAppConfig {
}
// 현재 위 클래스는 구성(설정) 정보 클래스이며, Scan에 추가할 대상, 제외할 대상 Filter가 걸려있다.
```

```java
// MyExcludeComponent.java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyExcludeComponent {
}
// MyIncludeComponent.java도 마찬가지
```

```java
// BeanB.java
@MyExcludeComponent
public class BeanB {
}
// BeanA.java도 마찬가지
```

```java
// test 클래스
@Test
void filterScan() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);
    BeanA beanA = ac.getBean("beanA", BeanA.class);
    Assertions.assertThat(beanA).isNotNull();

    assertThrows(NoSuchBeanDefinitionException.class,
                 () -> ac.getBean("beanB", BeanB.class));
}
```

- `ComponentFilterAppConfig.class`에서 ComponentScan을 시작.
- includeFilters 조건에 해당하는 컴포넌트만 스프링 빈으로 추가.
- 따라서 `beanA`는 빈으로 가져와지지만 `beanB`는 스캔대상에서 제외되었기 때문에 `NosuchBeanDefinitionException`오류 발생.





## 5. Bean의 중복 등록

- 가끔 발생.

  1. @Bean 혹은 @Component -> @Autowired로 자동으로 중복해서 등록될 경우

     : `ConflictingBeanDefinitionException` 예외 발생

  2. 자동으로 빈이 등록되어있는 상태에서 같은 이름으로 수동으로 직접 빈을 등록할 경우

     : 기존에는 수동으로 등록한 빈이 오버라이딩 되었으나, 최근 스프링부트에서는 오류를 발생하도록 기본 값을 바꿈. <br>`CoreApplication`을 실행하면 <br>*'Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true'* 스프링부트 에러를 볼 수 있음.



------

> 마지막 수정일시: 2022-08-13 02:17