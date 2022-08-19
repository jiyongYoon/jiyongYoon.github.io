---
title: "메일을 활용한 비밀번호 초기화"
layout: single
categories: Web
typora-root-url: ..\..\images\
toc: true
---









# 메일을 활용한 비밀번호 초기화

- 아이디와 이름을 확인하여 본인이 맞으면 로그인과 마찬가지로 uuid를 통해 메일로 비밀번호 재설정을 진행함.



## 1. find_password.html

```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>회원 비밀번호 찾기</title>
</head>
<body>
    <h1>회원 비밀번호 찾기</h1>
    <div th:replace="/fragments/layout.html :: fragment-body-menu"></div>

    <form method="post">

        <div>
            <input type="email" name="userId" placeholder="아이디(이메일)을 입력" required/>
        </div>
        <div>
            <input type="text" name="userName" placeholder="이름 입력" required/>
        </div>
        <div>
            <button type="submit">비밀번호 초기화 요청</button>
        </div>

    </form>

</body>
</html>
```

- form을 제출하면 `userId`와 `userName`값이 넘어감.

## 

## 2. MemberController.java

```java
@PostMapping("/member/find/password")
public String findPasswordSubmit(Model model, ResetPasswordInput parameter) {

    boolean result = false;
    try {
        result = memberService.sendResetPassword(parameter);
    } catch(Exception e) {
    }
    model.addAttribute("result", result);

    return "member/find_password_result";
}

@GetMapping("/member/reset/password")
public String resetPassword(Model model, HttpServletRequest request) {

    String uuid = request.getParameter("id");

    boolean result = memberService.checkResetPassword(uuid);

    model.addAttribute("result", result);

    return "member/reset_password";
}

@PostMapping("/member/reset/password")
public String resetPasswordSubmit(Model model, ResetPasswordSubmitInput parameter) {

    boolean result = false;
    try {
        result = memberService.resetPassword(parameter.getId(), parameter.getPassword());
    } catch (Exception e) {
    }

    model.addAttribute("result", result);

    return "member/reset_password_result";
}
```

- `resetPassword`: `memberService.checkResetPassword`메서드에 uuid를 받아서 넘긴 후 실행된 결과를 `member/reset_password`에 넘긴다.
- `resetPasswordSubmit`: `memberService.resetPassword` 메서드에 제출된 id와 password를 받아서 넘긴 후 실행된 결과를 `member/reset_password_result`에 넘긴다.



## 3. MemberServiceImpl.java

```java
@Override
public boolean sendResetPassword(ResetPasswordInput parameter) {

    Optional<Member> optionalMember = memberRepository.findByUserIdAndUserName(
        parameter.getUserId(),
        parameter.getUserName()
    );

    if(!optionalMember.isPresent()) {
        throw new UsernameNotFoundException("회원 정보가 없습니다.");
    }

    Member member = optionalMember.get();

    String uuid = UUID.randomUUID().toString();

    member.setResetPasswordKey(uuid);
    member.setResetPasswordLimitDt(LocalDateTime.now().plusDays(1));
    memberRepository.save(member);

    String email = parameter.getUserId();
    String subject = "[fastLMS] 비밀번호 초기화";
    String text = "<p>fastLMS 비밀번호 초기화 메일입니다.</p><p>아래 링크를 클릭하셔서 비밀번호를 초기화 해주세요.</p>"
        + "<div><a href='http://localhost:8080/member/reset/password?id=" + uuid + "'> 비밀번호 초기화 </a></div>";
    mailComponents.sendMail(email, subject, text);

    return true;
}

@Override
public boolean resetPassword(String uuid, String password) {

    Optional<Member> optionalMember = memberRepository.findByResetPasswordKey(uuid);
    if(!optionalMember.isPresent()) {
        throw new UsernameNotFoundException("회원 정보가 없습니다.");
    }

    Member member = optionalMember.get();

    // 초기화 기한 1일 체크
    if(member.getResetPasswordLimitDt() == null) {
        throw new RuntimeException("유효한 날짜가 아닙니다.");
    }

    if( member.getResetPasswordLimitDt().isBefore(LocalDateTime.now())) {
        throw new RuntimeException("유효한 날짜가 아닙니다.");
    }

    String encPassword = BCrypt.hashpw(password, BCrypt.gensalt());
    member.setPassword(encPassword);
    member.setResetPasswordLimitDt(null); // 비밀번호 초기화를 하니 기한 초기화
    member.setResetPasswordKey(""); // uuid도 초기화
    memberRepository.save(member);

    return true;
}

@Override
public boolean checkResetPassword(String uuid) {

    Optional<Member> optionalMember = memberRepository.findByResetPasswordKey(uuid);
    if(!optionalMember.isPresent()) {
        return false;
    }

    Member member = optionalMember.get();

    // 초기화 기한 1일 체크
    if(member.getResetPasswordLimitDt() == null) {
        throw new RuntimeException("유효한 날짜가 아닙니다.");
    }

    if( member.getResetPasswordLimitDt().isBefore(LocalDateTime.now())) {
        throw new RuntimeException("유효한 날짜가 아닙니다.");
    }

    return true;
}
```



