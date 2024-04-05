# JWT


## 쿠키와 세션

|           | 쿠키(Cookie)                                                           | 세션(Session)                                                     |
| --------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 저장 위치 | 클라이언트                                                             | 서버                                                              |
| 수명      | 지정된 만료 날짜까지 / 만료일을 설정하지 않으면 브라우저가 닫힐 때까지 | 기본적으로 브라우저가 닫힐 때. 사용자의 활동이 지속되는 동안 유지 |
| 용도      | 사용자의 환경 설정, 로그인 정보 등을 저장하는 데 주로 사용             | 사용자의 로그인 상태, 장바구니 정보 등을 저장하는 데 주로 사용    |


- 쿠키
  - 이름, 값, 만료일(저장 기간 설정), 경로 정보로 구성
  - 클라이언트에 총 300개의 쿠키
  - 하나의 도메인 당 20개의 쿠키
  - 하나의 쿠키는 4KB(=4096byte)까지
  - Response Header에 Set-Cookie

## http header

| Http header                                                                              |     |
| ---------------------------------------------------------------------------------------- | --- |
| [Authorization](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) |     |
| [Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie)               |     |

- Bearer
  - RFC 6750 The OAuth 2.0 Authorization Framework : Bearer Token Usage
  - RFC 7235 Hypertext Transfer Protocol (HTTP / 1.1) : Authentication을 참조하십시오.


## 쿠키기반

클라이언트 userid로 로그인
서버 userid기반 cookie_login생성 서버에 저장 및 클라이언트에 전송
클라이언트 cookie에 저장
클라이언트 요청시 cookie도 같이 전송



## JWT

- JSON Web Tokens are an open, industry standard RFC 7519 method for representing claims securely between two parties.
  - <Header(Base64)>.<Payload(Base64)>.<Signature(Base64)>
    - 점(`.`)으로 나뉘어 져 있다.

- 플로우
  - 로그인
    - username/password
    - password + salt => hasing => hash
    - payload { username: user.username, useruid: user.useruid },
  - 만들때
    - 클라에 넘겨줄 payload를 //secretkey//로 서명 알고리즘 alg로 서명
      - signature 얻고
      - header에 alg를 기록
      - Header.Payload.Signature를 조립하면 JWT완성
  - 확인할때
    - Header.Payload.Signature 에서
      - Payload를 base64로 디코딩
      - Header를 base64로 디코딩
      - 서버 secret으로 Header알고리즘을 이용 서명해서 Signature와 동일하면 인증 성공



| 주의                                                              |
| ----------------------------------------------------------------- |
| 클라가 alg를 none으로 넘겨주는 경우 확인                          |
| payload자체가 base64로 디코딩 가능하므로 민감한 정보 넣으면 안됨. |
| secretkey가 충분히 커야함.                                        |
| JWT 토큰 자체 탈취 가능성                                         |


|                     |                                                  |
| ------------------- | ------------------------------------------------ |
| HttpCookie.HttpOnly | 브라우저에서는 쿠키에 접근할 수 없도록 제한      |
| HttpCookie.Secure   | HTTPS가 아닌 통신에서는 쿠키를 전송하지 않습니다 |


``` txt


Header
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload
{
  "claim": value
  ...
}

claim 참고. RFC 7519
- registered
- public
- private


Signature
echo -n '<Header(Base64)>.<Payload(Base64)>' | \
  openssl dgst -binary -sha256 -hmac 'secret' | \
  base64
```


Post login
{username, password}
{access_token: JWT}

Post auth
Authorization: Bearer <access_token>
"Bearer"는 영어로 "소지인" 또는 "지참자"를 의미합니다. 원래는 사람이나 개체가 어떤 권한이나 권리를 소유하고 있는지 나타내는 용어입니다.



- https://www.youtube.com/watch?v=36lpDzQzVXs



## PASETO

Platform-Agnostic Security Tokens

- <https://paseto.io/>

|        |          |
| ------ | -------- |
| JWT    | SHA 계열 |
| PASETO | PBKDF2   |



https://github.com/P-oke/SocialAuthentication/blob/master/SocialAuthentication/GoogleAuthentication/GoogleAuthService.cs
https://github.com/joydipkanjilal/jwt-aspnetcore/blob/master/jwt-aspnetcore/Controllers/HomeController.cs