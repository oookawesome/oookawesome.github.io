---
layout: post
title: "ElastAlert를 이용한 ElasticSearch 로깅 알람 설정"
date: 2020-07-30 14:30:00 +0300
categories: ELK
tags: ElasticSearch ElastAlert
comments: 1
---
##### ElastAlert
Yelp에서 개발한 오픈소스 라이브러리로, ElasticSearch의 데이터 패턴에 대한 알림을 설정할 수 있다. 다양한 커스터마이징을 제공함

##### 사용 배경 
ElasticSearch에서 공식 제공하는 Alerting 플러그인은 X-Pack에 포함된 것으로, __유료임__다

##### 적용 방식
개발 서버의 로그를 Logstash에 저장하고 있다. 에러 발생 시, ERROR 로그를 출력하도록 되어있는데, 이 패턴을 감지하면 알림을 받고자 했다.

ERROR 로그는 에러시에만 발생하도록 되어있어, frequency 등의 설정은 필요가 없었고, 한번만 매치가 되도, 바로 알림을 전송하도록 했다. 

사내 메일이나 JIRA에 알람을 넣기 위해서는 결재 등 귀찮은 과정을 거쳐야 했기에, 우회 방식을 취했다.
현재 이용하고 있는 테스트 대시보드에 테스트 실패 결과를 업로드해 알림을 받도록 했다.

이를 위해, api를 호출하는 파이썬 스크립트를 작성했다. 알림 시, 이 스크립트를 실행하도록 구성했다.

##### Setting
- pip 패키지 설치
	```
	$pip install elastalert
	$pip install elasticsearch # 설치된 Elastic버전에 맞는 지 확인필요
	```
- config 설정
	- [elastalert git hub](https://github.com/Yelp/elastalert])에서 config.yaml.example 복사
	- 설정값 세팅
		- 파일명 : config.yaml
		- rules_folder : rule yaml이 있는 폴더
		- run_every : query 조회 시간 간격
		- es_host, es_port : elastic 서버 주소 및 port
- rule 설정
	- [elastalert git hub](https://github.com/Yelp/elastalert])에서 example_rules/example.frequency.yaml을 rules폴더로 복사
	- 설정값 세팅
		- 파일명 : default_rules.yaml
		- es_host, es_port : elastic 서버 주소 및 port
		- use_ssl
		- type : alert type 설정
			- type 종류는 [doc참조](https://elastalert.readthedocs.io/en/latest/ruletypes.html#rule-types)
			- any : 매칭되면 알림
		- index : 조회할 elastic 인덱스
			- server-*
		- filter : elastic search 필터 설정
			- [es 웹페이지 참조](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
			- query_string
				- query: "message: ERROR"
		- alert : 알림 방법. 자세한 방법은 [doc참조](https://elastalert.readthedocs.io/en/latest/ruletypes.html#alerts)
			- command : python command 모듈의 arg 형태로 인자를 받아 command 수행
				- 인자 : \['python', 'send_api.py'\]
			- JIRA, Slack 등 다양한 협업 도구의 알림을 지원한다고 함

##### 설정 완료 확인
```
$elastalert-test-rule --config config.yaml rules/default_rules.yaml
```

##### 실행 
```
$python -m elastalert.elastalert --config config.yaml --verbose --rule rules\default_rules.yaml
```

---
##### 참고
- https://www.theteams.kr/teams/865/post/64560