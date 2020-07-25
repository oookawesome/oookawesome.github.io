---
layout: post
title: "MSA에 Contract Test 적용 하기"
date: 2018-04-21 15:10:00 +0300
categories: Testing
tags: ContractTest MSA
---

### 투입!
17년 가을부터 대략 8개월간 MSA 개발 프로젝트에 테스트 엔지니어로 투입되었다. 4명이서 팀을 이뤄 테스트 전략 수립, 테스트 가이드, 테스트 자동화 업무를 했다. MSA에서의 테스트를 위해 자료도 찾아보고, 프로젝트 테크 리더의 조언을 중심으로 전체 테스트 전략을 잡았다. API Integration이 많은 마이크로서비스 아키텍처이기에, Contract Test, API Integration Test, Isolated Functional Test (외부 서비스는 모두 stub하고 자기 API를 테스트하는 것으로, 우리는 이렇게 불렀다.. 검색해보니 이를 Component Test라고 많이 한다.) 등 다양한 테스트를 했는데, 특히 고민과 이슈가 많고, 어려움이 많았던 Contract Test에 대해 정리를 해보려고 한다.

### Contract Test?
어짜피 처음 테스트 지원을 나간 것이라 모든게 생소했지만, 이전까지 Contract Test는 정말 들어본 적도 없었다. Contract Test란 무엇인가... Contract Test는 서비스 제공자와 사용자간의 계약(Contract)을 검증하는 것이다. 예를 들어 REST API 서비스가 있으면, 이 API의 사용자는 정의된 API 스펙을 바탕으로 POST, GET등의 서비스를 요청하게 된다. 이 때, API 스펙을 바탕으로 서비스 제공자-사용자간의 규약이 형성되는 것이고, Contract Test에서는 이러한 규약이 정상적으로 지켜지는 지를 테스트하는 것이다.

### Component Test와는 뭐가 다른 가
얼핏 생각하기엔 기존의 API에 대한 Component Test와의 차이점이 없는 것 처럼 느껴진다. 어짜피 API 스펙을 바탕으로 API가 정상적으로 동작하는 지를 테스트한다고 보면, 커버하는 영역이 같다는 생각이 들기 때문이다. 실제 커버하는 영역이 같아 보인다. 예를 들면, GET요청 시 "id"라는 key와 value를 돌려주는 API가 있다고 가정했을 때, Component Test와 Contract Test 둘 다, GET을 호출한 뒤, "id" key에 값이 잘 들어있는 지 확인할 것이기 때문이다.

그러나 차이가 있다. Contract는 실제 서비스 소비자가 활용하는 API의 규약이다. 앞선 예에서 만약 API소비자가 id를 활용하지 않으면, 그건 Contract가 아니고 Contract Test 대상이 아니다. 그렇기 때문에 사실 Contract test의 테스트 케이스는 간단하고 필요한 로직만 검증하도록 작성하게된다.

그래도 이상해보인다. Component Test의 큰 범위안에 속하는 것처럼 보이고 여전히 다 커버되기 때문이다.

### 그럼 이거 왜 할까?
그럼에도 많은 MSA서적, Article에서 Contract Test의 중요성을 강조하고 꼭 해야 한다고 이야기하는 이유가 뭘까? 그것은 Component Test와 Contract Test의 목적이 다르기 때문이다. Contract Test의 목적은 사실 서비스 제공자가 내 서비스를 사용하는 사용자에 대한 정보(insight)를 얻기 위한 것이다.

이를 이해하기 위해선 MSA와 Consumer Driven Contract(CDC)에 대해 알 필요가 있다. 우선, 서적이나 Contract Test를 말하는 곳에서 이야기하는 가장 큰 대전제는 이것이다.

"배포되어 실제 사용자가 사용중인 API는 항상 정상적으로 유지되도록 하고싶다."

MSA에서 서비스는 독립적으로 개발된다. 따라서 내 서비스의 사용자에 대한 정보를 얻기 힘들다. 거기에 덤으로 훨씬 많은 API가 서로 엮이고 개발된다. 이런 상황에서는 앞서 말한 전제를 충족시키기 너무 힘든 것이다. 그래서 소비자 주도 계약(Consumer Driven Contract)의 개념이 나오고 Contract Test의 개념을 도입됬다. 소비자 주도 계약(CDC)은 앞서 말한 Contract의 의미와 같지만, 좀 더 API소비자에 초점을 맞춘 단어라고 생각했고, Consumer Driven Testing도 이 CDC를 테스트하는 것이라 이해했다.

