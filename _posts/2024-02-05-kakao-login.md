---
title: 카카오 로그인 어디까지 해봤니?
categories:
  - DEV
tags:
  - kakao
  - login
  - OIDC
  - allauth
last_modified_at: 2024-02-05T22:30:00-00:00
---

## 기존에 운영중인 사이트에 sns 회원가입/로그인을 도입한다면 조금더 나아간 고민을 할수 밖에 없고 이러한 고민을 먼저 진행해본 사람으로써 도움을 주기위해 이번 글을 적어보겠다.

---

카카오 로그인 으로 검색해보면 카카오 간편로그인과 회원가입 정도만 나온다.

하지만 간단한 데모 사이트에서는 이정도만으로도 충분하지만 실제로 이미 email 계정을 베이스로 운영되고 있는 사이트에서 카카오, sns 로그인을 도입할때에는 이야기가 많이 달라진다.

- 기존 가입되어있는 회원과 신규 카카오 회원이 동일한 인물인지 어떻게 판단할수있을까?
- 또 이러한 경우에 데이터베이스에서는 어떻게 처리 해야할까?
- django 에서 카카오 로그인과 같은 sns 로그인을 추가할때에는 무엇을 이용해서 효율적으로 개발할수있을까?
- 카카오 로그인/회원가입 플로우는 어떻게 되고 어떻게 API를 설계할까?

기존에 운영중인 사이트에 sns 회원가입/로그인을 도입한다면 조금더 나아간 고민을 할수 밖에 없고

이러한 고민을 먼저 진행해본 사람으로써 도움을 주기위해 이번 글을 적어보겠다.

# 카카오 싱크 선택 그리고 전체 서비스 로그인 과정

카카오 로그인은 카카오계정으로 다양한 서비스에 로그인할 수 있도록 하는 소셜 로그인 서비스

카카오싱크는 서비스 간편가입 등 카카오 로그인에 더 다양한 확장 기능을 제공하는 비즈니스 설루션

카카오싱크는 카카오 로그인이 제공하는 기능 + 카카오 로그인 시 서비스 약관 동의를 통해 한 번의 동의 절차만으로 간편가입 기능을 제공한다.

![[그림 1] 카카오 로그인 기능 의사 결정 나무](/assets/images/kakaologin_decision_tree.png)

현재 글쓴이가 운영중인 사이트의 경우 의사 결정 나무에 따르면 

회원 가입 기능이 있는 서비스인가? Y / 이메일로 회원가입

서비스 가입 시 제공받아야 할 사용자 정보가 있는가? Y / 이메일, 전화번호 필요

이메일 외의 사용자 정보를 필수 제공받아야 하는가? Y / 전화번호, 생년월일

카카오싱크를 통한 간편가입이 필요했다.

서비스 로그인 과정의 시퀀스 다이어그램을 살펴보면

![[그림 2] 카카오 로그인 전체 시퀀스 다이어그램](/assets/images/kakaologin_process.png)

사용자 클라이언트, 서비스 서버는 운영중인 사이트의 프론트, 백엔드 서버를 나타내고

카카오는 인증 서버, API 서버가 있어 새로 발급받은 토큰을 이용해 각 서버와 API 통신을 할 수 있다.

![[그림 3] 서비스화면에서 카카오 로그인 버튼 예시 ](/assets/images/kakao_login.png)

Step 1. 카카오 로그인은 서비스화면에서 카카오 로그인 버튼을 클릭했을때 서비스 서버와 카카오 인증서버 사이에서 미리 발급받은 카카오 API_KEY 와 카카오 developer 앱에 등록된 서비스의 Redirect URI를 이용해

카카오 토큰을 발급 받는 과정이다.

카카오 토큰은 이후 단계에서 사용자 정보를 가져올때에 필요한 값이다.

이때 발급되는 카카오 토큰을 조금더 자세히 살펴보면

```python
{
	'access_token': 'xxxxxxxxxxxxxx', 
	'token_type': 'bearer', 
	'refresh_token': 'yyyyyyyyyyyyyyyyy', 
	'id_token': 'zzzzzzzzzzzzzzzzz', 
	'expires_in': 21599, 
	'scope': 'birthday account_email profile_image gender birthyear openid profile_nickname name phone_number', 
	'refresh_token_expires_in': 5183999
}
```

