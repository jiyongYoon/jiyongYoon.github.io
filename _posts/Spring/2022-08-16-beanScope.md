---
title: "⑨. Bean Scope"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true

---

이 포스트 시리즈는 inflearn의 ''스프링 핵심 원리 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

+이 포스트는 이번 포스트 시리즈의 마지막 포스트입니다.

------

# 9. 빈 스코프

[ 정리 한 문장: 싱글톤 빈이 워낙 강력하니 잘 사용하고, 필요에 따라 web 스코프를 사용하자. ]

------

이때까지 스프링 컨테이너를 만들고, 빈을 등록하고, 의존관계를 주입하는 방법에 대해 알아보았다.

그럼, 이 빈은 애플리케이션이 서버에 올라가서 실행될때부터 서버에서 내려올때까지 계속 유지되는 것일까? 필요에 따라 생성하고 소멸시키는 빈은 없을까?

빈의 스코프, 즉 빈이 존재할 수 있는 범위에 대해 알아보자.





## 1. 빈 스코프의 종류

- 싱글톤: 스프링에서 사용되는 기본 스코프. 컨테이너 시작과 종료까지 유지됨.

- 프로토타입: 일회성(?) 빈. 프로토타입 빈이 생성되면 컨테이너는 생성과 의존관계 주입까지만 하고 관리를 하지 않음.

- 웹 관련 스코프: 웹 서비스에 적합한 빈 스코프. (request / session / application)

  

  ### 스코프 등록방법

  1. 컴포넌트 스캔 자동 등록

  ```java
  @Scope("prototype")
  @Component
  public class HelloBean {}
  ```

  2. 수동 등록

  ```java
  @Scope("prototype")
  @Bean
  PrototypeBean HelloBean() {
  	return new HelloBean();
  }
  ```

  

  



## 2. 프로토타입 스코프

- 싱글톤은 컨테이너에서 하나의 빈을 유지하도록 관리한다면, 프로토타입은 조회요청이 올 때마다 새로운 빈을 생성하여 반환.
- 생성된 프로토타입 빈을 반환한 이후에는 컨테이너는 그 빈을 더이상 관리하지 않음. -> `@PreDestroy` 같은 어노테이션이 적용되지 않음.
- 리턴 받은 클라이언트가 빈을 관리할 책임을 가지게 됨.

```java
public class PrototypeTest {

    @Component
    @Scope("prototype")
    static class PrototypeBean {

        @PostConstruct
        public void init() {
            System.out.println("prototypeBean.init");
        }

        @PreDestroy
        public void destory() {
            System.out.println("prototypeBean.destory");
        }
    }
    
    @Test
    void prototypeBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
        System.out.println("find prototypeBean1");
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        System.out.println("find prototypeBean2");
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);

        System.out.println("prototypeBean1 = " + prototypeBean1);
        System.out.println("prototypeBean2 = " + prototypeBean2);
        Assertions.assertThat(prototypeBean1).isNotSameAs(prototypeBean2); // 인스턴스 다름

        ac.close(); // 컨테이너 종료
    }
}
```

```
[console]
find prototypeBean1
prototypeBean.init            -> 빈1이 생성될 때 초기화 콜백
find prototypeBean2
prototypeBean.init            -> 빈2가 생성될 때 초기화 콜백
prototypeBean1 = com.example.spring_basic.scope.PrototypeTest$PrototypeBean@8ad6665  -> 참조값 다름 = 다른 인스턴스
prototypeBean2 = com.example.spring_basic.scope.PrototypeTest$PrototypeBean@30af5b6b -> 참조값 다름 = 다른 인스턴스
00:50:52.215 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@942a29c, started on Tue Aug 16 00:50:52 KST 2022 
-> 스프링 컨테이너 종료
```

- 스프링 컨테이너가 종료되었으나, 소멸 전 콜백이 호출되지 않음. 왜냐? <u>빈을 만들어서 반환한 후에는 컨테이너가 더이상 관리를 하지 않기 때문.</u>



###  - 프로토타입 빈을 싱글톤 빈과 같이 사용하게 될 때 발생하는 문제!!