### 이상적인 Contract Test
이러한 전제를 이루기 위해선 당연히 서비스 제공자 측의 개발 변경분에 대한 배포 전에, Contract Test가 수행되어야 하고 테스트가 실패한 경우, 배포가 이루어 지지 않아야 한다. 만약 배포가 되게되면, 빌드는 다 되서 정상적으로 각 서비스들은 잘 떠있는데, UI상에서 실제 기능은 동작을 안하고 Exception을 내거나 하는, 요상한 상황이 나오게된다.

이제 앞선 예에서 Component Test와 Contract Test의 차이를 알 수 있다. 예를 들어 서비스 제공자 측에서 GET호출 시 "id" key를 변경하여 "uuid"라고 바꿨다면, Component Test와 Contract Test는 둘 다 깨질 것이다. 다만 Component Test의 관점에서는 개발 변경사항에 맞춰 테스트를 변경해야 한다. 하지만 Contract Test의 관점에서는 테스트를 수정하는 것이 아니라, 해당 배포를 Holding해야 한다. 그리고 해당 변경이 의도된 것인지 검토해야 하고, 서비스 소비자들과 Serious한 커뮤니케이션과 논의를 해야한다.(Article에서 이렇게 나오더라..) 한마디로 Contract를 깨는 서비스 변경은 매우 신중해야 한다는 이야기이다.

이것이 약간 이상적인 가정이었다고 생각한다.
업무를 하고 테스트를 적용하면서 겪었던 어려움과 시행착오의 근원이 이것이 아닐까?

개발중에 서비스API의 URL이나 Schema변경이 자주 일어났고, 서비스 소비자를 고려해서 개발하는 것도 힘들었다. 무엇보다 서비스 개발자 입장에서 Unit Test, Component Test를 정상적으로 통과해 개발상에 문제가 없음에도, 배포가 막힐 수 있는 상황이 부담이 되었다. 그리고 사실 Contract가 깨져서 배포가 안되는 상황이 되었다고 가정해도, 결국은 그 개발변경이 롤백되는 게 아니라, 배포가 진행되고 소비자가 맞춰서 수정하는 그림이 되는 게 현실이었다.

### Spring Cloud Contract
Contract Test를 위한 Spiking을 통해 나온 툴 후보는 둘이었다. Pact와 Spring Cloud Contract. 우리는 이 중에서 Spring Cloud Contract를 적용하기로 결정했다. 가장 큰 이유는 당시 모든 서비스가 Spring Boot기반으로 개발되었기 때문에 보다 쉽게 활용할 수 있으리란 판단 때문이었다.

Pact와 Spring Cloud Contract는 모두 앞서 말한 이상적 Contract Test의 조건(테스트 실패시 서비스 코드 배포 차단)을 충족할 방법을 보장해줬다. 하지만 일부 차이점이 있었다. 정리해보면 이렇다.

항목|Spring Cloud Contract|Pact
:------:|:------:|:------:
환경|Spring Framework|-
언어|JAVA|Ruby, JAVA, C#, JS ...
Contract명세 위치|서비스 제공자|서비스 소비자

명세 위치를 보면 알 수 있듯이, Pact와 Spring Cloud Contract의 프로세스는 좀 다르다.

Pact는 소비자가 Contract를 Stub코드를 작성하듯(Wiremock쓸 때, stubFor()로 정의하듯이..) 자기 코드에서 작성하면, Pact가 Pact file이라는 Contract명세를 만들고, 이것을 Pact Broker라는 저장소에 올린다. 그리고 소비자측에서는 Pact Broker에서 Pact file을 가져와 내 서비스가 Contract를 만족시키는 지 테스트 한다.

