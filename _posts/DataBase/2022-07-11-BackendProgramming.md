---
title: "백엔드 프로그래밍과 Database"
layout: single
categories: DataBase
typora-root-url: ..\..\images\
---

## 1. 백엔드 프로그래밍

<img src="..\..\images\backend.png" alt="backend" style="zoom: 50%;" />

웹 브라우저를 넘어 네트워크를 타고 웹서버에 요청한 이후부터 처리하는 과정 및 프로그램. <br>백엔드 프로그램 언어를 통해 영구적인 저장공간(DB)에 주어진 조건 및 로직에 맡게 데이터에 대해서 쓰기, 읽기, 수정하기, 삭제하기 작업을 하는 일.

- 웹 브라우저: HTML, CSS, Javascript를 통해 사용자에게 페이지를 보여주는 역할. 해당 데이터들은 HTTP 프로토콜을 통해 웹서버에 전달된다.
- 웹 서버: JSP(Java Server Page), PHP, ASP.Net 등으로 브라우저와 소통하여 데이터를 핸들링한다. 사용자로부터 온 데이터를 정제하여 Database에 넘겨주기도 하고, 사용자가 요청한 자료를 Database로부터 가져와 브라우저에 전달하기도 한다.
- Database: DB Query(SQL문)를 사용하여 핸들링한다. 데이터의 가장 기본적인 저장공간이다.



## 2. DBMS와 유형 및 종류

- DBMS: 데이터베이스를 운영하고 관리하는 소프트웨어
- DBMS 유형: 게층형(ex, 가계도, 조직도), 망형(ex, 업무 프로세스), 관계형(ex, 회원정보), 객체 지향형 등. 
  - 관계형 데이터베이스: 가장 많이 사용되는 데이터 베이스 종류. Table(엑셀의 Sheet와 비슷한 개념)로 구성되어 있고, Table은 행(row)과 열(column)으로 구성됨. 한 행(row)가 하나의 레코드이며, 한 열(column)이 하나의 속성.
- DBMS 소프트웨어: 오라클, MySQL, MariaDB, MsSQL 등이 대표적임.

------

> 마지막 수정일시: 2022-07-11 12:48