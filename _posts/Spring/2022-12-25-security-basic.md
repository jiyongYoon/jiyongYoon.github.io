---
title: "SpringSecurity - 기본"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true
tag: [SpringSecurity]
---



Spring Security는 Spring 프레임워크에서 제공하는 인증, 인가를 쉽게 처리할 수 있는 방식을 `Filter`를 활용하여 제공한다.

따라서, 기본 구조가 되는 Filter에 대해 알아본다.

# 1. Filter

## 1) Filter

- Spring Security의 Servlet은 Servlet Filter(javax.servlet)를 기반으로 하고 있음.
- 클라이언트가 HTTP를 요청하면 Servlet으로 가는 과정에 Filter Chain을 만들어서 처리함.
- 스프링 MVC 어플리케이션에서 Servlet은 DispatcherServlet을 말함.

## 2) DelegatingFilterProxy

- Servlet container - DelegatingFilterProxy - ApplicationContext 에 위치함.
- Spring Security에서는 이를 통해 <u>**Spring Bean으로 구현한 모든 필터를 등록**</u>함. (기본적으로 스프링 서블릿 컨테이너에서는 자체 표준으로 제시하는 필터 등록만을 허용한다)

## 3) FilterChainProxy

- Spring Security의 실질적인 서블릿 지원은 여기에 포함됨.
- 다양한 필터 인스턴스를 `SecurityFilterChain`을 통해 위임함.

-> 여기까지 정리하자면, 

1. Spring은 Servlet으로 가는 과정에 Filter를 거치게 되는데
2. Spring Security는 ApplicationContext 전에 `DelegatingFilterProxy`라는 필터를 가진다.
3. 이 필터는 Spring Bean으로 구현한 모든 필터를 등록하게 되는데,
4. `FilterChainProxy`라는 묶음을 빈으로 등록하여 필터를 사용하게 되고
5. 이 묶음을 구성하는 필터체인은 `SecurityFilterChain`에 위임하게 된다.

-> 더 간단하게 정리하자면, <u>**Spring Security는 ApplicationContext 전에 동작하는 필터를 `SecurityFilterChain`을 통해 생성(위임)한다.**</u>

## 4) SecurityFilterChain

![multi securityfilterchain](..\..\images\multi-securityfilterchain.png)

