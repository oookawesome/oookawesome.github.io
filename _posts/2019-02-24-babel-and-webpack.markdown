---
layout: post
title: "Babel과 Webpack, Jest 설정 방법"
date: 2019-02-24 14:30:00 +0300
categories: Frontend
tags: Babel Webpack Jest
---

지금까지 babel이나 webpack을 누군가 설정해놓은 환경에서 개발을 해왔다. 샘플 프로젝트를 처음부터 만들어 보면서, babel과 webpack을 설정해 보았고, 단위 테스트를 위한 Jest설정을 추가했다. 해당 내용을 정리해보겠다.

해당 내용은 Babel 7, webpack 4 버전 기준으로 작성되었다. Babel의 경우 7버전과 그 이하 버전에서 설정법이 차이가 있다.

1. npm 설정
    1. `$cd project-dir`
    2. `$npm init`
    3. `package.json 설정`

2. npm install
    1. `$npm install @babel/core --save-dev`
    2. `$npm install webpack --save-dev`

3. babel 설정
    1. `$npm install @babel/preset-env --save-dev`
    2. `프로젝트 루트에 .babelrc 파일 생성`
    3. `.babelrc 파일에 해당 설정 추가`
        ```
        {
          "presets": [
            [
              "@babel/preset-env",
              {
                "targets": {
                  "node": "current"
                }
              }
            ]
          ]
        }
        ```

4. Webpack 설정
    1. `$npm install webpack-cli --save-dev`
    2. `번들링 시 babel을 사용하기 위해 babel-loader 설치 : `
        `$npm install babel-loader --save-dev`
    3. `프로젝트 루트에 webpack.config.js 파일 생성`
    4. `webpack.config.js에 설정 추가`
        ```
        module.exports = {
          target: "node",
          module: {
            rules: [
              {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                  loader: "babel-loader"
                }
              }
            ]
          }
        }
        ```

5. Jest 설정
    1. `$npm install jest babel-jest regenerator-runtime --save-dev`
    2. `src와 같은 구조로 test 디렉토리 구성`
    3. `sample.test.js와 같이, 테스트 대상 파일명 뒤에 .test.js를 붙여서 테스트 파일 생성`

6. package.json scripts 수정
    1. `start : npm run build && node dist/main.js (webpack.config.js에서 따로 설정하지 않으면, main.js로 번들링됨)`
    2. `build : webpack --mode development`
    3. `prod : webpack --mode production`
    4. `test : jest --detectOpenHandles`

-----
출처
- https://wkdtjsgur100.github.io/react=jest/
- https://poiemaweb.com/es6-babel
