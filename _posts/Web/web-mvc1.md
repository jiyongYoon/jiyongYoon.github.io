---
title: "웹 MVC와 기본 작동 구조"
layout: single
categories: Web
typora-root-url: ..\..\images\
toc: true
---



# 웹 MVC와 기본 작동 구조



## 1. 기본적인 MVC 패턴

: [과거 포스트 참고](https://jiyongyoon.github.io/spring/web-mvc/)





## 2. 회원가입 페이지 예제로 보는 기본 작동 구조

### 1) index.html

```html
<!DOCTYPE html>
<html lang="ko">
    <head>
        <meta charset="UTF-8">
        <title>LMS</title>
    </head>
    <body>
        <h1> LMS </h1>

        <div>
            <a href="/member/register">회원 가입</a>
             |
            <a href="/member/login">로그인</a>
             |
            <a href="/member/logout">로그아웃</a>
        </div>
        <hr/>
    </body>
</html>
```

- `<meta charset="UTF-8">` 한글 인코딩을 해야 글씨가 깨지지 않음.
- `<a href="/member/register">` 하이퍼링크를 통해 해당 루트의 html로 이동함.



### 2) register.html

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>회원 가입</title>
</head>
<body>
    <h1>회원 가입</h1>

    <form method="post">
        <div>
            <input type="email" name="userId" placeholder="아이디(이메일) 입력" required />
        </div>
        <div>
            <input type="text" name="userName" placeholder="이름 입력" required/>
        </div>
        <div>
            <input type="password" name="password" placeholder="비밀번호 입력" required/>
        </div>
        <div>
            <input type="password" name="rePassword" placeholder="비밀번호 재확인" required/>
        </div>
        <div>
            <input type="tel" name="phone" placeholder="전화번호 입력" required/>
        </div>
        <div>
            <button>회원 가입 하기</button>
        </div>
    </form>

</body>
</html>
```

- `<form method="post">` form은 클라이언트에게 필요한 정보를 전달받기 위해 사용하는 태그. 방식에는 `GET`과 `POST` 두 가지가 있음.

  - `GET` 입력한 값이 URL의 파라미터로 그대로 노출되어 전달됨. `/member/register?userId=value1&userName=value2&password=value3&phone=value4` 이런식으로 전달됨.
  - `POST` 입력한 값이 URL의 파라미터가 아닌 HTTP 메시지의 body에 담겨서 전달됨. (전달되는 값은 파라미터로 보냈던 값 형태와 동일하다고 생각하면 됨)
  - [참고 포스트](https://noahlogs.tistory.com/35)

  **웹 브라우저 주소 형식?* - 프로토콜://도메인(IP)/서버하위디렉토리?쿼리스트링(파라미터)

- `<input type="email" name="userId" placeholder="아이디(이메일) 입력" required />`
  - `type` 입력받는 정보의 형식. email은 @가 들어가 있어야 하고, password는 **로 표시되는 등의 역할을 해줌.
  - `name` 입력받은 정보의 변수명으로 사용됨.
  - `placeholder` 입력받을 칸 뒤에 옅은 회색으로 클라이언트에게 보여지는 가이드정보.
  - `required` 필수정보로 입력받지 않으면 제출이 안됨.



### 3) MemberController.java

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Controller
public class MemberController {

    @GetMapping("/member/register")
    public String register() {

        return "member/register"; // resource/templates/member/register파일로 간다.
    }

    // request 객체 :  WEB -> SERVER
    // response 객체 :  SERVER -> WEB
    @PostMapping("/member/register")
    public String registerSummit(HttpServletRequest request, HttpServletResponse response
        , MemberInput parameter) {

        System.out.println(parameter.toString());

        return "member/register";
    }
}

```

- `@Controller` : MVC 중 C에 해당. 클라이언트로부터 요청이 들어오면 어디서 처리할 지 담당을 정해준다고 생각하면 됨. Controller는 Veiw를 처리할 때 templates를 리턴해줌. (html, mustache 등) json으로 바인딩 되는 API의 경우 `RestController` 어노테이션을 사용함.

- `@GetMapping("/member/register")` : Get 방식으로 전달되는 요청에 대해서 처리하는 메서드 어노테이션. /member/register의 주소로 접속이 되면 return값을 반환함.

- `@PostMapping("/member/register")` : Post 방식으로 전달되는 요청에 대해서 처리하는 메서드 어노테이션. HTTP의 body에서 오는 정보는 `request`객체로 웹서버에 전달하고, 처리 후 반환되는 정보는 `response` 객체로 클라이언트에게 전달함.

  **같은 페이지인데 다른 방식으로 처리하는 이유?* - get으로 접속하는 클라이언트는 회원가입을 하기 위해 접속하러 들어온 것이고, post로 매핑이 되는 요청은 회원가입 정보를 입력한 후 가입요청을 했기 때문에 다르게 처리하는 것임.

- `public String registerSummit(HttpServletRequest request, HttpServletResponse response, MemberInput parameter)` : HttpServletRequest와 HttpServletResponse 객체는 request와 response를 담는 객체로 `javax.servlet.http.*` 자바 표준 라이브러리. `MemberInput parameter`는 MemberInput 클래스의 필드와 request의 body값이 일치하는 경우 자동으로 객체로 랩핑이 됨.

  

### 4) MemberInput.java

```java
@ToString
@Data
public class MemberInput {

    private String userId;
    private String userName;
    private String password;
    private String phone;
    
}
```

- `@Data`, `@ToString` : lombok 라이브러리. [(Spring 학습 시 포스트 참고)](https://jiyongyoon.github.io/spring/autoInjection/#lombok)
- 해당 클래스는 클라이언트로부터 정보를 전달받은 **<u>'Model'</u>**에 속함.



(계속 수정중)



------

> 마지막 수정일시: 2022-08-18 02:08