access_token 과 refresh_token 은 카카오 서버와 통신할때 사용되는 전용 토큰으로 확인되고

scope 에서는 어떤 정보들을 조회할 수 있는 범위를 나타내는 것을 볼 수 있다.

id_token 을 decoded 한 PAYLOAD 는 아래와 같은 형태였다.

```python
{
  "aud": "3f7561964434b4340b6704218f0e8389",
  "sub": "2643349724",
  "auth_time": 1692921030,
  "iss": "https://kauth.kakao.com",
  "nickname": "DevLabs",
  "exp": 1692942630,
  "iat": 1692921030,
  "picture": "http://k.kakaocdn.net/dn/cgqkmY/btrYqACqZnK/q6yPInOLR66sELBvpfyoIk/img_110x110.jpg",
  "email": "test@devlabs.co.kr"
}
```

위 형태는 OIDC 형태로 앞서 id_token 은 다른 sns 회원가입시에도 제공하는 공통된 형태이다.

OpenID Connect(OIDC) 사용자가 안전하게 로그인하는데 사용할 수 있는 OAuth 2.0 기반의 표준 인증 프로토콜이 적용되어 있다보니 개발을 진행하는 과정에 해당 토큰을 전체토큰으로 사용할지 고민하였으나 기존 운영중인 서비스 토큰과 암호화방식이 다르고 (ID토큰은 RS256, 기존 서비스는 HS256)

카카오로그인으로 로그인 하는 회원이외에도 이메일로 가입하는 유저 등 다른 유저를 고려했을때에

open id 토큰 방식을 전체 토큰 방식으로 적용하지는 않았다.

이후에 다른 sns 로그인 방식까지 고려한다면 open id 토큰으로 통합하는 것을 고려해야 할 것 같다.

Step 2. 회원 확인 및 가입

Step 1 에서 발급받은 카카오 토큰으로 카카오에 등록된 사용자의 정보를 가져온다.

카카오 사용자 정보에는 닉네임, 프로필 사진, 카카오계정(이메일), 이름, 성별, 생일, 출생 연도, 카카오계정(전화번호) 과 다양한 개인정보가 들어있다.

아래 json 에서 kakao_account에는 앞서 카카오 토큰의 scope 에 해당하는 값들에 
대한 정보가 나온다. profile 을 기준으로 설명하면 profile 자체정보 뿐 아니라 profile에 대한 동의여부(profile_nickname_needs_agreement, profile_image_needs_agreement) 까지 포함하고 있다.

```python
{
'id': 2643349000,
'connected_at': '2023-01-30T03:24:11Z', 
'synched_at': '2023-05-15T10:22:05Z', 
'properties': 
	{
	'nickname': 'DevLabs', 
	'profile_image': 'http://ocdn.net/dn/cgqkmY/btrYqACqZnK/img_640x640.jpg', 
	'thumbnail_image': 'http://k.kakaocdn.net/dn/cgqkmY/btrYqACqZnK/img_110x110.jpg'
	}, 
'kakao_account': 
	{
	'profile_nickname_needs_agreement': False, 
	'profile_image_needs_agreement': False, 
	'profile': 
		{
		'nickname': 'DevLabs', 
		'thumbnail_image_url': 'http://k.kakaocdn.net/dn/cgqkmY/img_110x110.jpg', 
		'profile_image_url': 'http://k.kakaocdn.net/dn/cgqkmY/btrYqACqZnK/img_640x640.jpg', 
		'is_default_image': False
		}, 
...
	}
}
```

이러한 카카오 사용자 정보와 현재 서비스의 운영 DB 에 저장되어 있는 유저 정보를 비교해

A라는 사람이 기존 회원과 동일한 사람인지를 판별하는 것이다.

그럼 어떤 기준으로 동일 인물인지를 구분할지를 정해야한다.

여기서 잠깐, 기존 회원을 왜 구분해야하는지 의문이 생길수있다. 
카카오로 간편가입을 시도하는 사람을 신규 회원가입하면 되지않을까 생각할 수 있지만

A 라는 사람에게 이메일 계정1, 카카오 연동 계정2, 네이버 연동 계정3… 복수개의 계정이 제공되면
서비스 입장에서는 유저수가 늘어나는 효과가 있지만(?)

