---
title: "[albumProject] 1. 프로젝트 세팅"
layout: single
categories: java_Project
typora-root-url: ..\..\images\
toc: true
---





# 1. spring initializr

- 빌드: Gradle

- SpringBoot 버전: 2.7.3

- 패키지: Jar

  - 패키지에 대한 고민이 있었다. 정보를 검색해본 결과, SpringBoot를 사용하여 프로젝트를 만들때는 기본적으로 Jar로 패키지 세팅을 해주는 것이 좋다는 것을 확인하였고, Jar를 선택했다.

- 디펜던시

  - Lombok: 없으면 스프링부트 개발 못할듯.
  - Spring Web: Rest API를 만들기 위해서 추가.

  ![image-20220907135508134](..\..\images\image-20220907135508134.png)





# 2. DB 연동작업

- 기본적으로 mysql을 사용하고, 추가로 사진데이터는 aws의 s3를 사용하는 것을 생각중이다.

  - 바로 생겨버린 첫번째 에러. gradle 빌드가 완료되지 않았음. -> 어플리케이션 실행하니 바로 컴파일 에러가 떴고, jpa 디펜던시에 대한 내용이었다. 오탈자가 없는지 확인했고 빌드도 여러번했지만, 이상한 곳에서 꼬였겠거니 하고 다시 타이핑하여 빌드하니 패스!

- 기존에 사용하던 mysql에 새로운 DB 스키마를 생성하고 엔티티를 하나씩 만들어줬다.

  - 만드는 도중 두번째 에러.

  ```sql
  [sql문]
  create table family(
  	family_id long not null primary key auto_increment
  );
  ```

  ```
  [mysql error]
  Error code: 1063. Incorrect column specifier for column '~~~'
  ```

  - 원인

    - auto_increment 옵션은 int나 float같은 숫자값에만 사용할 수 있다. <u>잉? long은 숫자값이 아니라고?</u> 응 아니야.

  - 해결

    - bigint를 사용함. int와 bigint 중 고민을 하였지만, 1) pk값의 컨셉에 맞는 '최대한의 많은 수'를 표현하기 위함이였고, 2) 서버의 용량이나 효율은 현재 개발 단계에서 깊게 고려할만한 요인은 아니라고 판단(영향 미미)

  - 만드는 도중 세번째 에러. (워닝 문구)

    ```sql
    [sql문]
    create table user(
    	user_id bigint not null primary key auto_increment,
        phone VARCHAR(12) not null,
        password VARCHAR(50) not null,
        username VARCHAR(50) not null,
        family_relationship VARCHAR(50) not null,
        email VARCHAR(50) not null,
        confirm tinyint(1) not null,
        representative tinyint(1) not null,
        profile_path VARCHAR(255) null,
        create_dt datetime not null,
        modify_dt datetime null,
        delete_dt datetime null
    );
    ```

    ```
    [mysql warning]
    integer display width is deprecated and will be removed in a future release
    ```

    - 원인
      - tinyint안에 (1) 때문. 이제 사용하지 않으며 넘어가면 데이터가 지워질수도 있다는 경고같음. 데이터 저장 및 사용에는 지장이 없다는 글을 발견하였으나 신경쓰임.
    - 해결
      - (숫자) 없앰.

  

  

## - mysql의 데이터타입



### 1) 문자형 데이터타입

| 데이터 유형   | 정의                                                         |
| ------------- | ------------------------------------------------------------ |
| CHAR(n)       | 고정 길이 데이터 타입(최대 255byte)- 지정된 길이보다 짦은 데이터 입력될 시 나머지 공간 공백으로 채워진다. |
| VARCHAR(n)    | 가변 길이 데이터 타입(최대 65535byte)- 지정된 길이보다 짦은 데이터 입력될 시 나머지 공간은 채우지 않는다. |
| TINYTEXT(n)   | 문자열 데이터 타입(최대 255byte)                             |
| TEXT(n)       | 문자열 데이터 타입(최대 65535byte)                           |
| MEDIUMTEXT(n) | 문자열 데이터 타입(최대 16777215byte)                        |
| LONGTEXT(n)   | 문자열 데이터 타입(최대 4294967295byte)                      |
| JSON          | JSON 문자열 데이터 타입 - JSON 형태의 포맷을 꼭 준수해야 한다. |