- 프로토타입 빈은 인스턴스를 요청할 때마다 새로운 객체 인스턴스를 반환해주길 기대함.
- 그러나, `싱글톤 빈`에 `프로토타입 빈`을 주입하면, 컨테이너는 프로토타입 빈을 `프로토타입 빈`으로 생각하지 않고, 그저 `싱글톤 빈`에 주입된 객체 인스턴스로 보기 때문에 계속 같은 빈을 리턴하게 됨!!
- 이렇게 되면, 프로토타입 빈의 필드값이 공유되는 문제가 발생함.[(싱글톤 빈에서 가지는 문제와 동일!)](https://jiyongyoon.github.io/spring/singleton/#3-singleton-%EB%B0%A9%EC%8B%9D%EC%9D%98-%EC%A3%BC%EC%9D%98%EC%A0%90-)

### -  해결방법

1. ObjectProvider
   - ObjectFactory(과거 사용)하던 인터페이스 상속한 인터페이스.
   - `.getObject()`를 사용하면 스프링 컨테이너에서 해당 빈을 찾아서 반환.
   - 장점: 간단하다
   - 단점: 스프링에 의존적이다.

```java
@Autowired
private ObjectProvider<PrototypeBean> prototypeBeanProvider;

public int logic() {
    PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
    prototypeBean.addCount();
    int count = prototypeBean.getCount();
    return count;
}
```

```java
@Test
void singletonClientUsePrototype() {                               // 싱글톤 빈       // 프로토타입 빈
    ApplicationContext ac = new AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);

    ClientBean clientBean1 = ac.getBean(ClientBean.class);
    int count1 = clientBean1.logic();
    assertThat(count1).isEqualTo(1); // 각각 인스턴스 반환

    ClientBean clientBean2 = ac.getBean(ClientBean.class);
    int count2 = clientBean2.logic();
    assertThat(count2).isEqualTo(1); // 각각 인스턴스 반환

}
```

2. `javax.inject.Provider` 라이브러리
   - gradle에 추가하면 사용 가능.
   - `.get()` 메서드를 사용하면 스프링 컨테이너에서 해당 빈을 찾아서 반환.
   - 장점: 스프링에 의존하지 않음.
   - 단점: 라이브러리 끌어와야 함.

```java
dependencies {
    implementation 'javax.inject:javax.inject:1'
}
```

```java
@Autowired
private Provider<PrototypeBean> prototypeBeanProvider;

public int logic() {
    PrototypeBean prototypeBean = prototypeBeanProvider.get();
    prototypeBean.addCount();
    int count = prototypeBean.getCount();
    return count;
}
```



** 뭐 쓰지?* - 빈 조회를 위한 편의기능이 많이 있는 `ObjectProvider `추천. 단, 다른 컨테이너도 사용해야 한다면 java 라이브러리를 사용.

## 







## 3. 웹 스코프

- 웹 환경에서만 동작하는 스코프.
- 프로토타입과는 다르게 스프링이 해당 스코프 종료시점까지 관리. -> 종료 메서드 호출됨.
- request, session, application, websocket 등이 있으며, 여기서는 request만 학습.



### - request

- HTTP 요청이 들어와서 ~ 나갈 때 까지 유지.
- 각 요청마다 객체 인스턴스가 생성 및 관리됨.

```java
[라이브러리 추가]
implementation 'org.springframework.boot:spring-boot-starter-web'
```

- 웹 라이브러리가 추가되면, `AnnotationConfigApplicationContext`가 아닌 `AnnotationConfigServletWebServerApplicationContext`기반으로 애플리케이션 구동.

```java
@Scope(value = "request") // request 스코프 지정
public class MyLogger {

    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "]" + "[" + requestURL + "] " + message);
    }

    @PostConstruct
    public void init() {
        uuid = UUID.randomUUID().toString(); // 세계 유일한 식별아이디 생성메서드.
        System.out.println("[" + uuid + "]" + " request scope bean create:" + this);
    }

    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "]" + " request scope bean close:" + this);
    }
}
```

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        String requestURL = request.getRequestURL().toString();
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        logDemoService.logic("testID");
        return "OK";

    }
}
```

- requestURL 값은 myLogger 인스턴스에 저장이 되는데, 그 객체 인스턴스가 요청마다 다르게 생성되기 때문에 값이 섞이거나 덮이지 않음. (request 스코프)

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {

    private final MyLogger myLogger;
    public void logic(String id) {
        myLogger.log("service id = " + id);
    }
}
```

- 기대하는 메서드 순서
  1. init();
  2. myLogger.log("controller test");
  3. logDemoService.logic("testID");
  4. close()
- 실제는 에러가 남. 왜냐? 스프링 애플리케이션이 실행될 때는 request가 없기 때문에 빈을 생성할 수가 없기 때문. 어떻게 해결?



### - 방법 1. ObjectProvider

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final ObjectProvider<MyLogger> myLoggerProvide;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        String requestURL = request.getRequestURL().toString();
        MyLogger myLogger = myLoggerProvide.getObject();
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        logDemoService.logic("testID");
        return "OK";

    }
}
```

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {

    private final ObjectProvider<MyLogger> myLoggerProvider;
    
    public void logic(String id) {
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.log("service id = " + id);
    }
}
```

```
[console]
[4914acc4-b8d0-4df2-8c37-6f4104997862] request scope bean create:com.example.spring_basic.common.MyLogger@431b38dd
[4914acc4-b8d0-4df2-8c37-6f4104997862][http://localhost:8080/log-demo] controller test
[4914acc4-b8d0-4df2-8c37-6f4104997862][http://localhost:8080/log-demo] service id = testID
[4914acc4-b8d0-4df2-8c37-6f4104997862] request scope bean close:com.example.spring_basic.common.MyLogger@431b38dd
```

- 기대하는 메서드 순서대로 작동이 됨.
- `.getObject()`때 빈이 생성되기 때문에 requestURL 값을 받을 시간을 벌 수 있음.
- 그러나 코드가 길어짐.

### - 방법 2. ProxyMode

- 가짜 클래스를 생성하여 getObject() 메서드를 숨기고 한 단계 뒤로 지연시키는 방법.

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS) // 인터페이스는 .INTERFACES
public class MyLogger {
	(중복코드 생략)
}
```

- Controller와 Service 코드는 `ObjectProvider` 사용 전과 동일

- 이렇게 되면 `MyLogger`클래스가 직접 주입되는 것이 아니라 해당 클래스를 상속받은 가짜 클래스가 컨테이너에 빈으로 등록되게 된다. [(싱글톤에서 CGLIB처럼 스프링이 조작한 클래스가 빈으로 등록됨)](https://jiyongyoon.github.io/spring/singleton/#configuration)

- 이 가짜 프록시 객체는 요청이 오면 그때 내부에서 진짜 빈을 요청하는 로직으로 구성. 그 때 진짜 `MyLogger.logic()`을 호출하게 됨.

- 핵심 아이디어는 **'필요 시점까지 객체 조회를 지연시킨다'**는 점.

  

------

> 마지막 수정일시: 2022-08-16 02:16