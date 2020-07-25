---
layout: post
title: "Node.js와 Electron, TypeScript 기초"
date: 2018-06-14 17:30:00 +0300
categories: Frontend
tags: Node.js Electron TypeScript
---

##### 시작
새로운 프로젝트에서 개발업무를 시작하게 되었다. 프로젝트 기술스택에 대한 지식과 개념이 너무나 부족하고 정리가 안되있어, 본격적인 기술스택 학습에 앞서 사전 학습을 하게 되었다. [웹 기술에 대한 이전 스터디](/How-To-Work-Web)도 있다.

##### Node.js
Node.js는 V8(자바스크립트 엔진) 위에서 동작하는 이벤트 처리 I/O 프레임워크이다. (V8은 웹서버에서 응답받은 자바 스크립트를 크롬에서 해석해 돌리는 역할을 한다고 보면 된다.)

###### 특징
- 이벤트 기반, 논 블로킹 I/O모델을 사용해 가볍고 효율적이다.
- Node.js 라이브러리의 모든 API는 비동기식이다.
- 단일 스레드모델을 사용한다.
- Node.js는 웹서버가 아니다. 입문자들의 가장 큰 오해 중 하나다.
- CPU 사용률이 높은 어플리케이션에서는 Node.js사용을 권장하지 않는다.
- npm : Node Package Module의 약자로, Node.js에서 사용가능한 모듈을 모아놓은 패키지 매니저
- express : Node.js상에서 동작하는 웹 개발 프레임워크. 이를 이용해 손쉽게 웹서버를 구축할 수 있다.

###### express 디렉토리 구조
패키지 설치 : `$ npm install express`
express모듈 설치 : `$ express sample`
자동 생성된 sample폴더 디렉토리 구조 :  
```
/sample
  ⌊ /bin
      ⌊ www
  ⌊ /public
      ⌊ /images
      ⌊ /javascripts
      ⌊ /stylesheets
  ⌊ /routes
      ⌊ index.js
      ⌊ users.js
  ⌊ /views
      ⌊ index.jade
  ⌊ app.js
  ⌊ package.json
```
- package.json : 프로그램 이름, 버전 등 프로그램 정보, 타 모듈에 대한 dependency 등을 작성한다. npm은 이를 참조해 필요한 모듈을 모두 설치한다.
- bin/www: 서버 구동을 위한 코드. app.js 파일을 가져와 노드의 http객체와 연동한다. http관련 설정이 들어간다.
- app.js: 어플리케이션의 메인이되는 파일이다. 서버의 설정, 미들웨어 정의, 라우트 정의 등이 설정된다.
- /routes: 라우팅을 위한 폴더. 라우팅 리소스 별로 모듈을 만들어 라우팅 로직을 각 파일에 구현.
  ```
  // routes/index.js
  var express = require('express');
  var router = express.Router();

  router.get('/', function(req, res, next) {
    res.render('index', { title: 'Express' });
    });

    module.exports = router;
  ```
- /views/index.jade: 템플릿 파일인(Jade) 파일. 이 파일 제이드 엔진을 통해 HTML 코드로 변환된다.
- /public: 정적 파일을 위한 폴더. 자바스크립트 파일, 이미지 파일, 스타일시트 등으로 구성된다.

프로그램 구동 : `$ npm start`


##### TypeScript
TypeScript는 마이크로소프트에서 개발한 자바 스크립트의 슈퍼셋 언어이다. 슈퍼셋 언어이기에 자바 스크립트로 작성된 모든 프로그램은 타입스크립트로도 동작한다.

###### 특징
- 대규모 프로젝트에 적합하다.
- 자바 스크립트에 정적 타입 개념을 추가했다. 명시적 자료형 선언이 가능하다.
- TypeScript를 컴파일하면, 자바 스크립트를 얻을 수 있다.
- 객체지향 프로그래밍 지원
- 리팩토링에 용이

###### Hello, World!
```
function greeter(word: string) {
    return "Hello, " + word;
}

let word = "World!";
let word2 = [0, 1, 2];   

document.body.innerHTML = greeter(word); // Hello, World!
document.body.innerHTML = greeter(word2); // 컴파일 에러 발생
```

##### Electron
Electron은 웹 애플리케이션의 프런트엔드와 백엔드 구성 요소를 사용하여 데스크톱 그래픽 사용자 인터페이스 애플리케이션의 개발을 가능하게 해주는 툴이다. 백엔드로는 Node.js 런타임을, 프론트엔드로는 Chromium을 사용한다.

###### 프로세스 구조
Electron의 프로세스는 메인 프로세스와 렌더러 프로세스로 구성되어있다.  

![pivotal](../assets/images/posts/electron.png){: width="100%" height="100%"}

- 메인 프로세스는 반드시 1개이며, 각각 웹 페이지마다 렌더러 프로세스가 개별적으로 존재
- 메인 프로세스는 BrowserWindow 인스턴스를 생성하여 웹 페이지를 생성
- BrowserWindow 인스턴스는 자체 렌더러 프로세스에서 웹 페이지를 실행
- 각각 렌더러 프로세스는 독립적이며, 메인 프로세스를 통해 관리
- 대부분의 Electron API는 메인 프로세스에서만 사용가능하고, 일부만 렌더러에서 사용
- Node.js의 모든 API는 Electron에서 사용 가능

------
##### 참고
- Node.js
  - <https://velopert.com/1331>
  - <http://webframeworks.kr/getstarted/expressjs/>
  - <https://velopert.com/294>
- TypeScript
  - <https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html>
