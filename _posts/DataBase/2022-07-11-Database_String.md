---
title: "Database 오류 - UTF-8"
layout: single
categories: DataBase
typora-root-url: ..\..\images\
toc: true
---

## 1. 문제

: table에 String 값을 Insert 하려고 하니 오류 발생. 

<u>Encoding - UTF 8 문제였음</u>(진짜 어딜가나 튀어나오네)

![db-string 오류](..\..\images\db-string 오류.PNG)

------

## 2. 해결

### 1) 테이블 속성 변경

![db-string 오류-해결1](..\..\images\db-string 오류-해결1.PNG)

### 2) Column 속성 변경

![db-string 오류-해결0](..\..\images\db-string 오류-해결0-165754775662260.PNG)

------



## 3. 추가 탐구

: 매번 이렇게 바꿀 수 없다. 근본적인 해결방법이 필요하다.

=> 확인해 보았으나, DBeaver에서 설정으로 UTF-8을 잡아주긴 어려워 보인다. 대신, Create table 시 설정을 잡아주면 가능!!

```sql
[테이블 생성 시 마지막에 추가]

create table zerobase_member (
    name varchar(20),
    email varchar(100),
    mobile_no varchar(12),
    password varchar(50),
    marketing_yn bit,
    register_date datetime
) ENGINE=InnoDB DEFAULT CHARSET=utf8
or
ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
```

=> 쿼리를 통해 설정변경도 가능!!

```sql
[디비/테이블/컬럼 변경]

ALTER DATABASE {DB명} CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
ALTER TABLE {테이블명} CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE {테이블명} modify  {속성명} VARCHAR(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

------

> 마지막 수정일시: 2022-07-11 23:17