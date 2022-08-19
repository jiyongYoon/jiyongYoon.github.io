---
title: "스프링 웹 시큐리티로 로그인&로그아웃 구현"
layout: single
categories: Web
typora-root-url: ..\..\images\
toc: true
---



# 스프링 웹 시큐리티로 로그인&로그아웃 구현 

- 기존 방식: `WebSecurityConfigurerAdapter` 상속 후 `configure`메서드 오버라이딩.

- 새로운 방식: `SecurityFilterChain` 빈 등록. (Spring Security 5.7.0 이후부터 적용)

  

## 1) SecurityConfiguration.java

- 모든 사이트에 접근 허가하는 예시코드

```java
[기존]
@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter { // 상속
	
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            	.antMatchers("/", "/**")
            	.permitAll();
        
        super.configure(http);
    }
}
```

```java
[새로운 방식]
@Configuration
@EnableWebSecurity
public class SecurityConfiguration {
    
    @Bean // 빈 등록
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/", "/**")
                .permitAll();

        return http.build();
    }
}
```



```java
[실제 예시 코드]
@Configuration
@EnableWebSecurity
public class SecurityConfiguration {

    @Bean // 패스워드 발급 메서드
    public PasswordEncoder getPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean // 로그인 실패처리 메서드
    UserAuthenticationFailureHandler getFailureHandler() {
        return new UserAuthenticationFailureHandler();
    }

    @Bean // 로그인 권한 관련설정 메서드
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http.csrf().disable(); // 학습필요

        http.authorizeRequests() 
                .antMatchers(
                        "/"
                        , "/member/register"
                        , "/member/email-auth"
                )
                .permitAll() // 위 사이트는 접근 허가
                .anyRequest() // 나머지 사이트는 권한 요청
                .authenticated();

        http.formLogin()
                .loginPage("/member/login") // 로그인 페이지
                .failureHandler(getFailureHandler()) // 실패 시 실패처리
                .permitAll();

        http.logout() // 로그아웃
                .logoutRequestMatcher(new AntPathRequestMatcher("/member/logout"))
                .logoutSuccessUrl("/")
                .invalidateHttpSession(true);

        return http.build();
    }

    @Bean // 사용자 권한 주는 메서드
    AuthenticationManager authenticationManager(AuthenticationConfiguration auth) throws Exception {
        return auth.getAuthenticationManager();
    }
}
```



## 2) MemberServiceImpl.java

```java
@RequiredArgsConstructor
@Service
public class MemberServiceImpl implements MemberService {
    
    @Override // MemberService에서 상속한 UserDetailsService 클래스의 메서드 오버라이드
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        Optional<Member> optionalMember = memberRepository.findById(username);
        if(!optionalMember.isPresent()) {
            throw new UsernameNotFoundException("회원 정보가 없습니다.");
        }

        Member member = optionalMember.get();

        if(!member.isEmailAuthYn()) { // 메일 활성화 안되었을 때 오류 처리
            throw new MemberNotEmailAuthException("이메일 활성화 이후에 로그인해주세요.");
        }
        
		// Config 클래스에서 사용자 권한 관련 메서드와 연관
        List<GrantedAuthority> grantedAuthorities = new ArrayList<>(); 
        grantedAuthorities.add(new SimpleGrantedAuthority("ROLE_USER"));
		// 유저의 아이디와, 비밀번호, 그리고 역할(권한)을 담아서 리턴함
        return new User(member.getUserId(), member.getPassword(), grantedAuthorities);
    }
}
```

```java
[MemberService.java]
public interface MemberService extends UserDetailsService <- 해당 클래스 상속
```



## 3) MemberNotEmailAuthException.java

```java
public class MemberNotEmailAuthException extends RuntimeException {
    public MemberNotEmailAuthException(String s) {
        super(s);
    }
}
```



## 4) UserAuthenticationFailureHandler.java

```java
public class UserAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, 		
                 AuthenticationException exception) throws IOException, ServletException {

        String msg = "로그인에 실패하였습니다.";

        if(exception instanceof InternalAuthenticationServiceException) {
            msg = exception.getMessage();
        }

        setUseForward(true);
        setDefaultFailureUrl("/member/login?error=true");
        request.setAttribute("errorMessage", msg);
        System.out.println(msg);

        super.onAuthenticationFailure(request, response, exception);
    }
}
```

- 정확한 작동에 대해서는 추후 학습이 필요함. (빈으로 등록되어 관리되는 것을 따라가기가 어렵네....)



(해당 포스트는 내용이 많이 부실하여 추후 학습하여 보강할 예정입니다.)

------

> 마지막 수정일시: 2022-08-19 13:15
