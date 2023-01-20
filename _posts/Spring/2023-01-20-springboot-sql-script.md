---
title: "Springboot 개발환경에서 서버 부팅시 sql문 실행방법"
layout: single
categories: Spring
typora-root-url: ..\..\images\
toc: true
tag: []
---



Springboot 개발환경에서 SpringbootTest나 api 테스트를 해야할 경우, 서버 부팅 시 베이스가 되는 데이터들을 미리 세팅해주어야 할 필요가 있을 때가 있다.

이때, 프로젝트 디랙토리 내에 `.sql` 파일을 생성하여 자동으로 sql문을 실행할 수 있게 세팅할 수 있다.


## 개발환경

해당 포스트는 H2, JPA를 사용하는 것으로 가정하고 작성한다.

## 기본세팅

- dir: `src/main/resources`
- filename: `data.sql` 또는 `schema.sql`
  - db에 따라서 `schema-${platform}.sql` / `data-${platform}.sql` 처럼 파일명을 다르게 가져갈 수 있음.
  - properties도 `spring.sql.init.platform*=h2`
    (`hsqldb`, `h2`, `oracle`, `mysql`, `postgresql`, and so on) 처럼 세팅해주면 됨.

## properties

나머지 세팅들은 properties에서 함께 해주면 된다.

```groovy
// H2
spring.jpa.database=h2
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:test
spring.datasource.username=sa
spring.datasource.password=

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

// data.sql
spring.jpa.defer-datasource-initialization=true 
spring.jpa.generate-ddl=true

//jpa
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=create
```

`spring.jpa.defer-datasource-initialization=true`

- boot 2.5 이상 버전에서는 하이버네이트 초기화 과정보다 data.sql이 먼저 실행되도록 변경되었기 때문에 해당 옵션을 true로 바꿔주어야 sql 스크립트가 나중에 실행됨

`spring.jpa.generate-ddl`

- @Entity 어노테이션이 명시된 클래스를 찾아서 ddl을 어떻게 할 것인지
- true → 해당 클래스를 찾아 ddl을 생성하고 실행
    
    `spring.jpa.hibernate.ddl-auto`
    
    - none - 엔티티가 변경되더라도 데이터베이스를 변경하지 않는다.
    - update - 엔티티의 변경된 부분만 적용한다.
    - validate - 변경사항이 있는지 검사만 한다. (다르면 예외 발생)
    - create - 스프링부트 서버가 시작될때 모두 drop하고 다시 생성한다.
    - create-drop - create와 동일하다. 하지만 종료시에도 모두 drop 한다.
    
    **개발 환경에서는 보통 update 모드를 사용하고 운영환경에서는 none 또는 validate 모드를 사용한다.**


[data.sql 스크립트]
```SQL
INSERT INTO Users(ID, NAME) VALUES
                (1, 'HELLO'),
                (2, 'WORLD'),
                (3, 'NEW');
```
- 테이블 생성 sql은 없어도 됨. @Entity 어노테이션을 찾아서 알아서 ddl-auto 설정을 따름.

## 참고자료
[`springboot 공식문서`](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.data-initialization)


------

> 마지막 수정일시: 2023-01-20 09:19
