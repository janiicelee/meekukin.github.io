---
title: "JWT, Access Token & Refresh Token"
date: "2020-03-02T22:12:03.284Z"
template: "post"
draft: false
slug: "jwt-access-token"
category: "Computer Science"
tags:
  - "jwt"
  - "access token"
  - "refresh token"
description: "Json Web Token에 대해 더 알아보자."
socialImage: "#"
---
## 배경지식   
HTTP 통신은 stateless 성격 때문에, 각각의 HTTP 통신은 독립적이며 이전에 어떠한 통신들이 실행됐는지 알지 못한다. 이러한 HTTP의 성질 때문에 생기는 이슈 중 하나가 바로 인증 절차다. 현재의 HTTP 통신에서 이전에 통신에서 이미 인증이 진행됐는지 알지 못하기 때문이다. 그러므로 통신을 할 때는 해당 HTTP 요청을 처리하기 위해서 필요한 모든 데이터(로그인 정보 등)를 첨부해서 요청을 보내야 한다.   
access token은 바로 로그인 정보를 담고있는 데이터이다.

## JWT(JSON Web Token)

Access token을 생성하는 방법 중에 가장 많이 쓰이는 기술 중 하나로, JSON 데이터를 토큰으로 변환하는 방식이다. 

+ 그렇다면 왜 JSON 데이터를 토큰화해야 하는가?
    - 단순 json 데이터를 사용하면 해킹 가능성의 문제가 생긴다. 누구나 json 데이터를 http 요청에 첨부해서 전송할 수 있으므로 실제 사용자가 아님에도 해당 사용자라고 인식될 수 있다. 그래서 토큰은 누군가 해킹 목적으로 가짜 jwt를 전송한다해도 api 서버에서 자신이 생성한 jwt인지 아닌지 확인하는 기능도 제공한다. 

    - 파이썬에서는 pyjwt 라이브러리를 사용해서 jwt를 구현할 수 있다. jwt를 생성하고 복호화도 할 수 있다.

+ 구성   
Json web token은 세 파트로 나뉘어지며, 각 파트는 점(.)에 의해 구분된다.   
`xxxxxx.yyyyyyy.zzzzzzz` 이런식으로 구분된다.
    - header는 토큰의 타입과 해시 암호화 알고리즘으로 구성되어 있다. 
    - payload는 claim 정보를 포함하고 있다. user ID, expire, scope 등이 여기에 해당된다.
    - 마지막으로 signature는 secret key를 포함하여 암호화 되어있다. 

+ 과정   
일반적으로 jwt 기반의 인증 시스템은 처음 사용자를 등록할때 access token, refresh token 모두 발급되어야 한다. 
    - 사용자가 id, password 입력 후 로그인을 시도한다. 
    - 서버는 요청을 확인하고 secret key를 통해 access token을 발급한다.
    - 이후 jwt가 요구되는 api를 요청할때는 클라이언트가 authorization header에 access token담아서 보낸다.
    - 서버는 jwt signature를 체크하고 payload로부터 user 정보를 확인해 데이터를 리턴한다. 

## Access token & Refresh token   
앞서 말했듯이 기본적으로 두가지 토큰을 사용한다. api 요청을 허가하는데 access token을 사용하고, 액세스 토큰이 만료된 후 새로운 액세스 토큰을 얻기 위해 refresh token을 사용한다. 

+ Access token   
access token은 리소스에 직접 접근할 수 있도록 해주는 정보만을 가지고 있다. 클라이언트는 access token이 있어야만 서버자원에 접근할 수 있다. access token은 만료기간이 짧고 주로 '세션'에 담아서 관리한다. (서버 단에서 통제 불가능)

+ Refresh token   
refresh token은 새로운 access token을 발급받기 위한 정보를 갖는다. 클라이언트의 access token이 만료되었다면 refresh token을 통해 auth server에 요청해서 발급받을 수 있다. refresh token도 만료기간이 있지만 길다. 그리고 중요한 토큰이기 때문에 외부에 노출되지 않도록 엄격하게 관리해야한다. 그래서 주로 '데이터베이스'에 저장한다. (서버 단에서 통제 가능, 클라이언트 단에서는 안전한 스토리지에 저장됨)
   
+ Refresh token을 도입했을때 장단점   
    - 장점: 기존의 access token만 있을 때보다 안전하다.
    - 단점: 검증 과정이 길기 떄문에 구현이 복잡해진다. Access token이 만료될때마다 새롭게 발급하는 과정에서 생기는 http요청 횟수가 많다. 이는 서버의 자원 낭비가 될 수 있다.   


###### (참조)   
https://lordmyshepherd.github.io/posts/%EB%A9%B4%EC%A0%91%EC%A4%80%EB%B9%84(%EA%B8%B0%EC%88%A0)   
https://swalloow.github.io/implement-jwt   
https://tansfil.tistory.com/59

