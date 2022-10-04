---
title: "[albumProject] 2. 유저 기능 구현"
layout: single
categories: java_Project
typora-root-url: ..\..\images\
toc: true
---



![image-20221004134750381](..\..\images\image-20221004134750381.png)

ERD를 보고 크게 두 부분으로 프로젝트 구성을 나누기로 했다.

1. 유저 관련: 가족 - 아기 - 유저 테이블
2. 글 관련: 사진글 - 댓글 - 사진 및 각 글의 좋아요 테이블

그리고 글이 없어도 테스트와 구현이 가능한 유저단을 먼저 구현하기로 결정했다.

이번 글은, 유저 관련한 기능들을 구현하면서 배운 내용과 생각한 내용들에 대한 정리, 회고 포스팅이다. 구체적인 서비스로직과 작성이유에 대해서 구구절절 설명하긴 어려우니, 가장 고민했던 부분과 구현이유에 대해 작성하고자 한다.

------

# 1. Entity

처음으로 나에게 필요한 엔티티, 테이블에 대해 고민하고 연관관계와 pk - fk 값에 대해 고민해보았다. 정확한 기능구현을 하기 위해 학습도 정말 많이 필요했다.

스터디원들의 피드백도 받아가며 구성한 Entity는 다음과 같다.

![image-20221004135421000](..\..\images\image-20221004135421000.png)

## 1. 고민

- 로그인을 하게 되면, 이후에는 모든 작업이 그 '가족' 안에서만 이루어져야 하는데, 유저, 게시글 등을 한 덩어리로 식별해줄 무언가가 필요하다.

- 테이블간의 연관관계를 설정할 때, FK 값을 누가 들고있어야 하는가.

  

## 2. 구현(해결방안)

- 가족 테이블 사용으로 `가족id` 활용

  처음에는 같은 가족임을 식별할 수 있도록 유저테이블에 '가족uuid' 또는 '가족id'를 들고다니려고 했다. 그러나, 

  `1) 테이블이 무거워지고` 

  `2) 게시글과 다른 테이블에도 활용해아 하는데 활용도가 매우 낮아지기 때문`

  에 테이블을 하나 빼기로 결정했다. 그렇게 하고 나니, 가족 테이블을 가준으로 게시글을 묶을 수 있었고, 게시글도 가족id만 FK값으로 가지고 있으면 언제든 가족을 찾을 수 있는 것 같다. (아마 게시글 파트를 구현하면서 새로운 문제와 인사이트를 발견할 수 있을 것이다.)

  

- 일대다, 다대일 관계에서의 연관관계 주인은 `다`

  유저가 그 가족 아이디의 FK를 들고 있기로 했다. `JPA` 강의를 수강하면서 이론적으로 배웠던 내용이라 적용을 해보았다. 당시에 적용할때는 `일대다`가 아니라 `다대일`로 하는 것을 추천하는지 몰랐으나, 이후에 또 다른 스터디원의 프로젝트 구현을 함께 고민하면서 이유에 대해 간접경험을 해볼 수 있었다.

  

  - 다른 스터디원의 Entity

  ```java
  public class Coupon {
      @Id @GeneratedValue
      private Long id;
  
  }
  ```

  ```java
  public class Member {
      @Id @GeneratedValue
      private Long id;
      @OneToMany
      @JoinColumn(name = "coupon_id")
      private List<Coupon> couponList = new ArrayList<>();
  }
  ```

  객체지향적으로 본다면 맴버가 쿠폰을 가지고 있는게 적당해보인다. 하지만, OneToMany로 하게 되면 

  `1) 컬랙션타입`으로 들고있어야 하고, 

  `2) 쿠폰을 추가할때에도 쿼리가 멤버쪽으로 날아가야` 하는 상황이 발생할 수도 있다.

  

  그래서 스터디원에게 `일대다` 관계 말고 `다대일` 관계로 Coupon이 Member의 FK를 들고 있도록 구조를 변경할 것을 추천했다.

  ```java
  public class Coupon {
      @Id @GeneratedValue
      private Long id;
      @ManyToOne
      @JoinColumn(name = "member_id")
      private Member member;
  }
  ```

  ```java
  public class Member {
      @Id @GeneratedValue
      private Long id;
      
  }
  ```

  이렇게 되면 객체지향 관점에서는 조금 어색할 수 있지만, DB 테이블 관점에서는 더 명확하게 정리가 될 수 있겠다. Member가 가지고 있는 쿠폰을 찾고 싶으면 Coupon 테이블에 가서 Member 아이디와 동일한 쿠폰을 찾으면 될테니까.

  

  당시에 이해를 완벽히 하지 않고 적용했던 부분인데, 이렇게 상황이 발생하니 조금 더 이해가 되었다.



