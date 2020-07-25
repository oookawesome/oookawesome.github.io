---
layout: post
title: "Angular 학습 일지"
date: 2018-07-07 18:30:00 +0300
categories: Frontend
tags: Angular
---

### Angular 학습 일지

#### 프롤로그
Angular학습을 위해 "러닝! Angular 4" 책을 읽었다. 해당 내용을 장별로 정리했다.

#### 3장 앵귤러 시작
- angular CLI 명령을 통해 컴포넌트 등을 쉽게 생성할 수 있다.
- $ng generate component {name} 치면 html, scss, tc, spec.tc 등을 바로 만들어준다
- @NgModule 데코레이터는 특정 모듈(우리 경우에는 우리앱)의 모든 임포트, 선언 등을 한곳에서 관리하는 역할을 한다.
  - provider : injector가 받는 주입받는 서비스들
  - declaration : 우리 모듈에 속하는 다른 컴포넌트들
  - imports : 라이브러리와 같이, 우리 모듈에 속하지 않는 다른 컴포넌트들
  - bootstrap : 부트스트랩 컴포넌트 (메인 함수 역할)

#### 4장 컴포넌트
- component 생성하면 @Component 붙어있는 ts파일 생성됨
  - selector : HTML사용시 해당 컴포넌트가 가지게될 이름(태그명)
  - template : 해당 컴포넌트 모양을 인라인 HTML형식으로 지정
  - templateUrl : 해당 컴포넌트의 모양을 외부 템플릿으로 지정
  - styles : 해당 컴포넌트의 디자인을 인라인 형식으로 지정
  - styleUrls : 해당 컴포넌트의 디자인을 외부 CSS파일로 지정
- 초기 데이터 설정 시 생성자 사용
- @Input 이용해 외부에서 컴포넌트 내부로 데이터를 전달받는 것을 정의할 수 있음

#### 5장 표현식
- 앵귤러는 HTML 파일 내에서 자바 스크립트 표현식을 사용할 수 있음
- {% raw %}{{}}{% endraw %} 형식으로 표현식 사용
  - 내부에 +,-,=== 같은 연산자 사용가능
  - 컴포넌트 내부의 변수와 함수를 참조하는 것이 가능
  - 타입 스크립트 사용가능
- 파이프(\|) 사용
  - 파이프는 데이터의 표현 형식을 수정하는 연산자
  - 내장 파이프를 제공함 (json, date...)
  - ex. {% raw %}{{{% endraw %}expression \| json{% raw %}}}{% endraw %}
  - @Pipe 이용해 사용자정의 파이프 구현 가능

#### 6장 데이터 바인딩
- 데이터 바인딩은 애플리케이션의 데이터를 UI요소와 연결하는 것. 모델에서 데이터가 변경되면, 웹 페이지가 자동으로 업데이트 된다
  - 삽입식 : 이중중괄호({% raw %}{{}}{% endraw %})를 사용해 표현식을 작성
    {% raw %}
    ~~~~
    <img src = "{{imgUrl}}" />
    ~~~~
    {% endraw %}
  - 프로퍼티 바인딩 : HTML 요소의 프로퍼티를 설정해야 할 때 프로퍼티 바인딩 사용  
    {% raw %}
    ~~~~
    <img [src]="myValue">
    ~~~~
    {% endraw %}
  - 클래스 바인딩 : CSS 스타일 태그를 HTML 요소에 바인딩할 때 사용. 표현식의 결과가 true인 경우에 클래스가 할당됨  
    {% raw %}
    ~~~~
    <div [class.nameHere]="true">
    ~~~~
    {% endraw %}
  - 스타일 바인딩 : HTML 요소에 인라인 스타일을 지정할 때 사용. 클래스 바인딩과 유사하지만, 접두어로 style이 붙음  
  {% raw %}
  ~~~~
  <p [style.styleProperty] = "assignment">
  ~~~~
  {% endraw %}
  - 이벤트 바인딩 : 클릭, 키 입력, 마우스 이동과 같은 사용자 입력을 처리할 때 사용. HTML 이벤트 속성에서 on 접두사가 제거되고, 이벤트를 괄호(())로 감쌈  
  {% raw %}
  ~~~~
  <button (click)="myFunction()">
  ~~~~
  {% endraw %}
