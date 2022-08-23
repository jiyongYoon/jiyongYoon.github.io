---
title: "MyBatis와 Mapper"
layout: single
categories: Web
typora-root-url: ..\..\images\
toc: true
---

# MyBatis와 Mapper

- 웹 프로젝트에서 DB 쿼리문을 사용하는 sql Mapper중 하나. 
- vs. JPA : `extends JpaRepository` 상속받아 `.save()`메서드 등을 사용하는 것. @Entity가 선언된 Entity클래스를 객체로 사용함.

## Mybatis 사용을 위해 정의할 것

1. 쿼리문을 정의한 xml 파일 `MemberMapper.xml 의 <select> 태그 내부 </select>`
2. xml을 실행할 java 클래스 `MemberServiceImlp.java`
3. 쿼리의 결과데이터를 담는 중간 매개 Class(Dto, Vo 등) `MemberParam.java : web -> server` , `MemberDto.java : server -> web`
4. 위 3개의 위치를 알려주는 설정 `<select 내부의 id, parameterType, resultType 설정과 mapper의 namespace>`

- [참고 블로그](https://jung-story.tistory.com/128)



## 1) MemberMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zerobase.fastlms.admin.mapper.MemberMapper"> <!-- 여기에 Mapper클래스 위치설정 -->

    <!-- select에 쿼리문 작성 
	/ id는 Mapper.java의 메서드명
 	/ parameterType은 전달받을 객체
	/ resultType은 반환할 객체
	-->
    <select id="selectList" 
            parameterType="com.zerobase.fastlms.admin.model.MemberParam"
            resultType="com.zerobase.fastlms.admin.dto.MemberDto">

        select *
        from member
        where 1 = 1

          <if test="searchType != null and searchValue != null">
            <choose>
                <when test="searchType == 'userId'">
                    AND user_id like concat('%', #{searchValue}, '%')
                </when>
                <when test="searchType == 'userName'">
                    AND user_name like concat('%', #{searchValue}, '%')
                </when>
                <when test="searchType == 'phone'">
                    AND phone like concat('%', #{searchValue}, '%')
                </when>
                <otherwise>
                    AND
                    (
                        userId like concat('%', #{searchValue}, '%')
                        or
                        userName like concat('%', #{searchValue}, '%')
                        or
                        phone like concat('%', #{searchValue}, '%')
                    )
                </otherwise>
            </choose>
          </if>
    </select>
</mapper>
```



## 2) MemberParam.java

```java
@Data
public class MemberParam {
    String searchType;
    String searchValue;
}
```



## 3) MemberDto.java

```java
@Data
public class MemberDto {
    String userId;
    String userName;
    String phone;
    String password;
    LocalDateTime regDt;

    boolean emailAuthYn;
    LocalDateTime emailAuthDt;
    String emailAuthKey;

    String resetPasswordKey;
    LocalDateTime resetPasswordLimitDt;

    boolean adminYn;
}
```



## 4) list.html

```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>관리자 회원관리 페이지</title>
        <style>
            .list table {
                width: 100%;
                border-collapse: collapse;
            }
            .list table th, .list table td {
                border: solid 1px #000;
            }
            .search-form {
                padding: 5px 0 10px 0;
                text-align: right;
            }
        </style>
    </head>
    <body>
        <h1> 관리자 회원관리 페이지</h1>
        <div th:replace="/fragments/layout.html :: fragment-admin-menu"></div>
        <hr/>
        <!-- 이 부분은 페이지 내 리스트에서 검색기능 -->
        <div class="list">
            <div class="search-form">
                <form method="get">
                    <select name="searchType">
                        <option value="all">전체</option>
                        <option th:selected="${#strings.equals(param.searchType, 'userId')}" value="userId">아이디</option>
                        <option th:selected="${#strings.equals(param.searchType, 'userName')}" value="userName">이름</option>
                        <option th:selected="${#strings.equals(param.searchType, 'phone')}" value="phone">연락처</option>
                    </select>
                    <input th:value="${param.searchValue}" type="search" name="searchValue" placeholder="검색어 입력"/>
                    <button type="submit">검색</button>
                </form>
            </div>

			<!-- 이 부분은 페이지 내 리스트에 DB 회원목록 구현부 -->
            <table>
                <thead>
                    <tr>
                        <th>NO</th>
                        <th>아이디(이메일)</th>
                        <th>이름</th>
                        <th>연락처</th>
                        <th>이메일 인증 여부</th>
                        <th>가입일</th>
                        <th>관리자여부</th>
                    </tr>
                </thead>
                <tbody>
                    <tr th:each="item : ${list}">
                        <td>1</td>
                        <td th:text="${item.userId}"></td>
                        <td th:text="${item.userName}"></td>
                        <td th:text="${item.phone}"></td>
                        <td>
                            <p th:if="${item.emailAuthYn}">Y</p>
                            <p th:if="${item.emailAuthYn eq false}">N</p>
                        </td>
                        <td>
                            <p th:text="${item.regDt}"></p>
                        </td>
                        <td>
                            <p th:if="${item.adminYn}">Y</p>
                            <p th:if="${item.adminYn eq false}">N</p>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </body>
</html>
```



## 5) MemberServiceImpl.java

```java
@RequiredArgsConstructor
@Service
public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;
    private final MailComponents mailComponents;

    private final MemberMapper memberMapper;
 
    @Override
    public List<MemberDto> list(MemberParam parameter) {

        List<MemberDto> list = memberMapper.selectList(parameter);
        return list;

//        return memberRepository.findAll(); // 모든 것 가져오기 -> 그냥 다 가져오는곳, 위 메서드는 mybatis 사용하여 쿼리로 가져옴.
    } 
}
```



## 6) AdminMemberController.java

```java
@Controller
@RequiredArgsConstructor
public class AdminMemberController {

    private final MemberService memberService;

    @GetMapping("/admin/member/list.do")
    public String list(Model model, MemberParam parameter) {

        List<MemberDto> members = memberService.list(parameter);
        model.addAttribute("list", members);

        return "admin/member/list";
    }

}
```



## 7) MemberMapper.java

```java
@Mapper
public interface MemberMapper {

    long selectListCount(MemberParam parameter);
    List<MemberDto> selectList(MemberParam parameter);
}
```

- 인터페이스, implement하는 클래스는 없고, `@Mapper`어노테이션 붙여줌.
- 인터페이스 메서드는 `MemberMapper.xml`의 `<select id="메서드명" parameterType="전달받을객체" resultType="반환할객체"> 쿼리문 </select>`에서 실제로 수행하게 됨.



## 8) 전체 흐름

- `@AdminMemberController` - `@GetMapping`으로 클라이언트 요청을 `MemberParam parameter`에 받아서 `memberService.list()`에 전달하고, 처리된 결과를 `model.addAttribute()`를 통해 리스트에 담아서 `list.html`에 반환함.
- `MemberServiceImpl`- 컨트롤러가 전달한 `MemberParam parameter`를 `Mapper.xml`을 통해 쿼리수행을 하여 DB에 있는 리스트를 가져온 후, 리스트를 반환함.
- `MemberMapper.xml` - `<select> </select>` 내부의 조건에 맞는 쿼리를 실행함.

------

> 마지막 수정일시: 2022-08-23 11:08