- 각 FilterChain이 동작하는 URL을 등록하고, 해당 요청이 있을 때 그에 맞는 FilterChain을 선택하고 실행하게 됨. (ex, /api/** 로 접근하는 사람들은 Token 처리를 하고, /** 로 접근하는 사람들은 Session 처리를 하는 식으로)

## 5) Filter Stack

: Security 필터들은 아래와 같은 순서로 적용되어 있음. (순서를 알 필요는 없지만, 도움이 된다고 공식문서에 작성되어 있음)

- [`ForceEagerSessionCreationFilter`](https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html#session-mgmt-force-session-creation)
- `ChannelProcessingFilter` - HTTPS관련 내용을 처리하기 위한 필터
- `WebAsyncManagerIntegrationFilter` 
- `SecurityContextPersistenceFilter` - 인증을 하게 되면 담길 사용자 인증 정보들을 가진 객체인 SecurityContext를 영속하기 위해서 필요한 필터
- `HeaderWriterFilter` - 헤더값에 대한 가감을 설정할 경우 활용하는 필터
- `CorsFilter`
- `CsrfFilter` - CSRF(Cross-site request forgery) 공격에 대하여 처리하는 필터 -> csrf token을 활용하여 HTTP 요청에 대한 검증을 함
- `LogoutFilter` - 로그아웃 처리 필터
- `OAuth2AuthorizationRequestRedirectFilter`
- `Saml2WebSsoAuthenticationRequestFilter`
- `X509AuthenticationFilter` - 공개키 인증서와 인증 알고리즘 표준 등을 처리하기 위한 필터
- `AbstractPreAuthenticatedProcessingFilter` - 사전 인증된 요청을 처리하는 필터를 위한 기본 <u>클래스</u>. 인증의 주체가 이미 외부 시스템에 의해 인증 되었다고 가정하고 해당 필터를 활용하게 됨. 즉, Spring Security에서 인증 처리를 하지 않고 다른것으로 하게 되면 해당 필터를 활용하게 됨.
- `CasAuthenticationFilter` - CAS(Central Authentication Service) ' 중앙 인증 서비스'의 약자. SSO를 지원하는 별도의 인증 서비스. 이를 통한 인증 처리를 하는 필터. 위 `AbstractPreAuthenticatedProcessingFilter`가 이런 필터를 위한 기본 클래스임.
- `OAuth2LoginAuthenticationFilter` - OAuth2 인증 처리를 위한 필터
- `Saml2WebSsoAuthenticationFilter`
- [`UsernamePasswordAuthenticationFilter`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/form.html#servlet-authentication-usernamepasswordauthenticationfilter) - Spring Security의 기본 필터. 유저의 이용자명(username)과 비밀번호(password)을 담은 `UserDetail` 과 이를 처리하는 `UserDetailsService`를 활용하여 인증을 진행함.
- `DefaultLoginPageGeneratingFilter`
- `DefaultLogoutPageGeneratingFilter`
- `ConcurrentSessionFilter` - 동시 세션의 수를 관리해주는 필터 -> 동일 아이디로 1명만 로그인 한다거나 하는 처리들을 하게 해줌.
- [`DigestAuthenticationFilter`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/digest.html#servlet-authentication-digest)
- `BearerTokenAuthenticationFilter`
- [`BasicAuthenticationFilter`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/basic.html#servlet-authentication-basic) - HTTP 기본 인증 처리를 위해 제공되는 필터
- [`RequestCacheAwareFilter`](https://docs.spring.io/spring-security/reference/servlet/architecture.html#requestcacheawarefilter)
- `SecurityContextHolderAwareRequestFilter`
- `JaasApiIntegrationFilter` - Java 프로그래밍 언어의 보안 프레임워크
- `RememberMeAuthenticationFilter` - 세션이 사라지거나 만료되더라도 쿠키 또는 DB를 사용하여 저장된 토큰 기반으로 인증을 처리하는 필터
- `AnonymousAuthenticationFilter` - 익명자 처리를 위한 필터
- `OAuth2AuthorizationCodeGrantFilter`
- `SessionManagementFilter` - 세션 변조 공격 방지기능 필터
- [`ExceptionTranslationFilter`](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-exceptiontranslationfilter) - 인증 및 인가에 대한 예외 처리 필터
- [`FilterSecurityInterceptor`](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-requests.html#servlet-authorization-filtersecurityinterceptor) - 특정 리소스(URI)로 접근하게 되면 최종 접근 전에 `AccessDecisionManager`를 사용하여 인가처리를 하는 필터
- `SwitchUserFilter` - 이용자 전환 처리를 위한 필터. 이용자 정보(Context)를 전환하고자 하는 대상과 바꾸게 됨



# 2. UsernamePasswordAuthenticationFilter

- SpringSecurity에서 사용하는 http/form-login을 지원하는 기본적이면서 핵심적인 필터.

![img](..\..\images\image-4.png)

- 그림에서는 `WebSecurityConfigureAdapter`로 표시되어 있는 곳이 최신 버전들에서는 `SecurityFilterChain`으로 변경됨.
- 사용자가 입력한 인증정보를  IDP(identify provider, ex - db, in memory, oauth 등)에 있는 정보들과 비교하여 검증함.
- `Security Config`에서 설정하는 이부분은 `인증`에 대한 부분을 처리하게 되며, 이후 해당 내용을 Session에 넣거나, 리다이렉트 하는등의 후처리 부분은 `Success/Failuer Handler`에서 처리하게 됨.



# 3. IDP

- `In-memory` : Spring Security에 직접적으로 이용자 이름, 비밀번호, 권한 등을 설정하여 서비스 제공
- `JDBC` : RDBMS를 이용하여 이용자 정보 저장하고 인증할 수 있도록 설정하여 서비스 제공
- `LDAP` : LDAP를 활용하여 이용자 정보를 제공. 이는 별도의 서비스로 제공됨. Security에서는 이미 다른 서비스로 처리된 것으로 진행
- `SAML` : SP / IDP로 나뉨. Spring Security에서 SAML을 사용하겠다고 하면 SP는 Spring Security가 되고, 이 정보를 제공하는 외부 서비스들은 IDP가 됨.
- [`OAuth`](https://jiyongyoon.github.io/web/Oauth2.0/) : Server / Client로 나뉨. 



------

> 마지막 수정일시: 2022-12-25 12:59