- 양방향 바인딩 : ngModel을 사용해 변경사항을 감지하고, 값을 업데이트 함  
{% raw %}
~~~~
<input [(ngModel)] = "myValue">
~~~~
{% endraw %}

#### 7장 내장 디렉티브
- 디렉티브는 템플릿 데이터와 HTML요소의 동작방식을 정의
- 앵귤러 내장 디렉티브
  - ngFor : iterable객체 내의 각 항목에 대한 사본 생성  
  {% raw %}
  ~~~~
  <div *ngFor="let person of people">
  ~~~~
  {% endraw %}
  - ngIf : 해당 값이 true인 경우에만 DOM에 추가, false이면 DOM에서 제거됨  
  {% raw %}
  ~~~~
  <div *ngIf="person">
  ~~~~
  {% endraw %}
  - ngSwitch/ngSwitchCase/ngSwitchDefault : switch문을 구성함. 값이 아무것도 일치하지 않으면 DOM 생성하지 않음  
  {% raw %}
  ~~~~
  <div [ngSwitch]="timeOfDay">
    <span [ngSwitchCase]="'morning'">Morning</span>
    <span [ngSwitchCase]="'afternoon'">Afternoon</span>
    <span [ngSwitchDefault]="'daytime'">Evening</span>
  ~~~~
  {% endraw %}

#### 8장 사용자 정의 디렉티브
- @Directive 애노테이션을 통해 사용자 정의 디렉티브를 구성
- @HostListener, Renderer 등 앵귤러에서 임포트한 기능을 사용하여 구성가능

#### 9장 이벤트와 변경 감지
- 이벤트 바인딩은 이벤트 이름을 괄호(())로 감싸는 식으로 작성   
{% raw %}
~~~~
<input type="text" (change)="myEventHandler($event)" />
~~~~
{% endraw %}
  - ex. (click), (change), (focus), (submit), (keyup), (keydown), (keypress), (mouseover)

- 사용자 정의 이벤트를 구성할 수 있고, 해당 이벤트를 부모 컴포넌트로 내보낼 수 있음
  - @Output 애노테이션과 EventEmitter 클래스를 사용
  {% raw %}
  ~~~~
  @Output() name: EventEmitter<any> = new EventEmitter();
  myFunction() {
    this.name.emit(args);
  }
  ~~~~
  {% endraw %}
  - 내보내진 이벤트는 내장 이벤트와 유사하게 이벤트를 처리
  {% raw %}
  ~~~~
  <div (name)="handlerMethod(event)">
  ~~~~
  {% endraw %}
- Observable 활용

#### 11장 사용자 정의 앵귤러 서비스 만들기
- 사용자 정의 서비스를 구현해 특정 기능을 제공하는 것이 가능
- 사용자 정의 서비스 구현시, 재사용 코드조각이라고 생각할 필요가 있음
- @Injectable 애노테이션을 이용해 서비스를 작성
{% raw %}
~~~~
@Injectible()
export class CustomService {}
~~~~
{% endraw %}
- 사용자 정의 서비스를 가져다 쓰는 컴포넌트의 provider 부분에 해당 서비스를 추가
{% raw %}
~~~~
import { CustomService } from './path'

@Component({
  ..
  ..
  provider: [ CustomService ]
  })
~~~~
{% endraw %}
- 사용자 정의 서비스를 사용하기 위해 해당 서비스의 인스턴스를 생성
{% raw %}
~~~~
constructor(
  private myService: CustomService
  ){}
~~~~
{% endraw %}
