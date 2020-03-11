---
title: "Wecode 2차 프로젝트"
date: "2020-01-19T22:40:32.169Z"
template: "post"
draft: false
slug: "위코드-2차-프로젝트"
category: "Projects"
description: "위코드에서 진행한 보그코리아 클론 2차 프로젝트"
socialImage: "#"
---

## Introduction   
![](/media/메인화면.png)
영상: https://www.youtube.com/watch?v=HZryxyMC3pk&feature=youtu.be   

- [Vogue Korea](http://www.vogue.co.kr/) website cloning.
- Built in two weeks. (01/06/20 - 01/17/20)
- 프론트엔드 3명, 백엔드 2명으로 구성된 팀 프로젝트.
- [Front-end](https://github.com/wecode-bootcamp-korea/Vugue_frontend) git
- Trello를 이용한 협업, daily scrum meeting 진행

## 사용된 기술
- Python, Django, MySQL, Workbench
- Bcrypt, JWT
- RESTful API
- AWS EC2, RDS
- AWS Docker

## 참여한 부분
- 데이터 모델링
- 로그인/회원가입 API
- 소셜로그인 API
- 상세 페이지 API

## 느낀점   
- 데이터 모델링   
![](/media/vugue_erd.png)   
이번 프로젝트의 웹사이트는 관계도가 생각보다 복잡하지 않았다. 그래서 크게 유저, 카테고리, 아티클, 태그로 테이블을 나누고 모델링을 진행했다. 원래 댓글 기능은 없었지만 추가했고, 아티클과 유저가 N-N관계가 되도록 ERD를 작성했다. 유저가 권한이 있을때 글도 포스팅할 수 있도록 모델링을 했는데 결국 시간상 구현해보지는 못했다. 하지만 없던 기능을 추가해보는 과정에서 여러 면에서 한번 더 생각해 볼 수 있었고 어떻게 데이터를 가져올 수 있을지에 대해 더 고민해볼 수 있었다. 

- 로그인, 회원가입, 소셜로그인 API   
이번에도 로그인/회원가입 엔드포인트를 구현하고 추가로 카카오 소셜로그인 엔드포인트도 추가해보았다. 'kakao developers' 페이지에서 등록을 하고, api 키를 받고, 코드를 작성해보면서도 이해가 완벽히 되지 않아 어려움이 있었으나, 카카오 액세스 토큰을 받고 인증,인가를 하는 과정이 기존의 로그인/회원가입 절차와 비슷하다는 것을 알게 되었다. 자세한 과정과 원리를 제대로 공부하고 넘어가야겠다고 생각했고 네이버나 구글 소셜로그인도 많이 쓰이는 만큼, 다음기회에 꼭 해봐야겠다고 생각했다.   

- 상세페이지 API   
아티클이 보여질 수 있도록 프론트에게 제이슨 데이터를 보내주고, 검색을 통해 보고 싶은 아티클을 볼 수 있도록 로직을 구현했다. 팀원의 도움이 컸지만 이번 기회에 select_related, prefetch_related에 대한 개념과 sql문의 활용을 좀더 연습해 볼 수 있었다. 쿼리셋을 잘 다루는 것이 결국 핵심임을 다시 한번 깨달았다. 이외에 unit test와 도커의 사용법도 간단히 짚고 넘어갈 수 있어 많이 배우고 익히는 시간이었다.   

## 기록하고 싶은 코드    
- 카카오 소셜로그인   
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

- 상세페이지 중 비디오뷰   
```python
class VideoView(View):
    def get(self, request):
        offset = int(request.GET.get('offset',0))
        limit  = int(request.GET.get('limit',12))
        videos = Video.objects.select_related('category').order_by('id')[offset * limit: (offset+1) * limit]
        data   = [{
            'id'               : props.id,
            'title'            : props.title,
            'background_image' : props.background_image,
            } for props in videos]
        return JsonResponse(list(data), safe=False, status = 200)
```