## 3. 개선여지

- `BaseEntity` 사용이 가능할 것 같다. 엔티티 생성, 수정, 삭제 일시는 모든 엔티티에 존재하게 될 가능성이 있는 컬럼이기 때문에, `BaseEntity`로 묶어주는 것이 가능할 것 같다.
- `EntityListeners` 어노테이션으로 위 내용을 조금 더 편하게 구현할 수 있다는 것을 알게 되었다. 추후 공부하여 적용해볼 수 있겠다.
  - 연관 어노테이션: `@EnableJpaAuditing`, `@EnableJdbcAuditing`, `@CreateDate`, `@LastModifiedDate` 등



## 4. 학습내용

- 링크: [관계형 DB 설계방법 학습](https://jiyongyoon.github.io/database/RDB/)
- 링크: [JPA 학습](https://jiyongyoon.github.io/jpa/jpa-basic/)

------

# 2. ServiceImpl

서비스로직 구현에 있어서 가장 큰 벽은  바로 `Spring Security`였다. 서비스로직 뿐만 아니라 나중에 Controller 구현과 Test 코드 작성시에도 계속해서 따라다니는 큰 벽이였다...

구현 자체는 간단한 기능들이라, 스프링 빈에 잘 등록하고, 어노테이션들을 잘 활용했을때 크게 문제는 없었다(고 생각했다. 나중에 컨트롤러와 테스트코드 작성 후 멘토님에게 피드백을 받았을 때 수정할 부분이 많이 보였다.)

## 1. 고민

- 코드를 작성하다 보니 의문이 생겼다. 지금은 간단한 로직들이라 구현이 쉽게 가능하지만, 나중에 다른 `Service`를 의존하게 되는 경우가 생길까? 그렇다면 괜찮은 설계인 것인가? `Service` 클래스 하나가지고 복잡한 비즈니스 로직들을 다 감당할 수 있을 것인가??! (벌써 내 서비스만도 코드가 300줄이 되어 가는데...?)

- 테스트는 어떻게 하지?

  - `SpringSecurity` 테스트는 어떻게 하지????

    

## 2. 구현(해결방안)

- 이 문제를 가지고 여러 블로그 글과 자료를 찾아보았고, `도메인 모델 패턴`, `도메인 주도 설계`, `클린 아키텍처`, `ArchUnit` 등의 키워드들을 발견했다. 그리고 내 코드를 보니 `MailComponent`라는 클래스를 `Domain`처럼 사용하고 있었다. 하지만, 이러한 명확한 개념 없이 코드를 작성하다보니 추후 멘토님의 피드백으로 수정된 부분이 있었다.

  ```
  [멘토님의 피드백]
  sendInviteMail 의 경우도 성공여부를 t/f로 반환하는 것이 아닌 성공이면 정상적으로 메소드 종료, 에러인경우는 에러를 던짐 으로 처리하는 것이 좋을 것 같아요. 그리고 exception을 던지도록 변경한다면 이 메소드를 사용하는 곳에서 이를 처리하는 것이 더 좋을 것 같습니다. mail component는 메일을 보내기위한 역할을 하는 컴포넌트로 두는것이 좋을 것 같아요. 
  ```

  피드백 내용에 대해 조금 풀이하자면, 내 코드는 `Component` 클래스에서 `exception`을 던지고 있었다. 그러나 내 고민의 연장선상에서 생각해보면 `Component`는 그 역할이 메일을 보내기 위한 코드이기 때문에 `exception`처리는 서비스단에서 하는 것이 적합하다는 내용이다. 컴포넌트의 존재이유를 생각해보면 무척 당연한 이야기다.

  예외를 언제 어떻게 발생시키고 어떻게 처리할지는 내 서비스에 대한 문제이기 때문에 서비스단으로 빼서 수정하는 것이 필요했다.

- `Service` 단에서도 Layer를 구분하여 참조하는 것, 그리고 기존 `MVC패턴`에서의 `Service` 클래스는 DB로의 연결다리정도만 해주고 실제 비즈니스 로직은 `Domain`에서 구현하는 방법 등의 여러가지 내용들이 있었다. <u>**추가 학습 필요. 별표.**</u>

- 처음 구현한 `UserService`는 `SpringBoot`를 활용한 통합테스트로, 이후 구현한 `BabyService`는 `JUnit`을 활용한 `Mock`테스트로 진행하며 테스트 방법에 대해서도 고민하게 되었다.

  

## 3. 개선여지

- 추가로 `Exception` 을 어디서 어떻게 처리할 것인지에 대한 서비스 전반적인 고민이 필요하다. 백엔드 단에서 처리를 할 것인가(더미 데이터를 넣거나 혹은 그에 적합한 데이터 등을 리턴할 것인가), 혹은 그대로 클라이언트 단으로 넘길 것인가, 등등의 고민이 필요하다.

- 그리고 이를 위해 `controllerAdvice`와 `ExceptionHandler`의 역할이 있음을 알게 되었다. 학습을 하였지만 아직 적용을 해보지 못했다.

  

## 4. 학습내용

- 링크: [도메인 모델 패턴 관련글](https://velog.io/@yu-jin-song/Spring-Boot-%EA%B2%8C%EC%8B%9C%ED%8C%90-%EA%B5%AC%ED%98%84-2-%EA%B2%8C%EC%8B%9C%EB%AC%BC-%EB%93%B1%EB%A1%9D-%EC%88%98%EC%A0%95-API-%EA%B5%AC%ED%98%84)
- 링크: [JUnit 테스트 학습](https://jiyongyoon.github.io/spring/junit-test/)
- 링크: [ControllerAdvice 예외처리 학습](https://jiyongyoon.github.io/spring/controller-advice/)

------

# 3. Controller

컨트롤러는 어렵지 않겠지- 생각했지만 컨트롤러를 구현하면서 API의 가장 앞단까지 오다보니, 전체적인 수정이 생기게 되었다..

## 1. 고민

- 컨트롤러 메서드 내에서는 어느정도까지 기능을 담아야 하는 것일까. 단순히 `Service` 메서드를 호출하고 결과를 반환하는 역할? 아니면 `Repository`에서 `Entity`나 기타 정보를 찾아와도 되는 것일까?

- 아니 `SpringSecurity` 테스트는 어떻게 해?????????? `Principal` 객체는???

  

## 2. 구현(해결방안)

- 일단 컨트롤러는 `Service`메서드를 호출하고 결과를 반환하는, 즉 교통정리의 역할을 하는 것으로 결정했다. `Service`메서드에서 userId를 찾는 부분이 모두 추가되었다. 그렇게 되니 `Service`에서의 파라미터가 `Principal` 객체에서의 유저정보를 가져와야 했다.
- `Principal`을 사용하니, 테스트를 하기가 엄청 껄끄러워졌다. 어디서 가져와야하지? 결국 `@WithMockUser`와 `token`을 직접 테스트 메서드에서 생성해서 넣어주는 방법들을 알게 되었고, 테스트를 진행할 수 있었다.

```java
// 테스트코드 예시
User user = new User(
    userEntity1.getPhone(), 
    userEntity1.getPassword(), 
    AuthorityUtils.createAuthorityList("ROLE_USER", "ROLE_REPRESENTATIVE")
);
TestingAuthenticationToken testingAuthenticationToken = new TestingAuthenticationToken(user,null);

mockMvc.perform(
    post("/baby/add")
        .content(babyInputJson)
        .principal(testingAuthenticationToken)
        .contentType(MediaType.APPLICATION_JSON))
    .andExpect(status().isOk())
    .andDo(print());
```



## 3. 개선여지

- `Restful`API란 이야기를 들어봤으나, 뭔가 추상적인 내용이었다. 이번 컨트롤러를 구현하면서 체감되는 부분이 있었는데 바로 `Post`와 `Put`, `Delete`의 세분화였다.

- 기존에는 `Post`로 모든것을 처리했지만, `Put`과 `Delete`가 나왔고, 이를 통해 같은 url를 다른 메서드로 사용할 수 있었으며, Mapping만으로 기능을 조금 더 세분화하고 가독성을 높일 수 있다는 것을 알게 되었다.

- 다만, 실무에서는 아직 `Post`만으로 진행하고 있는 곳도 있다는 자료도 함께 알게 되었다. (운영하는 서버나 도메인 등에 따라서 지원을 안하는 경우가 있다고 한다.)

  

## 4. 학습내용

- 링크: [`@RequestMapping`에 대한 학습](https://jiyongyoon.github.io/spring/RequestMapping/)
- 링크: [`ResponseEntity`에 대한 학습](https://jiyongyoon.github.io/spring/springboot-responseentity/)

------

# 4. 느낀점

구현기간보다 학습기간이 훨씬 길었다. [제로베이스 5개월차 후기](https://jiyongyoon.github.io/diary/school-fourth/)에서도 작성했지만, 아는게 아는게 아니었다. 한 주에 하루나 이틀정도만 기능을 구현할 수 있었고, 이를 구현하기 위한 기본적인 공부와 이해가 훨씬 많이 필요했다.



## 1. 정확하게 이해해야 한다.

정확하게 모르니까 어떤 곳에 어떻게 써야할 지 모르겠더라. 여기서 말하는 '정확하게'란, '정답'이면 가장 좋겠지만, 그렇지 않더라도 `올바른 내용을 내가 내 언어로 풀어낼 수준` 이라고 정의하고 싶다. 내가 왜 이 기능을 사용했고, 어떻게 동작하는지에 대해서 설명할 수 있어야 해당 기능을 사용할 자신이 생겼었다. 내 프로젝트이다보니, 애정이 생겨서 그런지 더 정확하게 학습하고자 하는 의욕이 많이 생겼다.



## 2. 학습하는 습관.

1번의 연장선이다. 부트캠프를 다니면서 여러가지를 배우지만, 결국 실제 기능을 구현하기 위해서는 내가 이해하고 습득한 기술이 적용되기 때문에, 내가 직접 학습하는 것이 중요함을 알게 되었다. 그리고 Google뿐만 아니라 공식 Docs를 찾아보기 시작하게 되었고, 이를 활용하여 정확하고 필요한 지식을 얻는 방법, 그리고 무엇보다 `얻을 수 있다는 자신감`을 얻게 되었다.



## 3. 에러따위.

골치아프긴 하다. 그러나 컴파일단에서 보여주는 에러가 얼마나 감사하고 소중한지, 그리고 로그기록들이 얼마나 친절한지 많이 경험했다. 에러코드 보는 방법도 점점 늘고있다... (좋은거 맞지)

------



우수수강생에 선정되어 이번달부터 프론트-백 협업 프로젝트가 진행되게 된다. 핑계이지만, 때문에 개인프로젝트를 잠시 미뤄두어야 할 것 같다. 협업 프로젝트가 끝나고 나면 내 소중한 개인프로젝트를 꼭 마무리 하러 올 것이다.

잠시만 다녀오겠습니다. 그리고 엄청나게 성장해서 돌아오겠습니다.



------

> 마지막 수정일시: 2022-10-05 12:10
