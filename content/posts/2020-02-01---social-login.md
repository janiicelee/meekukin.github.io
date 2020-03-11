---
title: "카카오 소셜로그인 구현하기"
date: "2020-02-01T22:40:32.169Z"
template: "post"
draft: false
slug: "kakao-social-login"
category: "Django"
description: "카카오 소셜로그인을 어떻게 적용할 수 있는지 알아보자."
socialImage: "#"
---

### 카카오 개발자 등록하기   
- [Kakao Developers](https://developers.kakao.com/) 에 접속하여 개발자 등록을 하고 앱서비스를 등록하고 나면, api 키를 발급받을 수 있다. 우리는 rest api를 사용하므로 rest api 키를 발급받는다. 설정-사용자 관리탭에서 사용자 관리 활성화를 해주고, 서비스에서 사용자들에게 받고 싶은 정보를 설정한다.   
- 코드를 리다이렉트 받을 Redirect URI를 설정해야 합니다. Third-party(개발한 페이지)에서 user가 로그인을 요청하게 되면 카카오의 로그인 페이지로 이동합니다. 사용자가 로그인에 성공하면 다시 Third-party로 돌아가야 하는데
그 돌아갈 페이지의 주소가 Redirect URI 입니다.

### 액세스 코드 받기   
- 유저가 정보제공에 동의하고 나면, 코드는 쿼리스트링에 담겨서 redirect uri로 넘어가게 된다.   

```python
import json
from django.views import View 
from django.http  import JsonResponse, HttpResponse   

class KakaoLoginView(View:
    def get(self, request):
        kakao_access_code = request.GET.get('code',None)
        return HttpResponse(f'{code}')

```   
   
### 액세스 토큰 받기   
        
코드를 받았으니 코드를 이용해서 액세스 토큰을 받아와야 한다. 위의 코드에서 리턴 부분을 지우고 다음을 추가한다.   
```python
        url = 'https://kauth.kakao.com/oauth/token'
        headers = {'Content-type':'application/x-www-form-urlencoded; charset:utf-8'}
        body = {
            'grant_type': 'authorization_code',
            'client_id': app_key,
            'redirect_uri':'http://localhost:8000',
            'code': kakao_access_token
        }
        kakao_response = requests.post(url, headers = headers, data = body)
        return HttpResponse(f'{kakao_response.text}')
```   
지금까지 어떻게 통신이 이뤄지고 프론트앤드에서 어떻게 액세스 토큰을 받게 되는지까지 알아보았다.   

### 사용자 정보 요청   
- 사용자 정보 가이드라인에 맞춰 get 요청을 보낸다. 

### 회원가입, 로그인, 토큰발행
- Social Login은 하나의 EndPoint로 회원가입과 로그인이 모두 이뤄질 수 있다. 우리 서비스에서 소셜로그인을 한 이력이 있는 회원의 경우, DB에 정보를 넣을 필요없이 DB에 있는 회원정보를 불러와 JWT를 발행한 뒤, 로그인 처리를 하면 된다.

- 반대로 소셜로그인 이력이 없는 회원의 경우, DB에 필요한 정보를 저장시킨 뒤, 방금 저장시킨 정보를 토대로 JWT를 발행하고 로그인 처리를 하면 된다. 유저 입장에선 회원가입 후 로그인 혹은 그냥 로그인이지만 EndPoint는 하나로 처리하는 것이다. 

- 구현 코드   
```python
class KakaoSignInView(View):
    def get(self, request):
        kakao_access_token = request.headers["Authorization"]
        headers            = {'Authorization':f"Bearer{kakao_access_token}"}
        URL                = "http://kapi.kakao.com/v2/user/me"
        response           = requests.get(URL, headers = headers)
        kakao_user_info    = response.json()

        if User.objects.filter(social_id = kakao_user_info['id']).exists():
            user                 = User.objects.get(social_id = kakao_user_info['id'])
            payload              = {"user_id":user.id}
            encryption_secret    = SECRET_KEY
            algorithm            ="HS256"
            encoded_access_token = jwt.encode(payload, encryption_secret, algorithm = algorithm)
            
            return JsonResponse({"access_token":encoded_access_token.decode("utf-8")}, status = 200)
        
        else:
            User(
                    social_platform_id = User.objects.get(platform = "kakao").id,
                    social_id          = kakao_user_info['id'],
                    ).save()

            user                 = User.objects.get(social_id = kakao_user_info['id'])
            payload              = {"user_id": user.id}
            encryption_secret    = "HS256"
            encoded_access_token = jwt.encode(payload, encryption_secret, algorithm = algorithm)
            
            return JsonResponse({'access_token':encoded_access_token.decode('utf-8')}, status = 200)
```   


##### 참조:   
[Kakao Developers](https://developers.kakao.com/)    
https://velog.io/@devzunky/TIL-no.78-Django-Kakao-Social-Login-Back-End-4ik2xay36a  





