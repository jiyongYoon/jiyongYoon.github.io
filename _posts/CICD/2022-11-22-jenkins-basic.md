---
title: "젠킨스(Jenkins) 기본"
layout: single
categories: CICD
typora-root-url: ..\..\images\
toc: true
---



# 1. CI/CD란?

- Continuous Integration: 지속적인 통합. (뭘? 작업물을, 즉 코드를)
- Coutinuous Delivery(Deployment): 지속적인 배달(배포). (뭘? 제품, 서비스를, 즉 어플리케이션을)
- 즉, 개발 환경과 개발 내용을 사용 가능한 서비스로 지속적이고 자동적으로 전달(빌드, 테스트, 배포)하는 과정



# 2. Jenkins?

- `Java Runtime Environment` 위에서 동작하는 자동화 서버. 빌드, 테스트, 배포 등의 것을 자동화 해주는 자동화 서버.

- 설치하여 url로 접속하면 UI화면이 나와 설정 및 관리가 편함!!

- 다양한 플러그인을 통합하여 자동화 하는 원리.

  - `Git Plugin` 
    - 깃헙에서 소스코드를 가지고 올때 사용함.
  - `PipeLine` 
    - 일련의 자동화 작업의 순서들의 집합인 `PipeLine`을 통해서 자동화함. Jenkins의 핵심.
    - 좌측 메뉴바의 `새로운 Item` - `Pipeline`을 누르면 새로 파이프라인을 만들 수 있음.
  - `Credentials Plugin`
    - 배포에 필요한 각종 리소스(클라우드 리소스, ssh 키, AWS token, Git Access token 등)를 저장하고 관리해주는 플러그인
    - 좌측 메뉴바의 `Credentials` - `System` - `Global credentials`
      - 추가를 희망하면 `Add Credentials`
      - 종류(Kind)를 선택할 수 있으며, properties처럼 값을 넘겨주기 위해서는 `Secret text`를 통해서 `Secret`에는 값을 넣고, `ID`에는 Jenkins Job Config에서 보여줄 이름을 등록함.
  - `Docker Plugin`
    - 도커를 사용하기 위한 플러그인.
  
  

# 3. PipeLine?

- 파이프라인은 결국 플러그인들로 구성된 작업통로. 파이프라인을 통과하여 내가 원하는 작업물을 만들게 됨.

- Pipeline을 동작시키기 위해서는 '레시피'를 작성하게 되는데, 이를 작성할 때 사용하는 문법 중, 가독성이 좋고 최신의 것은 `Declarative Pipeline syntax` (또 하나는 `Scripted Pipeline`)

  ![image-20221122152524472](..\..\images\image-20221122152524472.png)

  

  ## - Stages Section

  - 가장 큰 카테고리. 어떤 일들을 처리할 것인지 정의하는 곳.

  ### - Agent Section

  - 어떤 slave node에 어떤 일을 하게 할 것인지 정의.

  ### - Post Section

  - 각 작업 순서마다 일을 끝낸 후 어디로 가야할지 후속 조취를 정의. 
  - ex) 성공 시 로그를 찍어라. 슬랙에 노티스해라. 실패 시 어디로 알려라. 건너뛰어라. 등등

  ### - Stage Section

  - 어떤 일들을 처리할 것인지 정의

  ### - Declaratives

  - Environment, Parameter, Triggers, When 등

  ### - Steps Section

  - 한 스테이지 안에서 단계로 작업할 내용들

  

  ## [ pipeline 예시 ]

  ```bash
  pipeline {
       agent any # 일하게 할 slave
       stages {
       	stage('prepare') { # 업무
       		steps { # 단계 작업
       			git ~~
       			
       		}
       		post {
                	echo "prepare success"
                }
       	}
       
           stage('build step') { 
                steps {
                   echo "Build stage is running"
                }
                post {
                	echo "build success"
                }
           }
           
           stage('Only for production') {
           	when { # Declaratives
           		branch 'production'
           		environment name: 'APP_ENV', value: 'prod'
           		anyOf {
           			environment name: 'DEPLOY_TO', value: 'production'
           		}
           	}
           }
       }
  }
  ```




- 참고자료 및 관련링크
  - [토크ON세미나](https://youtu.be/GOLHN3FHjpI)
  - [젠킨스로 빌드-배포하기](https://velog.io/@junho5336/jenkins%EB%A1%9C-%EB%B9%8C%EB%93%9C-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0#jenkins-%EC%8B%9C%EC%8A%A4%ED%85%9C%EC%84%A4%EC%A0%95)
  - [젠킨스 파이프라인을 활용한 배포 자동화](https://velog.io/@sihyung92/%EC%9A%B0%EC%A0%A0%EA%B5%AC2%ED%8E%B8-%EC%A0%A0%ED%82%A8%EC%8A%A4-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8%EC%9D%84-%ED%99%9C%EC%9A%A9%ED%95%9C-%EB%B0%B0%ED%8F%AC-%EC%9E%90%EB%8F%99%ED%99%94#%EB%B9%8C%EB%93%9C-%EC%9C%A0%EB%B0%9C%ED%95%98%EA%B8%B0)
  - [백엔드 Jenkins CI/CD 구축기](https://seongwon.dev/DevOps/20220717-CICD%EA%B5%AC%EC%B6%95%EA%B8%B02/)
  - [외부EC2연결](https://rbsks.tistory.com/9)
  - [도커 이미지 빌드관련1](https://velog.io/@imsooyeon/Jenkins-pipeline%EC%9D%84-%EA%B5%AC%EC%B6%95%ED%95%98%EC%97%AC-Docker-build-%EB%B0%8F-%EC%9D%B4%EB%AF%B8%EC%A7%80-push-%ED%95%98%EA%B8%B0)
  - [도커 이미지 빌드관련2](https://hyeinisfree.tistory.com/23)
  - 향로 블로그
    - [도커를 활용한 젠킨스 설치 및 세팅](https://jojoldu.tistory.com/139)
    - [젠킨스와 Github ssh 연동](https://jojoldu.tistory.com/442)
  - [젠킨스-슬랙설정-공식안내](https://w1661913672-14q788704.slack.com/services/B04CU2HR6GZ?added=1)
  - [젠킨스-슬랙설정-블로그](https://junhyunny.github.io/information/jenkins/jenkins-slack-notification/)
  - [젠킨스-깃헙웹훅연동](https://cokes.tistory.com/119)
  - [젠킨스-깃헙웹훅연동(앱사용)](https://fwani.tistory.com/23)
  - [젠킨스-깃헙웹훅연동(젠킨스파일사용)](https://www.youtube.com/watch?v=oFnS4dDzCD0&t=1054)

------

> 마지막 수정일시: 2022-11-26 04:33