사용하는 유저 입장에서는 복수개의 계정을 관리하기가 어렵고 (우리가 오랜만에 방문하는 사이트에서 늘 반복하는 계정찾기) 결국에는 서비스 만족도가 감소할 것이다.

처음부터 이메일 계정이 있는 유저라면 sns 연동을 유도해 카카오 로그인을 하더라도 기존 이메일 계정을 사용할 수 있게 하면 서비스의 연속성이 유지되고 사용자도 sns 로그인의 편리함을 누릴 수 있다.

https://developers.kakao.com/docs/latest/ko/kakaologin/common#user-mapping-case

해당 링크에는 카카오 로그인 안내문서에서 제공하는 유형별 연동안내에 대한 설명을 확인할수 있다.

여러 고민이 있었지만 글쓴이가 운영하는 사이트에서 유저통합시에 이메일과 이름을 기준으로 기존회원을 판별하기로 하고 해당 기준에 따라 크게 경우의 수를 구분하였고 각 경우는 아래의 경우와 같았다.

1. 일치하는 사용자 정보가 없는 경우
2. 사용자 정보가 일치하는 경우
3. 사용자 정보중 일부만 일치하는 경우 (ex. 이메일만 일치)

2) 사용자 정보가 일치한다면 기존 가입된 계정 정보를 일부 마스킹 된 형태로 제공하고 카카오 로그인 연동의사를 추가로 확인했다.

3) 사용자 정보중 일부만 일치하는 경우에는 실제 회원인지를 검증하는 장치가 필요한데
일치하는 정보에 대한 단서를 제공하고 로그인에 성공한 경우에 카카오 로그인 연동의사를 확인했다.

1) 일치하는 사용자 정보가 없는 경우는 깔끔하게 신규가입으로 진행했다.

Step 3. 서비스 로그인

카카오 로그인이 처음이라면 연동절차를 진행하고, 카카오 연동을 이미완료한 유저라면 카카오 로그인 후 

서비스에서 사용할 세션(토큰)을 발급해 로그인 완료처리를 한다.

로그인 이후에는 기존과 동일하게 서비스를 이용하면 되고 
카카오 로그인이 추가됨에 따라 로그아웃 시에도 약간의 로직 수정이 필요했다.

예를 들면 카카오 로그인인지 체크하고 카카오 토큰을 만료시키는 작업말이다.

# django-allauth

카카오 로그인 연동을 위해 django-allauth 를 사용했다. 모든 기능을 직접 구현해도 되지만 이용할수 있는 패키지가 있다면 십분활용하고 나머지 기능을 구현하는데 집중하는게 효과적이라 생각했다. 

https://docs.allauth.org/en/latest/installation/quickstart.html

**Quickstart 페이지에 따라 설치를 하면**

데이터베이스에 SocialApp, SocialAccount, SocialAccount 테이블이 추가된다

SocialApp 에서 provider 는 kakao 가 되고

client_id 은 내 애플리케이션 > 앱 설정 > 요약정보 > REST API 키에 해당

secret 은 보안을 강화하기 위해 추가로 사용할수있는 키로  제품설정 > 카카오 로그인 > 보안에서 코드를 생성해서 확인할 수 있다

sites 는 여러개를 등록할 수있는데 백엔드에서 사용하는 api 주소라기보다 프론트에서 카카오 로그인 관련 api를 호출할때 사용하는 도메인 주소에 해당한다.

