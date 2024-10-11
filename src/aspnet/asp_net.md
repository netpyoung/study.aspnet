appSettings.json
launchSettings.json

builder.Services는 애플리케이션이 시작될 때 서비스를 등록하는 데 사용됩니다.
app.Services는 애플리케이션이 실행된 후 이미 등록된 서비스를 가져와서 사용하고자 할 때 사용됩니다

ASPNETCORE_ENVIRONMENT  환경변수
appSettings.<ASPNETCORE_ENVIRONMENT>.json

IConfigurationSection section = builder.Configuration.GetSection("Section1");
int val = section.GetValue<int>("A");

- `[Route]` 어트리뷰트를 사용하여 경로를 설정할 수 있음.
- 실제로 해당 경로에 따라 요청을 분기하는 것은 UseRouting이 담당
- UseEndpoints()	엔드포인트 라우팅을 설정하여 최종적으로 요청을 처리할 특정 엔드포인트에 연결합니다.
- 엔드포인트는 최종적으로 요청을 처리하는 대상(컨트롤러, 페이지 등)입니다.



UseExceptionHandler("/Home/Error");
UseHsts();


UseHttpsRedirection();

MapAreaControllerRoute
MapControllerRoute

- 순서 유지 중요.
  - UseStaticFiles(); UseRouting 앞에 UseStaticFiles를 배치
  - UseRouting과 UseEndPoints 사이에 인증 및 권한 부여 미들웨어를 배치하는 것이 중요
  - UseRouting 요청이 라우팅 테이블을 따라 적절한 컨트롤러로 전달되도록 합니다.
  - UseAuthentication() 인증 - 사용자가 누구인지 확인
  - UseAuthorization() 권한부여 - 인증된 사용자가 특정 리소스에 접근할 수 있는지를 결정하는 정책을 설정
  - UseEndpoints는 엔드포인트 실행을 미들웨어 파이프라인에 추가합니다. 선택한 엔드포인트와 연결된 대리자를 실행합니다.

EndpointRoutingMiddleware



UseDeveloperExceptionPage
UseSwagger
UseSwaggerUI

UseExceptionHandler("/Home/Error")	예외 처리 미들웨어를 설정합니다. 지정된 경로로 리디렉션하여 예외 상황에 대한 사용자 친화적인 메시지를 표시합니다.

UseHsts()	HSTS(HTTP Strict Transport Security)를 활성화하여 HTTPS를 통한 보안을 강화합니다.
UseHttpsRedirection()	HTTP 요청을 HTTPS로 리디렉션하는 미들웨어를 추가합니다.
UseCors()	CORS(Cross-Origin Resource Sharing)를 설정하여 특정 도메인에서의 리소스 공유를 허용합니다.

UseStaticFiles()	정적 파일(예: HTML, CSS, JavaScript 등)을 제공하는 미들웨어를 설정합니다.
- UseEndpoints()	엔드포인트 라우팅을 설정하여 최종적으로 요청을 처리할 특정 엔드포인트에 연결합니다.
UseMiddleware<T>()	커스텀 미들웨어를 설정하는 메서드입니다. T는 미들웨어 클래스 타입을 의미합니다.
UseSession()	    세션을 사용하기 위한 미들웨어를 설정합니다. 세션 데이터는 서버에서 유지됩니다.




메서드	설명
app.MapControllerRoute()	컨트롤러 기반의 라우트를 정의합니다. 주로 MVC 패턴을 사용하는 애플리케이션에서 사용됩니다.
app.MapDefaultControllerRoute()	기본적인 컨트롤러 라우팅 패턴을 설정합니다. (예: /{controller=Home}/{action=Index}/{id?}).
app.MapRazorPages()	Razor Pages 라우팅을 설정합니다. Razor Pages는 페이지 중심의 웹 애플리케이션을 구축할 때 사용됩니다.
app.MapGet()	단순 HTTP GET 요청에 대한 엔드포인트를 정의합니다.
app.MapPost()	HTTP POST 요청에 대한 엔드포인트를 정의합니다.
app.MapPut()	HTTP PUT 요청에 대한 엔드포인트를 정의합니다.
app.MapDelete()	HTTP DELETE 요청에 대한 엔드포인트를 정의합니다.
app.MapFallback()	요청에 대해 다른 라우트가 없을 때 실행될 "백업" 라우트를 설정합니다.
app.MapWhen()	특정 조건이 충족될 때만 실행되는 라우트를 정의합니다.
app.MapHub<T>()	SignalR 허브를 설정하여 실시간 웹 기능을 구현합니다. T는 허브 클래스의 타입을 의미합니다.
app.MapBlazorHub()	Blazor 서버 애플리케이션에 필요한 SignalR 허브를 설정합니다.