### 2) 숫자형 데이터타입

| 데이터 유형         | 정의                                                         |
| ------------------- | ------------------------------------------------------------ |
| TINYINT(n)          | 정수형 데이터 타입(1byte) -128 ~ +127 또는 0 ~ 255수 표현할 수 있다. |
| SMALLINT(n)         | 정수형 데이터 타입(2byte) -32768 ~ 32767 또는 0 ~ 65536수 표현할 수 있다. |
| MEDIUMINT(n)        | 정수형 데이터 타입(3byte) -8388608 ~ +8388607 또는 0 ~ 16777215수 표현할 수 있다. |
| INT(n)              | 정수형 데이터 타입(4byte) -2147483648 ~ +2147483647 또는 0 ~ 4294967295수 표현할 수 있다. |
| BIGINT(n)           | 정수형 데이터 타입(8byte) - 무제한 수 표현할 수 있다.        |
| FLOAT(길이, 소수)   | 부동 소수형 데이터 타입(4byte) -고정 소수점을 사용 형태이다. |
| DECIMAL(길이, 소수) | 고정 소수형 데이터 타입고정(길이+1byte) -소수점을 사용 형태이다. |
| DOUBLE(길이, 소수)  | 부동 소수형 데이터 타입(8byte) -DOUBLE을 문자열로 저장한다.  |



### 3) 날짜형 데이터타입

| 데이터 유형 | 정의                                                         |
| ----------- | ------------------------------------------------------------ |
| DATE        | 날짜(년도, 월, 일) 형태의 기간 표현 데이터 타입(3byte)       |
| TIME        | 시간(시, 분, 초) 형태의 기간 표현 데이터 타입(3byte)         |
| DATETIME    | 날짜와 시간 형태의 기간 표현 데이터 타입(8byte)              |
| TIMESTAMP   | 날짜와 시간 형태의 기간 표현 데이터 타입(4byte) -시스템 변경 시 자동으로 그 날짜와 시간이 저장된다. |
| YEAR        | 년도 표현 데이터 타입(1byte)                                 |

- [자료출처](http://www.incodom.kr/DB_-_%EB%8D%B0%EC%9D%B4%ED%84%B0_%ED%83%80%EC%9E%85/MYSQL#h_481d7ea17f52ffb7672ef90a025c3913)







- 마찬가지로 민감한 정보들은 secret.properties로 따로 빼고 `.gitignore`에도 추가해둠.

  ```properties
  [application.properties]
  spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
  spring.profiles.include=secret <- 이 부분
  
  spring.jpa.show-sql=true
  spring.jpa.database=mysql
  ```

  ```properties
  [application-secret.properties]
  spring.datasource.url= 내 db주소
  spring.datasource.username= 내계정
  spring.datasource.password= 내계정
  ```

  



# 3. 추가 디펜던시 작업

```properties
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa' <- jpa 사용
	implementation 'org.springframework.boot:spring-boot-starter-jdbc' <- jdbc는 일단 미정..
	implementation 'mysql:mysql-connector-java' <- mysql 사용
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testAnnotationProcessor 'org.projectlombok:lombok' <- 테스트에도 사용하기 위함
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	runtimeOnly 'mysql:mysql-connector-java' <- mysql 사용
}
```





# 4. 느낀점

- 처음으로 어떤 도움 없이 검색해가면서 세팅해본 첫 프로젝트다. 덕지덕지 붙은 디팬던시들도 있을 것 같고, 개발 진행되면서 추가로 더 필요한 설정들도 많이 있을 것 같다.
- 이렇게 한 땀 한 땀 개발하니 한 세월일 것 같지만,,, 다음번에는 좀 더 빠르게 작업하면 되니, 그것을 목표로 열심히 부딪혀봐야겠다.
- 덤벼라 오류야. 다 맞아 주겠다.



------

> 마지막 수정일시: 2022-09-13 02:35