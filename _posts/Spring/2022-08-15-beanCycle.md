---
title: "⑧. Bean Life Cycle"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true

---

이 포스트 시리즈는 inflearn의 ''스프링 핵심 원리 - 기본편 : 김영한'' 강의와 개인적인 추가학습을 정리한 내용입니다.

------

# 8. 빈 라이프 사이클(빈 생성 주기)

[ 정리 한 문장: 스프링 내부에서는`@PostConstruct`, `@PreDestroy` , 외부 라이브러리는 `@Bean(initMethod = "메서드명", destroyMethod = "메서드명")` ]

------

애플리케이션 서비스 시에, 비즈니스 로직을 시작하기 위한 빈 상태를 '초기화 상태'라고 한다. 이러한 상태가 되면 빈을 사용할 수 있고, 사용 후에는 안전하게 빈을 소멸시켜주는 것이 안정적일 것이다.

그렇다면, 빈은 언제 초기화되고, 언제 소멸될까? 그리고 나는 그것을 어떻게 알 수 있을까?





## 1. 스프링 빈의 라이프 사이클

1. 스프링 컨테이너 생성
2. 스프링 빈 생성 (+실제로 대부분의 생성자 주입이 이루어지는 단계. 빈이 생성될 때 생성자도 같이 호출되다보니)
3. 의존관계 주입: 앞에서 배운 setter 주입, 필드 주입 등의 객체 주입이 이루어지는 단계.
4. 초기화 콜백("나 준비 됐어요")
5. 비즈니스에 신나게 사용
6. 소멸전 콜백("안녕히 계세요 여러분")
7. 스프링 종료

```java
public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
    }

    public void setUrl(String url) {
        this.url = url;
    }

    // 서비스 시작 시 호출
    public void connect() {
        System.out.println("connect: " + url);
    }

    public void call(String message) {
        System.out.println("call: " + url + " message = " + message);
    }

    // 서비스 종료 시 호출
    public void disconnect() {
        System.out.println("close " + url);
    }
}
```

```java
public class BeanLifeCycleTest {

    @Test
    public void lifeCycleTest() {
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();
    }

    @Configuration
    static class LifeCycleConfig {

        @Bean
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spring.dev");
            return networkClient;
        }
    }
}
```

```
[console]
생성자 호출, url = null
connect: null
call: null message = 초기화 연결 메시지
```

- 분명히 생성자에서 setUrl로 필드에 값을 주입했지만, url은 null임.
- 아직 set 이후 비즈니스로직을 실행한 것이 없기 때문. 
- 즉, 빈을 만들때 생성자만 호출 된 상태. 의존관계 주입은 이후에 이루어졌기 때문.

**그렇다면 생성자에 초기화까지 포함시키면 안되나?* <br>- 생성자는 생성자 본연의 역할을 하도록!<br>- 초기화는 비즈니스 로직을 담당하기 위해 하는 것이니, 그에 맞게 따로 메서드를 사용하는 것이 가독성과 유지보수 관점에서 좋다.

**우리는 객체주입이 완료되었는지를 어떻게 알 수 있나?*<br>- 빈 생명주기 콜백을 사용하여!





## 2. 1번 방법) 인터페이스 사용

- `InitializingBean`과 `DisposableBean`
- 초기화 콜백과 소멸전 콜백을 할 인터페이스와 메서드를 구현하면 됨.

```java
public class NetworkClient implements InitializingBean, DisposableBean{

    (반복 코드 생략)
    
    // InitializingBean
    @Override
    public void afterPropertiesSet() throws Exception {
        connect();
        call("초기화 연결 메시지");
    }
    
    // DisposableBean
    @Override
    public void destroy() throws Exception {
        System.out.println("Bean destory.");
        disconnect();
    }
}
```

```java
public class BeanLifeCycleTest {
    //사용 시에는 따로 할 것은 없음
}
```

- 스프링 전용 인터페이스라 외부라이브러리에 사용 불가.
- 옛날 방식이라 개선이 이루어져, 현재는 거의 사용 안함.





## 3. 2번 방법)  설정 정보에 메서드 지정

- 콜백에 사용할 메서드를 Config 설정정보에 `@Bean(initMethod = "메서드명", destroyMethod = "메서드명")` 이렇게 지정해주면 됨.

```java
public class NetworkClient {

    (반복 코드 생략)
    
    public void init() {
        System.out.println("init start.");
        connect();
        call("초기화 연결 메시지");
    }

    public void close() {
        System.out.println("Bean close.");
        disconnect();
    }
}
```

```java
@Configuration
static class LifeCycleConfig {

    @Bean(initMethod = "init", destroyMethod = "close")
    public NetworkClient networkClient() {
        NetworkClient networkClient = new NetworkClient();
        networkClient.setUrl("http://hello-spring.dev");
        return networkClient;
    }
}
```

- 특장점은 외부라이브러리에 있는 초기화 콜백, 소멸전 콜백 메서드를 설정정보에 명명해주기만 하면 되기 때문에 사용이 가능해짐!!
- 특히, `destroyMethod`는 안정적인 소멸을 위해 적지 않아도 자동으로 호출을 함. 기본 세팅은 `close` 또는 `shutdown` 메서드를 찾아서 호출함.
- 사용하기 싫으면 `destroyMethod = ""`





## 4. 3번 방법) @PostConstruct, @PreDestroy

- 초기화 콜백 메서드에 `@PostConstruct`, 소멸전 콜백 메서드에 `@PreDestroy` 어노테이션 붙이면 끝.
- `javax.annotation.PostConstruct` 라이브러리. 즉, java에서 기본 지원하는 라이브러리라 스프링이 아니어도 사용 가능.

```java
public class NetworkClient {

    (반복 코드 생략)
    
    @PostConstruct
    public void init() {
        System.out.println("init start.");
        connect();
        call("초기화 연결 메시지");
    }

    @PreDestroy
    public void close() {
        System.out.println("Bean close.");
        disconnect();
    }
}
```



------

> 마지막 수정일시: 2022-08-14 03:49