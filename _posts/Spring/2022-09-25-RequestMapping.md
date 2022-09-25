---
title: "@RequestMapping"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true
---



# 1. RequestMapping이란?

- URL을 컨트롤러의 메서드와 매핑할 때 사용하는 어노테이션.

- 클래스와 메서드에 모두 사용이 가능.

  ```java
  [컨트롤러 클래스에 사용]
  
  @RestController
  @RequestMapping("/user")
  @AllArgsConstructor
  public class UserController {
  }
  
  [메서드에 사용]
  @RequestMapping("/register")
  public ResponseEntity<?> register() {
  }
  -> 개인적으로 RequestMapping보다 메서드에는 가독성이 좋은 GetMapping이나 PostMapping을 쓰는 것을 선호함.
  ```

  - 클래스에 사용 시 클래스 메서드 전체에 적용됨.



# 2. RequestMapping을 동적으로 사용하기

- 경로변수 `@PathVariable`을 사용하여 파라미터를 동적으로 둘 수 있음.

  ```java
  @GetMapping("/mapping/{userId}")
  public String mappingPath(@PathVariable("userId") String data) {
      log.info("mappingPath userId={}", data);
      return "ok";
  }
  
  // 아래처럼 변수명과 파라미터 이름이 동일하면 생락도 가능
  @GetMapping("mapping/{userId}")
  public String mappingPath(@PathVariable String userId) {
      log.info("mappingPath userId={}", userId);
      return "ok";
  }
  
  // 다중으로도 사용 가능
  @GetMapping("/mapping/users/{userId}/orders/{orderId}")
  public String mappingPath(@PathVariable String userId, @PathVariable Long orderId) {
      log.info("mappingPath userId={}, orderId={}", userId, orderId);
      return "ok";
  }
  ```



- [참고자료](https://dahliachoi.tistory.com/42)

------

> 마지막 수정일시: 2022-09-25 22:11