---
title: "RESTful API"
date: "2020-01-18T22:12:03.284Z"
template: "post"
draft: false
slug: "restful-api"
category: "Computer science"
tags:
  - "RESTful API"
  - "URI"
  - "API"
description: "RESTful API란 무엇일까?"
socialImage: "#"
---

## RESTful HTTP API란 무엇일까?

![](/media/rstapi.png)

#### 기본 배경 지식
+ URI: Uniform Resource Identifier
해당 사이트의 특정 자원의 위치를 나타내는 유일한 주소.(/login, /news)
+ HTTP Method: HTTP request가 의도하는 action을 정의한것.(POST, GET 등)
+ Payload: HTTP request에서 보내는 데이터 (body)

#### REpresentational State Transfer
웹상에서 사용되는 여러 리소스를 HTTP URI로 표현하고 그 리소스에 대한 행위를 HTTP Method로 정의하는 방식. 즉, 리소스(HTTP URI로 정의된)를 어떻게 한다(HTTP Method + Payload)를 구조적으로 깔끔하게 표현하는것. Method는 주로 GET과 POST만 사용한다.

+ 유저의 보유 주식 종목들을 DB에 저장하는 HTTP 요청:

```
HTTP POST https://api.trueshort.com/user/portfolio

{
    "user_id" : 1,
    "stocks": [ 
        "005930",
        "298730",
        "378900"
    ]
}
```

#### RESTful API의 장점

+ Self-descriptiveness!
RESTful API는 그 자체만으로도 API의 목적이 쉽게 이해가 된다. 예를 들어, 위의 HTTP GET https://api.trueshort.com/stock/005930 ` 요청의 경우, 문서나 주석이 없이도 "https://api.trueshort.com 라는 API에서 삼성전자 주식에 관한 정보를 HTTP 요청을 통해 받아오는 구나" 라는 해석이 쉽게 가능하다. 

#### RESTful API를 개발할때 유의점

+ /(슬래시)는 계층 관계를 나타낼때 사용된다.
+ RI에 _(underscore)는 주로 포함하지않고 또한 영어 대문자보다 소문자를 쓴다. 그리고 너무 긴 단어는 잘 사용하지 않는다. 이 모든건 가독성을 높이기 위해서다.
+ URI는 명사를 사용한다. 그 이유는 동사는 GET, POST 같은 HTTP Method를 통해 표현하기 때문이다.

