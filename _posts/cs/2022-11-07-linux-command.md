---
title: "자주쓰는 리눅스 명령어"
layout: single
categories: cs
typora-root-url: ..\..\images\
toc: true
---

# 1. 리눅스 기본 명령어

```shell
# 디렉토리 상태
dir

# 이동
cd 디렉토리 이름

# 서비스 상태
service 서비스명 status

# 서비스 시작, 종료
service 서비스명 start / stop

# 메모리 사용량 확인
free -m
free -h (용량 단위가 나와서 확인이 편함)

# 프로세스 찾기
ps -ef (전체)
ps -ef | grep 프로세스이름

# 프로세스 죽이기
kill -9 프로세스ID

# 포트 및 서비스 확인
netstat -l (모든 수신 소켓 표시)
netstat -a(사용중인 모든 포트)
netstat -t (TCP 연결 표시)
netstat -p (port 표시)
netstat -n(서비스 이름대신 port번호 표시)
netstat -ltup 이런식으로 합쳐서 사용가능

# 포트 및 서비스 확인 2
lsof -i:포트번호
```

# 2. java 관련

```shell
# 리눅스에서 jar파일 실행
java -jar jar파일명

# 백그라운드로 실행
nohup java -jar jar파일명 &
-> 이렇게 실행하면 실행한 리눅스 dir에 nohup.out 파일이 생성됨.
```





# 3. vi 명령어

```shell
# 취소
esc

# 입력모드
i -> 현재 커서 위치
a -> 현재 커서 다음 위치

# 삭제모드
x -> 커서가 위치한 곳 1글자 삭제
dd -> 커서가 위치한 곳 한 줄 삭제

# 명령취소
u -> 방금 한 명령 취소

# 줄 복사 붙여넣기
yy -> 한 줄 복사
p -> 커서 아래줄에 붙여넣기

# 이동
G(대문자) -> 파일의 끝으로 이동
$ -> 줄의 맨 뒤로 이동

# 마지막 행 모드 (저장, 종료 관련)
:w -> 파일 저장
:w[파일명] -> 파일명 설정하여 저장
:q -> 저장하지 않고 vi 종료
:q! -> 저장하지 않고 vi 강제 종료
:wq -> 저장 후 vi 종료
:wq! -> 강제 저장 후 vi 종료
:set nu -> 몇 번째 줄인지 출력
```



------

> 마지막 수정일시: 2022-11-07 10:48