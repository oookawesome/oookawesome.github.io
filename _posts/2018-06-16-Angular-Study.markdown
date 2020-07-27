---
layout: post
title: "AngularJS 개념과 특징 그리고 동작방식"
date: 2018-06-16 17:30:00 +0300
categories: Frontend
tags: Angular
---

##### 시작
새로운 프로젝트에서 개발업무를 시작하게 되었다. 프로젝트 기술스택에 대한 지식과 개념이 너무나 부족하고 정리가 안되있어, 본격적인 기술스택 학습에 앞서 사전 학습을 하게 되었다. [웹 기술에 대한 이전 스터디](/How-To-Work-Web)도 있다.

##### AngularJS
AngularJS는 자바스크립트 기반의 프론트엔드 웹 애플리케이션 프레임워크이다. Client사이드의 MVC구조를 제공한다.

특징
- 싱글 페이지 애플리케이션(SPA) 프레임워크이다.
  - SPA : 서버로부터 새로운 페이지를 불러오는 것이 아닌, 현재 페이지에 새로운 view를 동적으로 다시 로드하는 애플리케이션
- MVC/MVVM 패턴이 적용된다.
- Model-View 간 양방향 데이터 바인딩이 가능하다. Model이 바뀌면 View 데이터도 같이 변경된다. 반대도 마찬가지.
- 의존 주입(Dependency Injection) 활용

##### MVC 모델
![angular-mvc](../../../assets/postImages/angular-mvc.png){: width="50%" height="50%"}

Model
- 데이터를 정의하는 부분으로, 변형되지 않는 자바 스크립트 객체로 구성
- 일반 MVC구조에서의 DTO 역할
- HTML속성 : ng-model

View
- 사용자가 사용하는 환경
- 템플릿과 모델이 합쳐져 보여지며 DOM구조

Controller
- 자바스크립트로 이루어진 제어로직
- 실질적으로 데이터에 대한 처리와 업데이트를 처리. Model에서 데이터를 가져와 서비스작업을 처리하고, 처리 데이터를 다시 View에 갱신

##### AngularJS의 특별한 개념들

Scope
- 특정 DOM 영역을 위한 모델
- Model과 View를 감시하고, Controller에 이벤트를 보내는 역할
- DOM과 가까운 계층구조를 가짐

Directive
- html을 확장하는 AngularJS의 지시어 (ex. ng-app, ng-model...)
- 사용자 정의 가능

양방향 데이터 바인딩
- Model과 View의 데이터를 실시간으로 연동

Template
- HTML자체를 템플릿으로 사용
- 지시어(Directive), 표현식, 필터 등을 포함함

Dependency Injection
![angular-di](../../../assets/postImages/angular-di.png){: width="50%" height="50%"}
- Injector : Dependency Injection을 담는 컨테이너 역할

Module
![angular-module](../../../assets/postImages/angular-module.png){: width="50%" height="50%"}
- 애플리케이션의 각기 다른 기능을 나타내는 컨테이너(Controller, Filter, Service, Directive 등을 포함)

Service
- 특정 기능을 담당하는 객체
- 싱글톤 객체로 하나의 인스턴스만 존재
- \$를 앞에 붙여서 표기함 (ex. \$compile, \$window)

Expression
- {% raw %}{{}}{% endraw %}로 사용됨 (표현식 내에 자바스크립트 문법을 사용)
- {% raw %}ex. {{ name }}, {{ 3%5 }}, {{"안녕" + "!"}}{% endraw %}

##### 부트스트랩 (Initialize)
![angular-start-up](../../../assets/postImages/angular-concepts-startup.png){: width="50%" height="50%"}
1. Browser가 HTML을 DOM으로 파싱
2. DOM이 전부 로딩된 상태(DOMContentLoaded)가 되면 자동으로 AngularJS 초기화 시작
3. 애플리케이션의 root를 나타내는 ng-app directive를 찾는다.
4. 관련있는 module을 로드
5. DI를 위한 \$injector를 생성
6. injector 서비스는 \$compile, \$rootScope를 생성
7. $compile을 통해 ng-app을 루트로하는 하위DOM을 컴파일하고, rootScope와 링크

------
##### 참고
- [AngularJS Developer  가이드](https://docs.angularjs.org/guide)
- <http://ithub.tistory.com/68>
- <http://heiswed.tistory.com/entry/AngularJS-%EC%86%8C%EA%B0%9C>
- <http://webclub.tistory.com/206>
- <http://mobicon.tistory.com/270>
- <http://webframeworks.kr/tutorials/angularjs/app_structure_with_module/>
