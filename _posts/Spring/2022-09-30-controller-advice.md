---
title: "Springboot 예외처리 방식 - @ControllerAdvice"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true
---





해당 포스트는 `어라운드허브 스튜디오`의 예외처리 강의와 개인적인 추가학습을 정리한 내용입니다.

------

# 1. Java의 예외 클래스

![예외 클래스 계층도](..\..\images\img_java_exception_class_hierarchy.png)

(사진출처: [TCP스쿨](http://www.tcpschool.com/java/java_exception_class))

- 파란색은 `Checked Exception`
- 주황색은 `Unchecked Exception`
  - 커스텀 exception은 `Unchecked Exception

|                       | Checked Exception                                            | Unchecked Exception                                          |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 처리여부              | 반드시 예외처리 필요<br />(문법적으로 오류가 있는 등의 예외이기 때문에 컴파일 시점에서 체크가 가능) | 명시적 처리 강제하지 않음<br />(데이터가 잘 들어올 것을 가정하고 코드를 구성하기 때문에 실행 중에 체크가 됨) |
| 확인시점              | **컴파일 단계** <u>(그 외 Exception)</u>)                    | 실행 중 단계 <u>(RuntimeException)</u>                       |
| 예외발생 시 트랜잭션* | 롤백하지 않음                                                | 롤백함                                                       |
| 예시                  | IOException<br />SQLException                                | NullPointerException<br />IndexOutOfBoundException<br />IllegalArgumentException |

- 예외발생 시 트랜젝션 처리*
  - 이 부분은 `스프링이 제공하는 기본적인 트랜잭션 처리로직`에 기반하며, 롤백을 하거나 하지 않는 것을 사용자가 명시할 수도 있으며, 비즈니스 로직 등에 따라 롤백을 할 지 말지도 정할 수 있음.

# 2. Springboot의 예외 처리 방식

### 1)  `@ControllerAdvice` / `@RestControllerAdvice`

- `@ControllerAdvice` / `@RestControllerAdvice` 를 사용하여 모든 `Controller`에서 발생할 수 있는 예외 처리
  - `@RestControllerAdvice(basePackages="패키지위치")`와 같이 설정을 통해 범위지정이 가능. (디폴트는 모든 Controller)
  - json 형태로 결과를 반환하려면 `@RestController` 사용.
  - 두 어노테이션 모두 `@Component`가 달려 있어서 부트시 자동으로 Bean 등록이 됨.
  - 보통 클래스에 달아서 사용함.

### 2)  `@ExceptionHandler`

- `@ExceptionHandler`를 통한 특정 `Controller`의 예외 처리(발생하는 예외마다 처리할 메서드를 정의)

  - `@ExceptionHandler(___Exception.class)`로 명시할 수 있으며, 명시할 클래스는 배열로 여러개도 받을 수 있음.
  - Exception은 최상위 클래스이기 때문에 하위 클래스 중에서 Handler 처리가 존재한다면 그 Handler가 우선권을 가지게 됨.
  - 보통 메서드에 달아서 사용함.

- 전역 설정인 `@ControllerAdvice` 보다 지역 설정인 `@ExceptionHandler`가 우선권을 가짐.

- [ 예시 ]

  - `controller 패키지` Controller.java

    ```java
    @RestController
    public class Controller {
        
        @PostMapping("/exception")
        public void exceptionTest() throws Exception {
            throw new Exception();
        }
        
        @ExceptionHandler(Exception.class)
        public void ExceptionHandler(Exception e) {
            System.out.println("Controller ExceptionHandler 호출")
        }
        
        
    }
    ```

  - `exception 패키지` ExceptionHandler.java

    ```java
    @RestControllerAdvice
    public class ExceptionHandler {
        
        @ExceptionHandler(Exception.class)
        public void ExceptionHandler(Exception e) {
            System.out.println("Advice ExceptionHandler 호출")
        }
    }
    ```

  - 위 상태로 Post로 /exception 요청시

    Advice는 전역 설정이기 때문에 Controller ExceptionHandler가 출력됨.

  - Controller에 ExceptionHandler가 없는 상태에서 Post로 /exception 요청시

    Controller 단에서 예외처리가 없기 때문에 Advice ExceptionHandler가 출력됨.





## 2-1. 예외 처리의 2가지 분류

- 처리가 가능한 예외: `try - catch - (final)`을 활용하여 그에 맞는 데이터를 넣는다던지 하는 적절한 처리를 함.
- 처리가 불가능한 예외: Exception을 클라이언트로 던져야 하기 때문에, 어떤 로그와 어떤 에러문구를 전달할지에 대한 고민이 필요함.

# 3. 실습

```java
@ControllerAdvice
public class ExceptionHandlerClass {
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<> ExceptionHandler(Exception ex){
        ErrorResponseEntity response = new ErrorResponseEntity(ErrorCode.INTERNAL_SERVER_ERROR);
        return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
    }
    
}
```

- 이렇게 되면 `Exception`이 발생했을 경우 `ResponseEntity`를 만들어서 반환하게 됨.
- 리턴 객체는 개발자가 커스터마이징이 가능해짐.

------

> 마지막 수정일시: 2022-12-18 17:43