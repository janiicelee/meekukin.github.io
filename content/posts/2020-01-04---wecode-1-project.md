---
title: "Wecode 1차 프로젝트"
date: "2020-01-04T22:40:32.169Z"
template: "post"
draft: false
slug: "위코드-1차-프로젝트"
category: "Projects"
description: "위코드에서 진행한 스페이스클라우드 클론 1차 프로젝트"
socialImage: "#"
---

## Introduction
![](/media/wespace_screenshot.png)   
영상: https://www.youtube.com/watch?v=ZN4xooW1LEQ

+ WeSpace: [SpaceCloud](https://www.spacecloud.kr/) 클론
+ 약 2주간 진행 (19.12.23 - 20.01.03)
+ 프론트앤드 2명 , 백엔드 2명으로 구성된 팀 프로젝트
+ Git으로 소스코드 관리 - [WeSpace Backend](https://github.com/meekukin/WeSpace_backend) git
+ Trello를 이용한 협업, daily scrum meeting 진행

## 사용된 기술

+ Python, Django, MySQL, Workbench
+ Bcrypt, JWT
+ RESTful API
+ BeautifulSoup4, Selenium

## 참여한 부분
+ 데이터 모델링
+ 로그인/회원가입 API
+ Main page - category endpoint

## 느낀점 
+ 데이터 모델링   
처음 해보는 것이어서 어려웠다. Aquery tool을 이용하여 테이블을 만들고 관계형 데이터베이스에 대해 고민해볼 수 있었다. 어떤 데이터의 테이블이 필요할지 생각해보고 서로 관계가 어떻게 되는지, 1-N인지 N-N인지 고민하는 과정을 통해 개념을 다시한번 익힐 수 있었으나 막상 실제로 적용해보려하니 쉽지만은 않았다. 처음에 Account app 안에 사용자와 호스트를 같이 넣지 않고 따로 model을 짰는데 나중에 호스트 로그인을 따로 만들어야 하기도 했다. 오래걸리더라도 처음부터 모델링을 할때 신중하게 해야될 것 같다는 생각을 했다. 

+ 로그인/회원가입 API   
try/except 로 여러 exceptions를 고려해야된다는 점이 어려웠지만 사용자 입장에서는 매우 중요하고 필요한 부분이라는 것을 깨달았다. 비크립트를 이용한 비밀번호 암호화 개념도 좀더 정리가 필요할 것 같다. 깃에 소스 코드를 올릴때는 gitignore, my_settings 등의 파일도 잊지 말 것! 소셜로그인 api를 구현하지 못해 아쉬웠지만 좀 더 공부해서 다음 프로젝트 때 구현해보고싶다. 데코레이터에 대한 이해가 부족했는데 로그인 데코레이터를 구현해보면서 어떤식으로 데코레이터가 적용되는지에 대해 더 알 수 있어서 좋았다.

+ 메인페이지 API   
프론트에서 어떻게 데이터를 받아야 매핑을 잘 할 수 있을지에 대해 무지했는데 이번에 구현하면서 조금은 알게 되었다. 로직 구현할때 python list comprehension을 쓰는 것과 flat=True 쓰기. 예를 들면, 하나씩 튜플로 나오지 않고 객체로 나와서 프론트에서 매핑하기에 더 편리했다.
```
>>> Entry.objects.values_list('id').order_by('id')
<QuerySet[(1,), (2,), (3,), ...]>
>>> Entry.objects.values_list('id', flat=True).order_by('id')
<QuerySet [1, 2, 3, ...]>
```
## 기록하고 싶은 코드

+ 로그인 데코레이터

```
import jwt
import json
from django.http import JsonResponse
from .models import Accounts, Hosts
from wespace.settings import SECRET_KEY

def login_decorator(func):
    def wrapper(self, request, *args, **kwargs):
        if "Authorization" not in request.headers:
            return JsonResponse({"error_code": "INVALID_LOGIN"}, status=401)

        encode_token = request.headers["Authorization"]

        try:
            data = jwt.decode(encode_token, SECRET_KEY, algorithm='HS256')
            account = Accounts.objects.get(id=data['id'])
            request.account = account

        except jwt.DecodeError:
            return JsonResponse({"error_code": "INVALID_TOKEN"}, status=401)

        except Accounts.DoesNotExist:
            return JsonResponse({"error_code": "UNKNOWN_USER"}, status=401)

        return func(self, request, *args, **kwargs)
    return wrapper

		
```

+ Python list comprehension 이요한 로직구현.

```
from django.views import View
from django.http import JsonResponse

from .models        import Spaces, Space_Categories, Images
from account.models import Hosts


class CategoryView(View):
    def get(self, request):
        try:
            categories = list(Space_Categories.objects.values('id','category'))
            return JsonResponse({'Categority':categories}, status=200)
        except:
            return JsonResponse({'result':'ERROR'}, status =400)

class RecommendView(View):
    def get(self, request):
        spaces = list(Spaces.objects.all())[:6]
        recommend = [{
            'title'    : space.title,
            'price'    : space.price,
            'location' : space.location,
            'image'    : list(space.images_set.filter(space_id = space.id).values('space_image'))
        } for space in spaces]

        return JsonResponse({'data':recommend}, status =200)

class EditorView(View):
    def get(self, request):
        spaces = list(Spaces.objects.all())
        editor = [{
           'image'       : list(space.images_set.filter(space_id = space.id).values_list('space_image', flat=True)),
            'tag'        : list(space.tags_set.filter(space_id = space.id).values('tag')),
            'title'      : space.title,
            'price'      : space.price,
            'long_intro' : space.long_intro
            } for space in spaces]

        return JsonResponse({'data':editor}, status = 200)
        
class DetailSpaceView(View):
    def get(self, request, space_id):
        space = [
            {
                'id': space.id,
                'title': space.title,
                'short_intro': space.short_intro,
                'long_intro': space.long_intro,
                'price': space.price,
                'location': space.location,
                'open_time': space.open_time,
                'close_time': space.close_time,
                'min_guest': space.min_guest,
                'min_time': space.min_time,
                'space_images': list(space.images_set.values_list('space_image', flat=True)),
                'host': list(Hosts.objects.filter(id=space.host_id).values('id', 'nick_name', 'email', 'phonenumber')),
                'tag': list(space.tags_set.values_list('tag', flat=True)),
                'notice': list(space.notices_set.values_list('notice', flat=True))
            }
            for space in Spaces.objects.filter(id=space_id)]
        return JsonResponse({'result': space}, status=200)
```






