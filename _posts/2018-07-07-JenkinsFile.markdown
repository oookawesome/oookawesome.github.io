---
layout: post
title: "JenkinsFile을 활용한 Jenkins Pipeline 설정방법"
date: 2018-07-07 17:30:00 +0300
categories: DevOps
tags: Jenkins
comments: 1
---

##### 프롤로그
Jenkins에 CI 구성 중에, 테스트 이후, 결과를 전송하는 Job을 만들려 했다. 하지만 Jenkins에서 테스트가 실패하는 경우, 뒷 작업을 skip하도록 되어있어, 테스트 실패시, 결과를 전송하는 것이 불가능했다. 검색결과, JenkisFile을 통해 Jenkins Pipeline을 구성하면 해결할 수 있었다. 해당 내용을 정리했다.

##### Jenkins Pipeline
Jenkins 2.x 이상에서는 build item으로 Pipeline 형식을 제공하고, Pipeline은 Groovy와 같은 DSL로 작성된 파일을 바탕으로, Code기반의 자동화 테스트 표현을 가능하게 해준다.

> JenkinsFile : Pipeline설정을 정의한 Groovy파일. 해당 파일에 Pipeline설정을 작성한다.

##### 관련 용어
- pipeline : 파이프라인은 CD Pipeline 모델을 나타낸다. 해당 부분의 코드는 전체 빌드 프로세스를 정의한다. 일반적으로 빌드, 테스트, 배포 등의 Stage를 포함하는 형식이 된다.  
(선언적 파이프라인 형식으로, script를 사용하기 위해서는 scipt영역을 지정해야 함)
- node : script 파이프라인 형식으로, script 키워드 없이 내부 스크립트 작성 가능
- stage : 빌드, 테스트 등과 같은 개념적으로 분리된 Subset을 나타낸다.
- step : Stage 내부에 위치하는 단일 Task를 의미한다. (ex. sh 'make')
- script : groovy스크립트를 사용하는 경우 script scope 내에 작성
- post : 파이프라인 실행이 종료된 이후의 작업 설정
- triggers : 파이프라인 트리거 설정 (cron, pollSCM 등의 키워드로 설정함)
- overview :
```
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                //
            }
        }
        stage('Test') {
            steps {
                //
            }
        }
        stage('Deploy') {
            steps {
                //
            }
        }
    }
    post {
      always {
        echo 'bra bra'
      }
      failure {
        echo 'fail!'
      }
    }
}
```
```
// node 키워드로 파이프라인을 정의하면, script {} 없이 if-else와 같은 스크립트 작성 가능
node {
    stage('Example') {
        if (env.BRANCH_NAME == 'master') {
            echo 'I only execute on the master branch'
        } else {
            echo 'I execute elsewhere'
        }
    }
}
```

##### script 주요 명령어
- sh : bash쉘에서 쉘 스크립트 실행 (ex. sh 'gradlew test')
- __bat : windows에서 쉘 스크립트 실행 (ex. bat 'gradlew test')__
- echo : 콘솔 출력
- sleep : 슬립 (ex. sleep 30)
- pwd : 현재 파일 디렉토리 경로 출력
- try-catch문 사용가능

##### Example (Windows에서 jenkins구동)
```
pipeline {
    agent any

    // SCM trigger 설정
    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('Kill previous instance') {
            steps {
                // 기존 포트(8093)에 떠있는 프로세스를 찾아 taskkill로 죽인다.
                bat 'set PID=0\n' +
                        'FOR /F \"tokens=5 delims= \" %%P IN (\'netstat -ano ^| findstr :8093\') DO SET /A PID=%%P\n' +
                        'IF /I \"%PID%\" GEQ \"1\" (\n' +
                        '    TaskKill /F /PID %PID%\n' +
                        ')'
                bat 'ping 127.0.0.1 -n 11 > nul'
            }
        }

        stage('Refresh gradle dependency') {
            steps {
                // gradle dependency 업데이트
                bat 'gradlew --refresh-dependencies'
            }
        }

        stage('Unit test') {
            steps {
                bat 'gradlew clean test'
            }
        }

        stage('Component test') {
            steps {
                bat 'gradlew componentTest'
            }
        }

        stage('Contract test') {
            steps {
                script {
                    String currentDirectory = pwd()
                    try {
                        bat 'gradlew contractTest'
                    }

                    catch (err) {
                        // 실패하는 경우,
                        bat 'gradlew updateContractTestResult --from=decision --reportPath=\"' + currentDirectory + '\\build\\reports\\tests\\contractTest\\packages\"'
                        error("Contract Test Failed!")
                    }
                }
            }
        }

        stage('Upload contract test result') {
            steps {
                script {
                    String currentDirectory = pwd()
                    bat 'gradlew updateContractTestResult --from=decision --reportPath=\"' + currentDirectory + '\\build\\reports\\tests\\contractTest\\packages\"'
                }
            }
        }

        stage('Build') {
            steps {
                bat 'gradlew build'
            }
        }

        stage('Start service') {
            steps {
                bat 'start java -jar build/libs/msa-mountainbook-decision-service-0.0.1-SNAPSHOT.jar'
            }
        }
    }
}
```

##### CI 설정 방법
1. 작성한 JenkinsFile.groovy 파일을 프로젝트 루트에 저장
2. Jenkins 서버 접속 > 새로운 Item > Pipeline 선택
3. Pipeline 탭 > Definition > Pipeline Script from SCM 선택
4. SCM > Git선택
5. Repository URL/Credential/Branch 설정
6. Script Path > JenkinsFile.groovy 입력
(JenkinsFile을 루트에 놓지 않은 경우, 경로 지정)

---
##### 참고
- [파이프라인 개요](https://jenkins.io/doc/book/pipeline/)
- [파이프라인 Syntax](https://jenkins.io/doc/book/pipeline/syntax/)
- [샘플 코드](https://jenkins.io/doc/pipeline/examples/)