```python
class SocialApp(models.Model):
    objects = SocialAppManager()

    # The provider type, e.g. "google", "telegram", "saml".
    provider = models.CharField(
        verbose_name=_("provider"),
        max_length=30,
        choices=providers.registry.as_choices(),
    )
    # For providers that support subproviders, such as OpenID Connect and SAML,
    # this ID identifies that instance. SocialAccount's originating from app
    # will have their `provider` field set to the `provider_id` if available,
    # else `provider`.
    provider_id = models.CharField(
        verbose_name=_("provider ID"),
        max_length=200,
        blank=True,
    )
    name = models.CharField(verbose_name=_("name"), max_length=40)
    client_id = models.CharField(
        verbose_name=_("client id"),
        max_length=191,
        help_text=_("App ID, or consumer key"),
    )
    secret = models.CharField(
        verbose_name=_("secret key"),
        max_length=191,
        blank=True,
        help_text=_("API secret, client secret, or consumer secret"),
    )
    key = models.CharField(
        verbose_name=_("key"), max_length=191, blank=True, help_text=_("Key")
    )
    settings = models.JSONField(default=dict, blank=True)

    if allauth.app_settings.SITES_ENABLED:
        # Most apps can be used across multiple domains, therefore we use
        # a ManyToManyField. Note that Facebook requires an app per domain
        # (unless the domains share a common base name).
        # blank=True allows for disabling apps without removing them
        sites = models.ManyToManyField("sites.Site", blank=True)
```

social account 는 

user 는 서비스에 별도 존재하는 user 테이블과 foreignkey 로 연결되어 id 값이

provider 는 kakao

uid 는 social account 에서 생성하는 id 값이다

extra_data 는 카카오 로그인시 카카오에서 제공하는 개인에 대한 정보가 json 형태로 저장된다

```python
class SocialAccount(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    # Given a `SocialApp` from which this account originates, this field equals
    # the app's `app.provider_id` if available, `app.provider` otherwise.
    provider = models.CharField(
        verbose_name=_("provider"),
        max_length=200,
    )
    # Just in case you're wondering if an OpenID identity URL is going
    # to fit in a 'uid':
    #
    # Ideally, URLField(max_length=1024, unique=True) would be used
    # for identity.  However, MySQL has a max_length limitation of 191
    # for URLField (in case of utf8mb4). How about
    # models.TextField(unique=True) then?  Well, that won't work
    # either for MySQL due to another bug[1]. So the only way out
    # would be to drop the unique constraint, or switch to shorter
    # identity URLs. Opted for the latter, as [2] suggests that
    # identity URLs are supposed to be short anyway, at least for the
    # old spec.
    #
    # [1] http://code.djangoproject.com/ticket/2495.
    # [2] http://openid.net/specs/openid-authentication-1_1.html#limits

    uid = models.CharField(
        verbose_name=_("uid"), max_length=app_settings.UID_MAX_LENGTH
    )
    last_login = models.DateTimeField(verbose_name=_("last login"), auto_now=True)
    date_joined = models.DateTimeField(verbose_name=_("date joined"), auto_now_add=True)
    extra_data = models.JSONField(verbose_name=_("extra data"), default=dict)
```

SocialToken 은 앞서 나온 2개 테이블 SocialApp, SocialAccount 을 foreign key 관계로 연결되어있으며

token에는 sns 로그인 성공시 발급되는 token 이 저장된다.

```python
class SocialToken(models.Model):
    app = models.ForeignKey(SocialApp, on_delete=models.SET_NULL, blank=True, null=True)
    account = models.ForeignKey(SocialAccount, on_delete=models.CASCADE)
    token = models.TextField(
        verbose_name=_("token"),
        help_text=_('"oauth_token" (OAuth1) or access token (OAuth2)'),
    )
    token_secret = models.TextField(
        blank=True,
        verbose_name=_("token secret"),
        help_text=_('"oauth_token_secret" (OAuth1) or refresh token (OAuth2)'),
    )
    expires_at = models.DateTimeField(
        blank=True, null=True, verbose_name=_("expires at")
    )
```

# 주요로직 상세설명

이제 부터는 신규 추가된 url 을 기준으로 주요로직을 확인해보면

```python
path('kakao/login', kakao_login, name='kakao_login'),
path('kakao/callback', kakao_callback, name='kakao_callback'),
path('kakao/login/finish', KakaoLogin.as_view(), name='kakao_login_todjango'),

path('kakao/register-new-user', kakao_register_new_user, name='register_new_user'),
path('kakao/connect', connect_kakao_account, name='kakao_connect'),
path('kakao/link', link_existing_account_to_kakao, name='kakao_link'),

path('logout', unified_logout, name='logout'),
path('signup', signup, name='signup'),
```

서비스화면에서 카카오 로그인 버튼을 클릭하면 /kakao/login  을 호출하고 카카오로그인 성공시

카카오 개발자 어플리케이션에 등록되어있는 redirect uri 로 이동한다

