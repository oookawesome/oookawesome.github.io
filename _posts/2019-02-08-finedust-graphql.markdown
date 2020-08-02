---
layout: post
title: "GraphQL과 Apollo를 이용한 미세먼지 서버 개발"
date: 2019-02-08 14:30:00 +0300
categories: Frontend
tags: GraphQL
comments: 1
---

##### 프롤로그
지난해 GraphQL 서버 개발을 주제로한 사내 발표를 듣고 GraphQL 관심을 가지고 있었다. 거기에 2018년을 정리하며 찾아본 [The State of JavaScript 2018](https://stateofjs.com/)에서 GraphQL이 언급된 것을 보고, 관련 문서와 레퍼런스를 참조해 간단한 미세먼지 GraphQL서버를 만들어보았다.

##### GraphQL?
2015년 페이스북에서 발표한 API에 대한 데이터 질의어이다. 데이터 질의어라는 말에서 알 수 있듯이, 페이스북은 데이터를 주고 받기위한 규약을 정의해 발표한 것 뿐이고, 구현을 위한 라이브러리는 여러 언어별로 각각 개발되고 있다.

##### GraphQL vs REST
GraphQL은 기존의 REST API와는 많이 다르다.
가장 핵심은 REST는 각 리소스별로 endpoint를 생성하지만, __GraphQL은 하나의 endpoint만 생성한다는 점이다.__

예를 들어, 학생을 조회하는 API가 있다고하면, REST에서는 id별로 조회하고자 할 때, /student/1, /student/2 등과 같이, 리소스별 endpoint를 통해 조회가 이루어진다.
그러나 GraphQL에서는 오직 /graphql 하나의 단일 endpoint에서 조회가 이루어진다. __이 endpoint에 정해진 형식으로 쿼리를 보내 우리가 원하는 데이터만을 서버로부터 가져온다.__

##### Example
예를 들어 보겠다. 다음과 같이 서버에서 GraphQL스키마를 정의해 놓으면,

```
type Query {
  student(id: ID!): Student
}

type Student {
  id: ID
  name: String
  address: String
}
```
클라이언트에서는 다음과 같은 GraphQL 요청을 통해,
```
{
  student(id: "1") {
    id
    name
  }
}
```
다음과 같이 id와 name 데이터를 response로 받아올 수 있다.
```
{
  "data" : {
    "student" : {
      "id" : "1",
      "name" : "Hong Gil Dong"
    }
  }
}
```
실제 Student 스키마에는 address필드도 가지고 있으나, GraphQL쿼리를 통해 id와 name만 가져왔다!

##### Why GraphQL?
GraphQL을 페이스북에서 만든 이유를 생각해보면 GraphQL의 목표와 장점을 알 수 있다.

페이스북이 처음 GraphQL을 고안한 시기가 2012년인데, 이 때 페이스북에서 REST기반으로 모바일 앱을 개발하면서 여러 어려움을 겪었다고 한다. 안드로이드와 iOS용 앱을 개발하면서, 필요한 데이터가 서로 많이 달랐고, 그에 따라 서버에는 데이터를 준비하기 위한 코드가, 클라이언트에는 이 데이터를 파싱하기 위한 코드가 많이 들어가게 되었다고 한다.

이 때문에 GraphQL이 고안된 것이다. 샘플에서 볼 수 있듯, 필요한 데이터를 하나의 endpoint에서 가져올 수 있게 함으로써,
- 필요 이상의 데이터를 가져오는 것 (overfetching)
- 필요한 데이터를 위해 여러번 데이터를 요청하는 것 (underfetching & n+1 problem)

등의 문제를 해결하려는 것이다.

##### Sample
그럼 이제 코드에 대해 살펴보면, 기본적인 GraphQL서버의 큰 구조는 3가지 정도로 나누어 볼 수 있다.
- app : 서버 메인
  - 서버의 endpoint는 /graphql로 단 하나이다.
  ```
  var app = express();
  app.use('/graphql', graphqlHTTP({
    schema: schema,
    rootValue: resolver,
    graphiql: true,
  }));
  app.listen(4000);
  console.log('Running a GraphQL API server at localhost:4000/graphql');  
  ```
- schema : 스키마 정의
  - GraphQL의 스키마 정의문법에 따라 작성한다. (참조 : https://graphql.org/learn/schema/#type-language)
  - type Query는 쿼리 설정을 위해 반드시 설정해야 한다.
  ```
  var schema = buildSchema(`
    type Query {
      student(id: ID!): Student
    }
    type Student {
      id: ID
      name: String
      address: String
    }
  `);
  ```
- resolver : 쿼리에 대한 응답을 처리하는 부분
  ```
  var resolver = {
    student: (id) => {
      return getStudentById(id);
    }
  };
  ```

이 세 가지 구조를 바탕으로 서버를 구현하게 된다. 당연하겠지만, 주로 schema와 resolver에 신규 쿼리에 대한 내용을 추가해나가는 방식으로 개발이 진행되게 된다.

##### Apollo
미세먼지 데이터를 제공하기 위해서는 공공 OpenAPI에서 데이터를 받아와야 했다. 그래서 REST API 요청을 쉽게 할 수 있는 게 있나 찾아보다 보니 GraphQL서버, 클라이언트 구현을 위한 다양한 기능을 제공해주는 Apollo 프레임워크를 알게 됬다.

Apollo 프레임워크의 주요한 기능은 이렇다.
- REST API와 같은 다양한 데이터 소스에 대한 접근을 편리하게 해줌
- 캐싱에 대한 편의기능 제공
- 상태관리 용이함

Apollo의 여러 기능을 활용해본 것은 아니고, 튜토리얼을 보면서 REST API에 데이터 소스를 연결하는 기능만 활용해 보았다.

주요 소스코드는 이러하다.
- app.js
  - Apollo 프레임워크를 활용하므로 ApolloServer로 인스턴스를 만들고, config의 dataSources필드의 restApi설정에 내 RestAPI 소스를 연결하도록 설정한다.
  ```
  const server = new ApolloServer({
      typeDefs,
      resolvers,
      dataSources: () => ({
        restApi: new RestAPI()
      })
  });
  ```
- resolvers
  - REST호출이 있기때문에 함수는 async로 선언해야 하며, argument로 dataSources를 받는다. 서버 인스턴스를 만들때 설정한 dataSources의 restApi를 호출해 REST요청을 처리한다.
  ```
  var resolver = {
    Query: {
        AirPollutionInfoByStationName: async (_, {stationName, dataTerm}, {dataSources}) => {
            return dataSources.restApi.getRealTimePollutionByStationName(stationName, dataTerm);
        },
  };
  ```
- RestAPI
  - Apollo의 RESTDataSource를 extends 받는다.
  - this.get() 등의 메서드를 통해 REST API를 호출한다.
  ```
  class RestAPI extends RESTDataSource {

    constructor() {
      super();
    }

    async getRealTimePollutionByStationName(stationName, dataTerm) {
       const res = await this.get(URL);
       return res && res.length ? this.airPollutionDataReducer(res) : [];
   }
  }
  ```

##### 마치며
GraphQL 서버를 만들어 보면서 개념이나 구조를 알아볼 수 있어 좋았다. Apollo를 이용해 쉽게 기존 REST API에 연결할 수 있어 편리했다. 미세먼지 GraphQL API서버의 전체소스는 [_깃허브_](https://github.com/oookawesome/finedust-graphql-api)에 있다.

처음 관심을 가졌을 때는 REST의 단점을 완전히 상쇄시키고 대체하는 것이 가능할까에 대해 궁금했는데, 여러 포스트를 보니 둘 사이의 장단점이 달라, 완전한 대체제라고 보기는 어려운 것 같았다. 언급된 몇 가지의 GraphQL의 단점으로는,
- 파일 전송 처리가 복잡하다.
- 캐싱 처리가 어렵다
- 초기 단계라 레퍼런스나 문서가 부족하다
- 헤더 content-type을 application/graphql로 설정해야 한다
- Node.js 기반이 아닌 경우, 관련 라이브러리가 부족
- 백엔트 툴 부족

이런 것들이 있었다.

앞으로 어떻게 발전해나갈지 지켜보면 재밌을 것 같다.

-----
##### 출처
- https://graphql.org/graphql-js/
- https://www.apollographql.com/docs/apollo-server/essentials/server.html
- https://engineering.huiseoul.com/%ED%95%9C-%EB%8B%A8%EA%B3%84%EC%94%A9-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EB%8A%94-graphql-421ed6215008
- https://engineering.huiseoul.com/%ED%95%9C-%EB%8B%A8%EA%B3%84%EC%94%A9-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EB%8A%94-graphql-421ed6215008
- https://www.clien.net/service/board/lecture/12489593
- https://d2.naver.com/helloworld/2838729
- https://medium.com/@FourwingsY/graphql%EC%9D%84-%EC%98%A4%ED%95%B4%ED%95%98%EB%8B%A4-3216f404134
- https://www.holaxprogramming.com/2018/01/20/graphql-vs-restful-api/
