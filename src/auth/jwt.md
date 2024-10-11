# JWT



https://learn.microsoft.com/en-us/aspnet/core/security/authentication/?view=aspnetcore-8.0
https://learn.microsoft.com/en-us/aspnet/core/security/authorization/limitingidentitybyscheme?view=aspnetcore-8.0&viewFallbackFrom=aspnetcore-2.2&tabs=aspnetcore2x



https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-8.0&tabs=windows
프로젝트 간 공유 방지
dotnet user-secrets init
<PropertyGroup>
  <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>


dotnet user-secrets set "Movies:ServiceApiKey" "12345"
In Windows : %Appdata%\Microsoft\UserSecrets\<guid>\secrets.json
In Linux: ~/.microsoft/usersecrets/<guid>/secrets.json

C:\Users\pyoung\AppData\Roaming\Microsoft\UserSecrets\984b9acf-23e3-4cdc-99f4-f46c36877269\secrets.json
{
  "Movies:ServiceApiKey": "12345"
}




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
namespace System.IdentityModel.Tokens.Jwt;
// Summary:
//     List of registered claims from different sources https://datatracker.ietf.org/doc/html/rfc7519#section-4
//     http://openid.net/specs/openid-connect-core-1_0.html#IDToken
[StructLayout(LayoutKind.Sequential, Size = 1)]
public struct JwtRegisteredClaimNames


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


================================
## asp.net

순서 유지 중요.
UseAuthentication() 인증 - 사용자가 누구인지 확인
UseAuthorization() 권한부여 - 인증된 사용자가 특정 리소스에 접근할 수 있는지를 결정하는 정책을 설정
services.AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.Authority = "https://your-auth-server/";
        options.RequireHttpsMetadata = false;
        options.Audience = "your-audience";
    });

services.AddAuthorization(options =>
{
    options.AddPolicy("AuthenticatedUser", policy => policy.RequireAuthenticatedUser());
});
[Authorize(Policy = "AuthenticatedUser")]


## Authentication 인증 - 너가 누군지

https://docs.microsoft.com/en-us/aspnet/core/security/authentication/?view=aspnetcore-6.0#authentication-concepts
	
	Authentication scheme
	The default authentication scheme, discussed in the next section.
	Directly set HttpContext.User.


	
## Authorization 권한부여 - 어떠한 일을 할 수 있는지

[Authorize] attribute 사용가능.
ref: https://auth0.com/blog/securing-grpc-microservices-dotnet-core/


===============

ref
https://docs.microsoft.com/en-us/aspnet/core/grpc/authn-and-authz
https://docs.microsoft.com/en-us/aspnet/core/grpc/interceptors




## server

``` csharp
builder.Services.AddGrpc(opts =>
{
	opts.Interceptors.Add<LoggerInterceptor>();
});


// ==============================
public class LoggerInterceptor : Interceptor
    {
        private readonly ILogger<LoggerInterceptor> _logger;


        public LoggerInterceptor(ILogger<LoggerInterceptor> logger)
        {
            _logger = logger;
        }

        public override async Task<TResponse> UnaryServerHandler<TRequest, TResponse>(
            TRequest request,
            ServerCallContext context,
            UnaryServerMethod<TRequest, TResponse> continuation)
        {
            _logger.LogInformation($"before - {context.Method}");
            TResponse response;
            if (context.Method.EndsWith(nameof(WDService.RequestLogon)))
            {
                response = await base.UnaryServerHandler(request, context, continuation);
            }
            else
            {
                // Microsoft.AspNetCore.Http.IRequestCookieCollection cookie = context.GetHttpContext().Request.Cookies;
                var ctx = context.GetHttpContext();
                var header = context.GetHttpContext().Request.Headers;
                var x = header["Authorization"];
                Debug.Assert(x == "Bearer ");
                response = await base.UnaryServerHandler(request, context, continuation);
            }
            _logger.LogInformation($"after - {context.Method}");

            return response;
        }

        private void Log<TRequest, TResponse>(MethodType methodType, TRequest request, TResponse response,
            ServerCallContext context)
        {
            _logger.LogInformation(
                $"gRPC call. Type: {methodType}. Request: {request.ToString()}. Response: {response.ToString()}");
        }
    }
// ==============
public override async Task<R_RequestLogon> RequestLogon(Q_RequestLogon request, ServerCallContext context)
{
	Metadata m = new Metadata();
	m.Add("Authorization", "x");
	await context.WriteResponseHeadersAsync(m);

```

## client

``` csharp
Channel channel = new Channel("127.0.0.1:5299", ChannelCredentials.Insecure);
CallInvoker invoker = channel.Intercept(new ClientLoggingInterceptor());
WDService.WDServiceClient network = new WDService.WDServiceClient(invoker);

// ===============
  public class ClientLoggingInterceptor : Interceptor
    {
        public override AsyncUnaryCall<TResponse> AsyncUnaryCall<TRequest, TResponse>(
            TRequest request,
            ClientInterceptorContext<TRequest, TResponse> context,
            AsyncUnaryCallContinuation<TRequest, TResponse> continuation)
        {
            Console.WriteLine($"EEE - before {context.Method.Name}");

            if (context.Method.Name == nameof(WDService.WDServiceBase.RequestLogon))
            {
                jar = new CookieCollection();
            }
            else
            {
                Cookie cc = jar["Authorization"];
                // handle cc == null
                string ck = cc.Name;
                string cv = cc.Value;
                var m = new Metadata();
                m.Add("Authorization", "Bearer ");
                CallOptions op = context.Options.WithHeaders(m);
                context = new ClientInterceptorContext<TRequest, TResponse>(context.Method, context.Host, op);
            }

            AsyncUnaryCall<TResponse> call = continuation(request, context);
            return new AsyncUnaryCall<TResponse>(call.ResponseAsync, XY(call.ResponseHeadersAsync), call.GetStatus, call.GetTrailers, call.Dispose);
        }

        private async Task<Metadata> XY(Task<Metadata> responseHeadersAsync)
        {
            var meta = await responseHeadersAsync;
            Metadata.Entry e = meta.Get("Authorization");
            if (e != null)
            {
                var k = e.Key;
                var v = e.Value;
                jar.Add(new Cookie(k, v));
            }
            return meta;
        }


        CookieCollection jar = new CookieCollection(); //header jar
    }
```

==================================
==================================
https://github.com/jwt-dotnet/jwt
https://www.nuget.org/packages/JWT/8.9.0
		<PackageReference Include="JWT" Version="8.9.0" />

https://jasonwatmore.com/post/2021/06/02/net-5-create-and-validate-jwt-tokens-use-custom-jwt-middleware
https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt/
https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet
<PackageReference Include="System.IdentityModel.Tokens.Jwt" Version="6.17.0" />


https://gist.github.com/pradeepn/89646bd37de3bb3535de5582f0959610
https://jasonwatmore.com/post/2020/05/25/aspnet-core-3-api-jwt-authentication-with-refresh-tokens



https://velog.io/@tlatldms/%EC%84%9C%EB%B2%84%EA%B0%9C%EB%B0%9C%EC%BA%A0%ED%94%84-Refresh-JWT-%EA%B5%AC%ED%98%84