![[그림 4] 카카오 개발자 어플리케이션에서 Redirect URI 등록하는 화면](/assets/images/kakao_redirect_uri.png)

redirect uri 는 (프론트를 한번 걸치긴 하지만 결국) 백엔드의 /kakao/callback를 호출한다

여기서는 앞서 성공한 카카오 로그인에서 code 값을 가져와

kakao 토큰을 조회하고

해당 토큰을 갖고 카카오에 저장되어있는 프로필 정보를 가져온다

해당 정보를 가지고 기존 운영 데이터와 비교하여 일치하는 사용자, 일부일치 사용자, 불일치 사용자를 구분한다

내부적으로 구현시에는 모든 경우의 수가 9가지 정도 되었다. A/B 서비스 유저인지, 비활성화된 유저인지 등.

사용자 정보 일부일치의 경우에도 이름과 이메일 로 동일인을 구분하는 기준을 정하다보니

이름만 일치하는 경우와 이메일만 일치하는 경우로 나뉘어 이후 로직이 구분되었다. 

나뉘어있는 전체 경우 중에서 아래의 경우에서는 kakao_callback() 함수 안에서

소셜계정이 있는 경우(카카오 연동되어있어 이미 로그인을 했던 사람)

소셜계정은 없지만 기존 회원인 경우 를

process_login_or_register() 메서드를 호출해 처리해다

```python
def process_login_or_register(access_token, code):
    data = {'access_token': access_token, 'code': code}
    accept = requests.post(f"{BASE_URL}/auth/kakao/login/finish", data=data)

    if accept.status_code != 200:
        return None, None, accept.status_code
    accept_json = accept.json()
    user_id = accept_json["user"]["id"]
    accept_json.pop('user', None)
    return user_id, accept_json, accept.status_code
```

process_login_or_register() 은 소셜계정을 조회하거나 소셜계정을 등록하는 작업을 진행하는데
/kakao/login/finish 을 호출,

/kakao/login/finish 는 SocialLoginView을 상속받은 KakaoLogin 클래스와 연결

KakaoLogin 클래스에는 adapter_class, client_class, callback_url 가 있다.

그럼 그 끝에 연결된 CustomSocialAccountAdapter 클래스의 각 메서드가 무엇을 의미하는지 확인해봐야할 것이다.

```python
from rest_auth.registration.views import SocialLoginView
from allauth.socialaccount.adapter import DefaultSocialAccountAdapter

class CustomSocialAccountAdapter(DefaultSocialAccountAdapter):
	def populate_user(self, request, sociallogin, data):
			(생략)
			
	def pre_social_login(self, request, sociallogin):
			(생략)

	def disconnect(self, provider, access_token, social_id=None):
			(생략)

class KakaoLogin(SocialLoginView):
	adapter_class = kakao_view.KakaoOAuth2Adapter
	client_class = OAuth2Client
	callback_url = KAKAO_REDIRECT_URI_CALLBACK
```

django-allauth 에는 populate_user()에 대해 아래와 같이 설명한다.

• **`populate_user**(self, 요청, 소셜로그인, 데이터`): 사용자 인스턴스`(sociallogin.account.user`)를 추가로 채우는 데 사용할 수 있는 훅입니다. 여기서 `데이터는` 공급자가 이미 추출한 일반적인 사용자 속성`(첫` 번째 이름, `마지막 이름`, `이메일`,`사용자 이름`, `이름`)의 딕셔너리입니다.

```python
def populate_user(self, request, sociallogin, data):
        provider = sociallogin.account.provider
        user = super().populate_user(request, sociallogin, data)

        login = sociallogin.serialize()
        account = login.get('account')
        extra_data = account.get('extra_data')
        kakao_account = extra_data.get('kakao_account')

        if provider == 'kakao':
            email = kakao_account.get('email')
            name = kakao_account.get('name')
            nickname = kakao_account.get('profile')['nickname']

            user.email = email
            user.first_name = name
            user.username = nickname

        elif provider == 'another_sns':
            pass

        return user
```

populate_user() 에서는 필수값들만으로 바로 user를 테이블에 추가할수 있지만

