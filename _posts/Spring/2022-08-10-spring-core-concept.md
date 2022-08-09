---
title: "1. 객체지향 설계와 스프링"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true
---

이 포스트 시리즈는 inflearn의 ''스프링 핵심 원리 - 기본편 : 김영한'' 강의를 학습하며 정리한 내용입니다.

------

# 1. 객체지향 설계와 스프링

[ 정리 한 문장: Spring은 Java의 객체지향, 다형성을 극대화하고자 사용하게 되는 프레임워크다]

------



## 1) 자바 개발의 과거와 스프링

- Java개발의 메인 기술이었던 EJB라는 툴로 자바 개발함.
- 기능은 많았지만 세팅, 학습 등의 난이도가 엄청나게 어려웠음.
- EJB의 문제점을 개선한 로드 존슨이 예제 코드를 책으로 출간함.
- 2003년 스프링 Framework 1.0 출시 - XML 설정
- 2009년 스프링 Framework 3.0 출시 - 자바 코드로 설정 가능해짐
- 2014년 스프링 부트 1.0 출시 - spring의 단점인 build, setting을 보완함
- 현재 현업에서는 스프링 프레임워크를 기반으로한 스프링부트를 거의 대부분 사용하고 있으며, 스프링 생태계가 커지고 있음.(스프링 데이터, 스프링 세션, 스프링 시큐리티, 스프링 Rest Docs, 스프링 배치, 스프링 클라우드 등)





## 2) 스프링은 왜 쓰는가?

: 스프링은 자바 언어 기반의 프레임워크. 자바 언어의 가장 큰 특징은 '객체 지향 언어'. 이 특성을 가장 잘 살려내기 위한 컨셉으로 시작된 것이 바로 스프링, 스프링 프레임워크!



### - [객체 지향 프로그래밍(OOP)](https://jiyongyoon.github.io/java/Heritance/#%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-%EC%83%81%EC%86%8Dheritance-%EB%8B%A4%ED%98%95%EC%84%B1-%EC%B6%94%EC%83%81%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4)

- 핵심개념: 다형성. 즉, 컴퓨터 프로그램을 '명령어의 모음'으로 보는 것이 아니라 '각각 객체의 모임'으로 보는 것. 각 객체는 메시지를 직접 주고받으며, 데이터 처리가 가능.
- 이러한 특성은 프로그램을 **유연**하고 **변경이 용이**하게 만들어 줌.

#### * 인터페이스와 객체

- **인터페이스:** 역할. 사용자(클라이언트)는 인터페이스의 작동원리를 몰라도 됨. 작동하는 규칙만 알면 됨.
  - 자동차의 역할: 핸들로 방향 조작, 엑셀과 브레이크로 이동.
  - 키보드의 역할: 매핑된 키를 눌러 컴퓨터에 데이터 입력.
  - 콘센트의 역할: 220v 교류 전기를 사용가능하게 함.
  - TV리모컨의 역할: 버튼을 눌러 TV를 동작함.
- **객체:** 구현체. 사용자(클라이언트)는 구현체의 역할을 알기 때문에 역할을 수행할 수 있음.
  - 자동차 구현체: 그랜져, 테슬라, 버스 등
  - 키보드 구현체: 로지텍 키보드, 기계식 키보드 등
  - 콘센트 구현체: 벽체 콘센트, 멀티텝 등
  - TV리모컨 구현체: LG리모컨, 삼성리모컨 등

#### * 다형성

- 인터페이스를 구현한 객체는 실행 시점에 유연하게 변경할 수 있다.

  ```java
  public class MemberService implements MemberRepository {
  	private MemberRepository memberRepository = new MemoryMemberRepository();
      // or
  	private MemberRepository memberRepository = new JdbcMemberRepository();
  }
  ```

- 인터페이스에 맞는 구현체를 무한하게 설계하면 계속 확장이 가능해진다!!

- 대신, 인터페이스가 변경되면 큰 변경이 발생하게 된다. --> **인터페이스를 안정적으로 설계하는 것이 가장 중요!**

#### * 스프링을 쓰는 이유!

- 자바 언어의 특징인 '객체 지향', '다형성'을 극대화해서 규모가 큰 프로젝트들도 효율적으로 구현할 수 있게 해줌.
- <u>이 부분에서 '스프링 컨테이너'. 즉, 제어의 역전(IoC), 의존관계 주입(DI)이라는 핵심 개념이 등장함!!</u>





## 3) SOLID 원칙

### 1. SRP - 단일 책임 원칙

- 한 클래스는 하나의 책임만. (변경이 있을 때 그 클래스만 바꾸면 된다면 잘 따르고 있다는 것)

### 2. OCP - 개방-폐쇄 원칙

- 확장에는 열려있으나 변경에는 닫혀있어야 함. 그러나... Java에서는 따를 수 없다.

  ```java
  public class MemberService implements MemberRepository {
  	private MemberRepository memberRepository = new MemoryMemberRepository();
  	private MemberRepository memberRepository = new JdbcMemberRepository();
  }
  ```

  변경 시 클라이언트가 직접 new로 구현할 객체를 선택하고 있음. ***-> Spring 컨테이너가 필요한 이유!***

#### 3. LSP - 리스코프 치환 원칙

- 인터페이스의 역할을 지켜야 함. 더하기 인터페이스인데 빼기가 작동되면 안됨.

#### 4. ISP - 인터페이스 분리 원칙

- 인터페이스도 책임을 나눌수록 좋다. 자동차 인터페이스에도 운전자 인터페이스, 정비사 인터페이스 등으로 분류하면 역할이 더 분명해지고 대체 가능성도 높아진다.

#### 5. DIP - 의존관계 역전 원칙

- 클라이언트(사용자) 코드가 구현클래스(구현체)가 아닌 인터페이스(역할)를 의존하도록! 그러나... 이 역시 Java에서는 따를 수 없다.

  ```java
  public class MemberService implements MemberRepository {
  	private MemberRepository memberRepository = new MemoryMemberRepository();
  	private MemberRepository memberRepository = new JdbcMemberRepository();
  }
  ```

  계속 동일한 문제다.

  Java는 다형성이 특징이자 장점이지만, Java만으로는 SOLID 원칙이 지켜지지 않는다. 

***-> Spring 컨테이너(의존관계, 의존성 주입)가 필요한 이유!***

*Q. 모든 구현에 인터페이스를 다 만드는 것이 과연 유리할까? - 실무적인 고민으로, 인터페이스를 모두 구현할 때 발생하는 비용과 그렇게 하지 않았을 때를 비교하여 장점이 단점을 넘어설 때 적용하는 것이 좋음.*

------

> 마지막 수정일시: 2022-08-10 00:57