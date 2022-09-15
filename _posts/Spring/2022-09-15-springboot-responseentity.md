---
title: "SpringBoot에서 사용되는 ResponseEntity"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true
---



## ResponseEntity

- Spring Framework에서 제공하는 클래스로, 일반적인 API에서 반환하는 리소스(상태코드, 응답 메시지 등을 포함) 객체 자체로 쓰임.

- `HttpEntity` 클래스를 상속받음.

  - 데이터를 형식에 맞게 가공해서 보내기 위해서는 `Http` 에 맞게 송-수신을 해야하기 때문에 당연히 `Http` 형식을 따르기 위함.

    **프론트단과 데이터를 주고받기 위해서는 `통신`이 필요.*

  - `HttpStatus`, `HttpHeaders`, `HttpBody`를 포함.

- ### ResponseEntity 구조 (와 상속받은 HttpEntity의 구조)

  ```java
  public class ResponseEntity extends HttpEntity {
    	private final Object status;
  }
  
  
  public class HttpEntity<T> {
      public static final HttpEntity<?> EMPTY = new HttpEntity<>();
    
      private final HttpHeaders headers;
    
      @Nullable
      private final T body; // Generic type = 밖에서 Wrapping될 타입을 지정할 수 있음.
  }
  ```

- ### ResponseEntity를 활용하여 return하는 예시 (배당금 프로젝트에서)

  ```java
  @PostMapping         // 여기 제네릭의 ? 는 와일드카드. 런타임까지 유연하게 가져간다는 것.
  public ResponseEntity<?> addCompany(@RequestBody Company request) {
      String ticker = request.getTicker().trim();
      if(ObjectUtils.isEmpty(ticker)) {
          throw new RuntimeException("ticker is empty");
      }
  
      Company company = this.companyService.save(ticker);
      this.companyService.addAutocompleteKeyword(company.getName());
  
      return ResponseEntity.ok(company);
  }
  ```

  - `addCompany` 메서드는 `Company`형태의 파라미터를 전달받는데, http 형태에 맞게 `body`로 전달받기 위해 `@RequestBody`를 붙여줌.

  - 메서드 안에서 형태를 가공함.

  - `ResponseEntity` 타입으로 return.

    - 생성자로도 리턴이 가능하지만, Builder패턴을 더 선호.(일반적인 Builder 패턴 사용이유와 비슷)

    - 상태코드만 반환 시

      ```java
      return ResponseEntity.ok().build();
      ```

    - Body까지 보내줄 때

      ```java
      return ResponseEntity.ok(Body에 넣을 객체);
      ```

      

## @RequestParam

- HTTP 파라미터 형태로 값을 전달받을 때 사용하는 어노테이션.

  ```java
  @PostMapping("/api/notice")
  public NoticeModel addNotice(@RequestParam String title, @RequestParam String contents) {
      NoticeModel noticeModel = NoticeModel.builder()
          .id(1)
          .title(title)
          .contents(contents)
          .regDt(LocalDateTime.now())
          .build();
  
      return noticeModel;
  }
  ```

  ```http
  ###
  POST http://localhost:8080/api/notice
  Content-Type: application/x-www-form-urlencoded
  
  title=제목1&contents=내용1
  ```

  ![image-20220915004738189](..\..\images\image-20220915004738189.png)



## @RequestBody

- HTTP 바디의 형태로 값을 전달받을 때 사용하는 어노테이션.

  ```java
  @PostMapping("/api/notice")
  public NoticeModel addNotice(@RequestBody NoticeModel noticeModel) {
      noticeModel.setId(3);
      noticeModel.setRegDt(LocalDateTime.now());
  
      return noticeModel;
  }
  ```

  ```http
  ###
  POST http://localhost:8080/api/notice
  Content-Type: application/json
  
  {
    "title" : "제목3",
    "contents" : "내용3"
  }
  ```

  ![image-20220915004631058](..\..\images\image-20220915004631058.png)



- [참고자료1](https://tecoble.techcourse.co.kr/post/2021-05-10-response-entity/)
- [참고자료2](https://thalals.tistory.com/268)

------

> 마지막 수정일시: 2022-09-15 02:08