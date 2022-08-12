---
title: "⑤. Singleton"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true

---

이 포스트 시리즈는 inflearn의 ''스프링 핵심 원리 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------

# 5. Singleton

[ 정리 한 문장: 객체를 무상태로 잘 설계한 후, 스프링에서 관리해주는 싱글톤을 마음껏 누리자! ]

------

싱글톤은 스프링의 강력하고 멋진 기능으로 생각된다.

싱글톤을 이렇게 쉽게 보장해주는 것 만으로도 Spring이라는 프레임워크를 사용할 의향이 생기는 것 같다.





## 1. 웹 애플리케이션

- 웹 애플리케이션은 보통 여러 고객이 동시에 요청을 보냄.
- 수 많은 고객들이 요청을 보낼때, 그 요청을 처리하기 위해 매번 서비스 객체를 생성한다면? 객체가 생성될 때마다 메모리가 소모됨. -> 비효율적임. (물론 컴퓨터 성능이 월등히 좋아져서 불가능하진 않지만, 효율적으로 처리하는 방법으로 개발하는 것이 당연히 필요함)







## 2. Singleton

- 필요한 객체를 딱 1개만 생성하고, 그 객체를 여러 요청을 처리할 때 동시에 사용하는 방법.

### - 기존 자바 프로젝트에서의 싱글톤 패턴 적용방법

```java
public class SingletonService {
    // 1. static 영역에 객체 1개 생성해둠. 그러면 여기 인스턴스에 객체 참조주소를 넣어놓음.
    private static final SingletonService instance = new SingletonService();

    // 2. getInstance메서드를 실행하면 이미 static에 있는 객체 반환.
    public static SingletonService getInstance() {
        return instance;
    }

    // 3. 생성자가 private이기 때문에 외부에서 생성자 사용 불가 -> 외부에서 new가 호출되지 않음.
    private SingletonService() {
    }
}
```

```java
public class SingletonTest {
    // 객체 가져와서 사용할 때 new가 불가능하기 때문에 getInstance()로 가져와서 사용.
	SingletonService singletonService = SingletonService.getInstance();
}
```

![singleton-](..\..\images\singleton-.PNG)

- 핵심은 외부에서 객체를 생성하는 메서드나 변수에 접근하지 못하게 private로 막아놓고, 객체는 get으로만 가져가서 사용할 수 있도록 하는 것.
- 객체는 static이라 메모리에 하나 올려둠.
- 문제는, 
  - 순수 자바로 구현하기에는 코드가 많이 필요하고, 결국 클라이언트(service)에서 객체를 가져갈 때 구현체에 의존하게 되기 때문에 의존성 문제가 생김.
  - private 생성자로 자식 클래스를 만들기 어려움.
  - 유연성이 떨어짐.
- 그러면 스프링에서는,,, 어떻게 관리하지?



### - 스프링 컨테이너에서의 싱글톤 패턴

- 스프링에서는 컨테이너가 알아서 관리해준다....! 기본적으로 싱글톤 패턴으로 객체를 관리해주기 때문에, 빈으로 등록해두기만 하면 싱글톤패턴에 대해서는 신경을 쓰지 않아도 된다. 즉, 이미 싱글톤 패턴으로 객체를 관리하고 있었던 것.
- 물론, 요청할 때마다 새로운 객체를 생성해서 반환하는 기능도 제공하지만, 실무에서 특별한 케이스가 아니고는 사용할 일이 거의 없음.

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

MemberService memberService1 = ac.getBean("memberService", MemberService.class);
MemberService memberService2 = ac.getBean("memberService", MemberService.class);

System.out.println("memberService1 = " + memberService1);
System.out.println("memberService2 = " + memberService2);
// 객체 참조주소를 보면 똑같고, 테스트도 통과한다.
Assertions.assertThat(memberService1).isSameAs(memberService2); 
```

![spring-singleton](..\..\images\spring-singleton.PNG)







## 3. Singleton 방식의 주의점! **

- 싱글톤은 객체를 static에 1개만 올려서 관리하는 것이라고 했다. 그런데, 이 객체에 의존하는 값(필드 변수 등)이 있다면 어떻게 될까? 다른 사용자도 그 변수를 공유해서 사용하게 되는 엄청난 불상사가 발생하게 된다!!!
- 즉, 객체를 ***무상태(stateless)로 설계하고 유지해야 한다.***
  - 클라이언트에 의존하는 필드가 있으면 안됨.
  - 값을 변경할 수 있는 필드가 있으면 안됨.
- 지역변수, 파라미터, ThreadLocal 등이 해결방법!

```java
public class StatefulService {

    private int price; // 상태를 유지하는 필드

    public void order(String name, int price) {
        System.out.println("name: " + name + " / price: " + price);
        this.price = price; // 여기가 문제
    }

    public int getPrice() {
        return price;
    }
    
}
```

- 위 코드에서, 이 객체를 가지고 price값을 불러오게 된다면(`getPrice()`) 싱글톤으로 관리되고 있는 해당 객체의 필드값 price에 대한 값이 튀어나오게 되고, 이는 클라이언트마다 다른 값이어야 하기 때문에 잘못된 설계이다. 아래와 같이 무상태(stateless)로 설계를 바꿔보자.

```java
public class StatefulService {

    public int order(String name, int price) {
        System.out.println("name: " + name + " / price: " + price);
        return price;
    }

}
```

- 객체에 의존하는 필드값이 없고, 해당 메서드를 실행하여 나온 리턴 값을 서비스에서 지역변수로 받아주면 된다.

  `int userAPrice = statefulService.order("userA", 10000);`

  `userAPrice`는 지역변수로, 해당 값에 다른 값을 할당하지 않는 한 값은 유지가 되며, 클라이언트에 의존적이지 않다.







## 4. @Configuration과 Singleton

- Spring 컨테이너는 어떻게 객체를 1개로 관리할까? `AppConfig`를 @Configuration 으로 등록해놓은 상태에서 클래스타입을 확인해보자.

![appconfig.class](..\..\images\appconfig.class.PNG)

- AppConfig Bean의 클래스타입을 확인해보면 `.AppConfig$$EnhancerBySpringCGLIB$$~~` 이라고 되어 있다. 왜 그냥 AppConfig가 아니라 뒤에 뭐가 더 붙은걸까?

### @Configuration

- 해당 어노테이션은 구성(설정) 파일 클래스에 붙여주는 어노테이션이다. 이 어노테이션을 붙여주면, 스프링이 부트되며 Bean으로 등록되는데, 그때 이 어노테이션이 붙어있는 클래스는 **스프링에서 해당 클래스를 상속받은 본인만의 클래스**를 만든다. 이게 `.AppConfig$$EnhancerBySpringCGLIB$$~~` 얘다.
- `.AppConfig$$EnhancerBySpringCGLIB$$~~` 는 '컨테이너에 해당 객체가 있으면 그 객체를 쓰고, 없으면 새로 빈으로 등록해' 라는 내부적인 로직을 가지고 있다. 즉, 얘가 기존 클래스에 추가로 메서드를 오버라이딩하여 싱글톤으로 관리를 해주고 있다는 것!!

**@Configuration이 아니라 @Bean으로 AppConfig.class를 관리하면 어떻게 될까?* <br> 역시 Bean으로 관리되기는 하지만, 구성(설정)정보라고 인지되지 않아 객체의 싱글톤 유지가 안된다. (즉, CGLIB 클래스를 안만들고 해당 클래스를 그대로 빈으로 등록하여 사용한다!)

------

> 마지막 수정일시: 2022-08-12 15:17