Spring Cloud Contract는 소비자가 Contract를 제공자 코드쪽에 Groovy로 작성한다. 이 후 서비스 코드 빌드시, Spring Cloud Contract는 이 명세를 자동으로 테스트 코드로 generate한다. 이 테스트 코드를 Unit Test와 같이 테스트 하게된다. 테스트가 정상적으로 통과되면, 넥서스와 같은 repository에 stub.jar라는 형태로 업로드된다. 이제 소비자 쪽에서는 이러한 stub.jar를 가져와 실제 서비스로 가정하고 원하는 명세에 대한 테스트를 작성한다.

![spring cloud contract](../../../assets/postImages/spring-cloud-contract.png){: width="100%" height="100%"}

### 적용하면서 어려웠던 점
Spring Cloud Contract를 적용하면서 가장 어려웠던 점은 소비자가 제공자 코드에 Groovy코드를 작성해야 한다는 점이었다. 개발자들은 놀라울 정도로 자기 프로젝트 코드, 빌드 파이프라인에만 관심이 있었다. 코드를 내려받아서, 내게 오너십이 없는 다른 코드에 손을 대 푸시를 하는 일에 불편함을 느끼는 것 같았다. 사실 그리고 테스트 담당자인 우리도 이해하는 데 한참 걸린 이 복잡한 프로세스를 설명하고 공감대를 형성하는 일 자체도 어려운 일이었다.

결국은 Spring Cloud Contract를 활용해 Contract Test를 하는 것은 확실히 진행되지 못하고 진척이 안되는 상황이 되었다.

### 그래서 결국
하지만 개발이 진행될수록, 타 서비스 연계가 많은 서비스들은 어떤 식으로든 호출 서비스의 응답에 대해 테스트를 해야 했다. 하지만 앞서 말한대로 서비스 제공자 측의 배포를 막거나 API변경에 대해 재고하는 일은 현실적으로 불가능 했으므로, 타협점을 찾아 테스트를 수정했다.

소비자 측에서 배포전에 실제 서비스에 대해 테스트를 하는 식으로 방향을 바꿨다. 이렇게 되면서, 제공자 측의 배포는 막지 못하더라도, 소비자 측 배포전에 내가 호출하는 서비스가 정상적으로 응답을 주는 지는 확인이 됬고, 변경됬는 지 모르고 배포를 해버리는 상황은 막을 수 있었다. 실제로 이 테스트를 통해 여러 차례 변경이 캐치가 됬고, 개발자들의 활용도도 높았다.

### 다음에 Contract Test를 한다면
다음에는 Spring Cloud Contract말고 Pact로 해보고 싶다. Pact를 사용하면 제공자 측 코드를 건드리는 일은 막을 수 있을 것으로 보인다. 개발자들의 거부감을 없애 줄 것이라고 생각한다. 물론, Pact를 사용해도 제공자 측 배포를 막는 작업이 이루어 지지 않으면 큰 의미는 없을 것 같다.

상황을 봐야겠지만, 이곳에서 했던 것 처럼, 소비자 측이 배포전에 테스트를 통해 Contract를 확인하고 최대한 빨리 변경을 캐치하는 방식이 현실적이지 않을까?

------
#### 참고
- Contract Test
  - <https://martinfowler.com/bliki/ContractTest.html>
  - <https://martinfowler.com/articles/consumerDrivenContracts.html>
  - <https://www.mabl.com/blog/understanding-contract-testing-microservices-mabl>
  - [도서 인용](https://books.google.co.kr/books?id=OeUlDwAAQBAJ&pg=PA205&lpg=PA205&dq=contract+testing+%EC%9D%98%EB%AF%B8&source=bl&ots=mndEbB-B_w&sig=ycw5DOJJYIZnlyYhQbRNkhhuDV8&hl=ko&sa=X&ved=0ahUKEwjt5NfX2MPaAhVGEpQKHW6iDFAQ6AEIXzAF#v=onepage&q=contract%20testing%20%EC%9D%98%EB%AF%B8&f=false)

- Spring Cloud Contract
  - <https://cloud.spring.io/spring-cloud-contract/single/spring-cloud-contract.html>
  - <https://specto.io/blog/2016/11/16/spring-cloud-contract/>

- Pact
  - <https://docs.pact.io/best_practices/contract_tests_not_functional_tests.html>
  - <https://github.com/DiUS/pact-jvm>
