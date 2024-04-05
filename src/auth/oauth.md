# OAuth

- [[10분 테코톡] 홍실의 OAuth 2.0](https://www.youtube.com/watch?v=Mh3LaHmA21I)
- [opentutorial: WEB2 - OAuth 2.0](https://opentutorials.org/course/3405)
- [rfc: The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)


``` mermaid
sequenceDiagram
    participant ResourceOwner
    participant UserAgent
    participant Client
    participant AuthorizationServer

    ResourceOwner->>Client: OAuth 요청
    Client ->> UserAgent : 로그인 URL 제공
    UserAgent ->> AuthorizationServer : 로그인 페이지 접근
    AuthorizationServer ->> UserAgent : 로그인 페이지 반환
    ResourceOwner ->> UserAgent : 인증 수행과 scope지정
    UserAgent ->> AuthorizationServer : 인증 수행과 scope지정
    AuthorizationServer ->> Client : Authorization Code 반환
    Client ->> AuthorizationServer : AccessToken 발급 요청
    AuthorizationServer ->> Client : AccessToken 반환

```

## 

``` txt
useruid | oauth_provider_id | oauth_provider_token |
oauth_provider_id | providername | wsendpoint|
```