카카오에서 제공하는 개인 프로필 정보들을 이용해 시스템에서 필요한 추가 사용자 정보들을 원하는 필드에 셋팅할수 있게 한다.

pre_social_login() 에서는 소셜 로그인을 하기전에 유저의 상태를 체크해 처철리 수 있게 한  부분이다.

첫번째 ‘MANDATORY_ITEMS_NOT_AGREED’ 경우는

서비스에서 유저의 필수값 정보들이 없는 경우를 나타낸다. 예를 들면 서비스에서 이름이 필수값임에도

앞서 카카오 토큰으로 저장되어있는  개인정보를 가져오는 부분에서 이야기했던 프로필에 대한 동의여부가 False 인 경우에 해당하겠다.

또한 식별한 유저가 자체 서비스에서 비활성화된 유저인 경우를 구분해서

이러한 경우에는 비활성을 해제하는 이메일을 보낼수 있다.

마지막 경우는 재인증의 경우인데, 어떤 경우에 해당하냐면

sns 로그인을 한 이후에 프로필이나 마이페이지에서 다시 개인정보를 수정하기 위해 다시 재인증을 받는 경우이다.

sns로 로그인했기에 개인정보 수정과 같이 민감한 정보에 접근하거나 수정하는 경우

다시 권한 체크를 하기 위함이다.

```python
def pre_social_login(self, request, sociallogin):
        user = sociallogin.user
        provider = sociallogin.account.provider
        access_token = sociallogin.token.token

        if access_token is None:
            return

        if not user.id:
            if not user.username or not user.first_name:
                self.disconnect(provider, access_token)
                return JsonResponse({"user_mapping": "MANDATORY_ITEMS_NOT_AGREED",                }, status=status.HTTP_400_BAD_REQUEST)

        elif not user.is_active:
            context = {
                "verification_link": generate_email_verification_link(user)
            }
            send_email(
                subject="Activate your account",
                template="email/template.html", #f'Click the link to activate: {verification_link}',
                context=context,
                from_email='noreply@yourapp.com',
                to_email=user.email,
            )
            return JsonResponse({
								"user_mapping": "DEACTIVATION_USER",
            }, status=status.HTTP_401_UNAUTHORIZED)

        # 재인증 호출시점 확인
        if SocialAccount.objects.filter(user=user, provider=provider).exists():
            if request.user.is_authenticated:
                # 재인증 로직 처리
                return JsonResponse({"user_mapping": "RECERTIFICATION",                }, status=status.HTTP_200_OK)  # new
```

disconnect() 부분은 이미 카카오 연동을 마친 회원이 프로필이나 마이페이지에서 카카오 연동해제를 요청하는 경우 또는 카카오 연동을 시도하는 중 이탈하는 경우(카카오 연동 의사를 물어봤지만 거절한 경우)에 소셜연결을 끊을때 필요한 메서드이다. 카카오에서는 이러한 경우를 위해 unlink 와 같은 api를 제공한다.

```python
def disconnect(self, provider, access_token, social_id=None):
        # 각 소셜에 대한 연결 끊기 API 호출
        if provider == 'kakao':
            # 카카오 연결끊기 로직
            response = requests.post(
                "https://kapi.kakao.com/v1/user/unlink",
                headers={"Authorization": f"Bearer {access_token}"},
            )
            if response.status_code != 200:
                logger.info("disconnect kakao error")

        elif provider == 'another_sns':
            raise NotImplementedError("Disconnect for another_sns is not implemented")
```

# 카카오 연동 개발 실전 노하우

위 설명에서 보시다시피 우리가 편리하게 사용하는 sns 연동 뒤에는 여러 다양한 케이스들을 다루는 로직이 숨어있었다. 사이트를 처음 개발할때부터 sns 로그인/회원가입을 고려해 개발하는 것이 베스트인 것 같고

초기에 이러한 기회를 놓치게 되면 기존 유저 테이블을 고려해야하다보니 경우의 수가 더 많이 늘어나 난이도가 훨씬 상승하는 것 같다.

카카오 싱크를 사용하는 이유가 서비스 약관 동의를 한번에 받을 수 있어 회원가입을 쉽게 할수 있다고 작성한 처음부분을 기억하는가?