## 4. reset_password.html

```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>회원 비밀번호 초기화</title>

    <!-- 여기는 jquery에서 비밀번호 2번 입력받은 값이 동일한지 클라이언트단에서 확인하는 코드 -->
    <script src="https://code.jquery.com/jquery-3.6.0.min.js" integrity="sha256-/xUj+3OJU5yExlq6GSYGSHk7tPXikynS7ogEvDej/m4=" crossorigin="anonymous"></script>
    <script>

        $(document).ready(function () {

            $('form').on('submit', function () {

                var password = $(this).find('input[name=password]').val();
                var rePassword = $(this).find('input[name=rePassword]').val();

                if(password != rePassword) {
                    alert('입력하신 비밀번호값이 다릅니다.');
                    return false;
                }

            });
        });

    </script>
</head>
    
<body>
    <h1>회원 비밀번호 초기화</h1>
    <div th:replace="/fragments/layout.html :: fragment-body-menu"></div>

    <div th:if="${result eq true}">
        <form method="post">

            <div th:text="${uuid}">
            </div>
            <div th:text="${parameter}">
            </div>

            <div>
                <input type="password" name="password" placeholder="비밀번호 입력" required/>
            </div>
            <div>
                <input type="password" name="rePassword" placeholder="비밀번호 재확인" required/>
            </div>
            <div>
                <button type="submit">비밀번호 초기화하기</button>
            </div>

        </form>
    </div>

    <div th:if="${result eq false}">
        <p> 입력값이 유효하지 않습니다. </p>
        <div>
            <a href="/">메인으로 이동</a>
        </div>
    </div>

</body>
</html>
```



## 5. reset_password_result.html

```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>회원 비밀번호 초기화 결과</title>
    
</head>
<body>
    <h1>회원 비밀번호 초기화 결과</h1>
    <div th:replace="/fragments/layout.html :: fragment-body-menu"></div>

    <div th:if="${result eq true}">
        <p> 비밀번호가 초기화 되었습니다. </p>
        <div>
            <a href="/member/login">로그인 하기</a>
        </div>
    </div>
    <div th:if="${result eq false}">
        <p> 비밀번호 초기화에 실패했습니다. </p>
        <div>
            <a href="/member/find-password">재시도</a>
        </div>
    </div>

</body>
</html>
```





## 6. 전체 순서

1. `find.html` - id와 이름을 입력하여 비밀번호 초기화 요청을 보냄.
2. `Controller findPasswordSubmit()` - PostMapping으로 `ResetPasswordInput parameter`에 담겨온 정보를 `memberService.sendResetPassword(parameter)`에 담아서 넘기고, 처리된 결과를 result에 받아서 결과페이지로 넘김.
3. `ServiceImpl sendResetPassword()` - 넘겨받은 정보로 회원을 찾아서 예외처리. 이후에 uuid를 만들어서 클라이언트의 메일에 uuid를 넘기는 링크메일 발송
4. 클라이언트는 링크메일로 접속하여 uuid를 서버에 전달.
5. `Controller resetPassword()` - GetMapping으로 `HttpServletRequest request`에 담겨온 정보 중 uuid를`memberService.checkResetPassword(uuid)`에 담아서 넘기고 , 처리된 결과를 result에 받아서 결과페이지로 넘김.
6. `ServiceImpl checkResetPassword()` - 넘겨받은 정보로 회원을 찾아서 예외처리. 이후에 uuid와 기간이 유효하면 true 반환.
7. `reset_password.html` - 클라이언트 단에서 비밀번호 2번 입력받은 값이 동일한지 확인 후에 재설정 할 비밀번호 post로 보내기.
8. `Controller resetPasswordSubmit()` - PostMapping으로 `ResetPasswordSubmitInput parameter`에 담겨온 정보를 `memberService.resetPassword(parameter.getId(), parameter.getPassword())`에 담아서 넘기고, 처리된 결과를 result에 받아서 결과페이지로 넘김.
9. `ServiceImpl resetPassword()` - 넘겨받은 id와 password로 회원을 찾아서 예외처리. 이후에 password를 암호화한 후 memberRepository로 정보 넘기고 true 반환.
10. `reset_password_result.html` - 결과에 따라 페이지 반환.



------

> 마지막 수정일시: 2022-08-20 00:18