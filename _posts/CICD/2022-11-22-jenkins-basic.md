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

- 다양한 플러그인을 통합하여 자동화 하는 원리.

  - `Git Plugin` 
    - 깃헙에서 소스코드를 가지고 올때 사용함.
  - `PipeLine` 
    - 일련의 자동화 작업의 순서들의 집합인 `PipeLine`을 통해서 자동화함. Jenkins의 핵심.
  - `Credentials Plugin`
    - 배포에 필요한 각종 리소스(클라우드 리소스, ssh 키, AWS token, Git Access token 등)를 저장하고 관리해주는 플러그인
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

  

------

> 마지막 수정일시: 2022-11-22 23:55