유저를 등록할때에 서비스 약관 동의 이력을 따로 남겨야 해서 해당 정보를 체크해 이력을 쌓았다.

카카오 로그인 관리화면에서 약관 동의 부분을 등록할때에 TAG 설정을 할수 있고

해당 TAG를 기준으로 회원의 약관동의 여부를 체크했다.

데이터베이스를 따로 공유하지 않는 패밀리 사이트 회원가입여부는

같이 사용하는 카카오 어플리케이션 기준으로 약관동의를 이미 했는지 여부를 조회해 구분할수 있었다.

ex. 현재 서비스에는 해당 유저정보가 없어 신규회원임에도 사용중인 카카오앱에서 조회한 약관동의 여부가 동의인 경우

카카오에서 제공하는 개인정보들 중 scope 에 해당하는 정보들은

유저테이블과 별로도 관리하는 유저 기타정보 테이블에 해당 정보들을 1차 가공해서 저장하였다.

주의할 것은 일부 유저의 경우 생년월일을 양력이 아닌 음력으로 저장한 유저도 있는데

이러한 경우 birthday_type 이 같이 제공되기 떄문에 해당 정보를 통해 구분할 수 있다.

(SOLAR(양력), LUNAR(음력))

해당 부분은 은근히 빠뜨리긴 쉬운데 운영하는 서비스가 생년월일 정보가 중요한 정보인 경우 해당 필드를 체크해 일관성 있게 저장하는게 중요할 것 같다.

카카오는 개발자 어플리케이션 내부에 테스트앱을 제공한다

실제 운영환경에 배포를 위해선 처음 시작할때부터 테스트앱과 개발환경을 구분해서 관리하는게 낫다

카카오 로그인 메뉴에 있는 Redirect URI 설정부분도 테스트환경에 따라 다르게 관리하는게 권장된다.

![[그림 5] 카카오 개발자 어플리케이션에서 테스트 앱을 설정하는 화면](/assets/images/kakao_test_app.png)

테스트 앱의 경우 앱 설정 > 팀 관리에 이미 등록되어있는 계정에 한해서만 접근을 허용한다.

함께 개발하는 팀원이 있다면 팀 관리 메뉴에서 팀원 초대를 할 수 있다.

![[그림 6] 카카오 개발자 어플리케이션에서 팀 관리 화면](/assets/images/kakao_team_mngt.png)

기존에 서비스 중이었고 패밀리사이트에서 먼저 회사의 카카오앱을 연동하여 사용하고 있는 중이라

서로 다른 회원관리 체계와 비활성화 된 유저 등 예상보다 훨씬 더 많았던 경우의 수를 처리하는게 쉽지않았다.

이러한 상황에서 개발을 진행한다면 가장 처음에 할일은  전체 상황을 파악할 수 있게 플로우를 작성하는 것이다.

신규회원은 어떤 화면들을 지나 우리 서비스의 메인화면으로 이동하는지 그 과정에서 내부적으로는 어떤 것을 체크하는지 등을 포함할 수 있도록.

해당 플로우를 바탕으로 생각하지 못했던 경우를 추가해 나간다면 복잡한 상황을 정리해 카카오 로그인을 도입하는게 불가능하지는 않겠다.

# 기능 개발 결과.

어려움 끝에 카카오 회원가입/로그인을 연동하였고 기능 개발 시점 전후 2개월을 비교했을때에

신규 회원가입자 수는 241.9 % 증가 하였으며

신규 회원가입자 수 중 카카오 로그인을 통한 회원가입 비중도 전체 중 75%로 확인되었다.

기존에는 이메일주소와 문장인증을 통한 b2c 회원가입만 있었기에 해당 기능 개발이 유의미한 결과를 가져왔음을 알수 있었다.

더 나아가 이런 신규 회원가입의 증대로

매출과 연결되는 결제 성공건과 결제 금액은 각각 직전 2개월 대비 4배, 3.8배 증가하였고

결제 실패건을 포함한 전체 결제건수로도 3.2배 증가하였다.

총 거래량이 크게 증가하고 결제 실패율이 감소한 것은 사용자 경험과 운영 효율성이 향상되었음을 나타내기도 하기에 
카카오 로그인 신규 기능을 도입한 것이 기능 개발로 유의미한 가치를 갖게되었다.