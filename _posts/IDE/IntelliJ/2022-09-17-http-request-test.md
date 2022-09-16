---
layout: single
title: "[IntelliJ] .http파일로 Request 보내기"
categories: IDE
typora-root-url: ..\..\images\
toc: true
---



# [IntelliJ] .http파일로 Request 보내기



수강한는 강의에서 POSTMAN을 사용하며 테스트를 했지만, 웬걸 POSTMAN이 잘 작동하지 않았다.

그냥 IntelliJ에서 해보자.



------



## `.http`파일 생성

- 프로젝트 우클릭 -> New -> HTTP Request 생성![image-20220917021225646](..\..\images\image-20220917021225646.png)



## `.http` Request 구성

![image-20220917021420641](..\..\images\image-20220917021420641.png)

- `###` -> 메서드명처럼 활용하여 Service 콘솔에 해당 내용을 출력하여 어떤 테스트인지 확인 가능.
- `POST` -> 요청 보낼 방식. 그 뒤에 바로 주소.
- 그 아랫줄은 HTTP요청의 `HEADER`. Key: Value 형태.
- 한 줄 띄고 나서는 HTTP요청의 `BODY`



------

> 마지막 수정일시: 2022-09-17 02:16