---
title: "도커를 활용하여 AWS에 Springboot 띄우기"
layout: single
categories: Docker
typora-root-url: ..\..\images\
toc: true
---

- 참고할 이전 글
  1. [도커 기본](https://jiyongyoon.github.io/docker/docker/)
  2. [AWS - EC2](https://jiyongyoon.github.io/server/aws-ec2)

------

# 1. jar파일 패키징

## 1. gradle

- gradle 매뉴 - build - `bootJar` 실행

  ![image-20221024031754248](..\..\images\image-20221024031754248.png)

- project 디렉토리의 `/build/libs`에 jar파일 생성

  ![image-20221024031742396](..\..\images\image-20221024031742396.png)

## 2. maven

```shell
mvn package
```

명령어 실행하면 `target/` 위치에 jar파일 생성됨.



# 2. Dockerfile 작성

```dockerfile
### base가 되는 이미지
FROM openjdk:8-jre

### 위치: 컨테이너 밖, 컨테이너 안
### 버전은 계속 바뀔 것이기 때문에 프로젝트명 뒤에 * 사용하면 편리함
COPY build/libs/inandout-*.jar app.jar

### 컨테이너 시작시 실행할 스크립트
ENTRYPOINT ["java", "-jar", "app.jar"]

```

- Dockerfile의 위치는 루트(컨벤션)





# 3. 이미지 생성 및 로컬에서 실행해보기

```shell
docker build (-f Dockerfile 이름) -t (이미지 이름) .(도커 파일로의 상대경로)
```

```shell
docker run -p 8080:8080 이미지명
```

부트가 실행되면 로컬에서도 도커 이미지를 활용하여 부트에 접속할 수 있음.





# 4. 이미지 Hub에 push

- DockerHub(hub.docker.com) 에 내 Repository 만들기

  - 파일명은 `계정이름`이 자동으로 앞에 붙고 `/내가 만들 Repo이름`이 붙음.
  - public으로 해야 로그인 없이  접근 가능

  ```shell
  ### 테그 변경 명령어
  docker tag local-image:tagname new-repo:tagname
  
  ### hub에 push 명령어
  docker push Repo이름:tagname ### Repo이름은 계정이름/만든Repo이름이 전체가 들어가야함
  ```

  



# 5. 이미지 pull 및 실행

1. AWS에 접속 후 Docker 설치

2. 도커 로그인 (모든 명령어는 작동이 안되는 경우 sudo로 관리자 권한 부여)

   ```shell
   docker login
   ```

   - 아이디, 비밀번호 입력하면 로그인 됨

3. 도커 이미지 pull

   ```shell
   docker pull Repo이름:tagname
   ```

4. 도커 이미지 실행

   ```shell
   docker run (-d) -p 8080:8080 --name 컨테이너이름(처음 실행 시 붙여줄 이름) 이미지명 (-d는 백그라운드 실행)
   ```

   

------

> 마지막 수정일시: 2022-10-24 03:41