---
layout: single
title: "[IntelliJ] 깃허브에 민감정보(properties) 올리지 않는 방법"
categories: IDE
typora-root-url: ..\..\images\
toc: true
---



# [IntelliJ] 깃허브에 민감정보(properties) 올리지 않는 방법



프로젝트 파일을 업로드 할 때, 생각도 못하고 비밀번호를 업로드 했다가 gmail계정이 광고메일을 보내는 것으로 사용되어 버렸다...

더 큰 일이 나기 전에 이런 경험이 있었다는 것을 다행이라고 생각하며 정신승리 후, 민감정보를 숨기는 방법을 찾아보고 적용하여 작성한다.



------



- gradle / springboot 사용 예시



1. application.properties와 같은 레벨에 application-아무이름.properties 파일을 하나 더 만든다.

   ![image-20220830212321545](..\..\images\image-20220830212321545.png)

2. 만든 파일에 민감정보(url, password 등)를 입력한다.

3. 원래 application.properties에는 불러올 파일명을 적어준다.
   ```properties
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   spring.profiles.include=secret
   ```

   

4. .gitignore에 숨길 파일을 등록한다.

   ```
   ### IntelliJ IDEA ###
   .idea
   *.iws
   *.iml
   *.ipr
   out/
   !**/src/main/**/out/
   !**/src/test/**/out/
   application-secret.properties <- 추가



------

> 마지막 수정일시: 2022-08-30 21:25