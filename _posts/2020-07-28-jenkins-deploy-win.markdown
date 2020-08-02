---
layout: post
title: "윈도우 Jenkins에서 스프링부트 서비스 배포하기"
date: 2020-07-28 14:30:00 +0300
categories: DevOps
tags: Jenkins
comments: 1
---
Windows에서 Jenkins를 이용해 CI/CD환경을 구성하면서, 여러 삽질을 경험했다. 해당 삽질을 통해 학습한 내용을 정리하려고 한다. Windows는 Windows 10 Enterprise환경을 대상으로 했다. 또한 빌드시스템은 gradle 기준으로 작성했다.

##### Docker로 Springboot 서비스 배포하기
1. Docker for Windows 설치
    - Win7 이하에서는 사용불가
    - Hyper-V 설정을 해줘야 함. 해당 설치 방법은 검색 참고
2. DockerFile 작성
    - 검색 참조
    - 샘플
        ```
        FROM openjdk:8-jdk-alpine
        VOLUME /tmp
        COPY build/libs/*.jar spring-server.jar
        ENTRYPOINT ["java","-jar","/spring-server.jar"]
        ```
3. docker-compose.yml 작성
    - 검색 참조
    - 샘플
        ```
        version: '2' # specify docker-compose version
 
        # Define the services/containers to be run
        services:
        front-end: # name of the first service
            build: front-end # specify the directory of the Dockerfile
            ports:
            - "4200:4200" # specify port forewarding
        
        back-end: #name of the second service
            build: spring-server # specify the directory of the Dockerfile
            ports:
            - "8080:8080" #specify ports forewarding
        ```
4. 빌드 : `$gradlew.bat clean build`
5. 기존 서비스 kill : `$docker-compose down`
    - 무중단 배포를 고려하는 경우, Kubernate도입 등의 고려가 필요함
6. 새 서비스 docker image 빌드 및 실행 : `$docker-compose up -d --build --force-recreate`
    - -d : detached 모드 옵션. 서비스가 backgroud에서 실행됨. __서비스 실행 이후 Jenkins job이 종료되기 위해 필요함.__ 해당 옵션을 넣지 않으면, job이 끝나지 않음
    - --build : 컨테이너 실행 전 docker image 빌드
    - --force-recreate : docker config나 image변경이 없어도 강제로 다시 컨테이너 생성

##### Jar로 Springboot 서비스 배포하기
1. Jenkins 설정 변경
    - java로 jenkins를 실행한 경우
        - jenkins 실행 시, -Dhudson.util.ProcessTree.disable=true 옵션 추가
        - `$java -Dhudson.util.ProcessTree.disable=true -jar jenkins.war`
    - msi파일로 인스톨한 경우
        - jenkins 설치폴더 내 jenkins.xml 설정 변경
        - service-argument태그에 -Dhudson.util.ProcessTree.disable=true 옵션 추가
            ```
            <service>
                <arguments>-Dhudson.util.ProcessTree.disable=true -jar ...</arguments>
            </service>
            ```
2. Jenkins 재수행
3. 기존 서비스 kill : `$for /f "tokens=5" %%a in ('netstat -aon ^| find ":8080" ^| find "LISTENING"') do taskkill /f /pid %%a`
    - 8080포트의 프로세스를 찾아 프로세스 종료
    - 프로세스가 2개(0.0.0.0:8080, []:8080) 찾아지는 경우, 앞 서비스를 kill하고 두번째 것을 kill하려고 하면, 해당 프로세스가 없다고 에러를 출력하기도 함. 이 경우에는 "0.:8080"으로 find 검색어를 바꿔서 실행
4. 빌드 : `$gradlew.bat clean build bootJar`
5. 새로운 서비스 샐행 : `$start /min java -jar spring-server\build\libs\spring-server-0.0.1-SNAPSHOT.jar`
    - start /min : 해당 명령어를 background에서 실행됨. __서비스 실행 이후 Jenkins job이 종료되기 위해 필요함.__
    - JAVA_HOME등의 환경변수 이용 시, Program Files같이 공백이 포함된 경로때문에 syntax에러나는 등의 짜증나는 상황이 발생할 수 있으므로, 유의해야 함
