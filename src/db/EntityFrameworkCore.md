EF(Entity Framework) Core

2008 - Entity Framework 1.0 (EF 1.0)  .NET Framework 3.5 SP1
2016 Entity Framework의 완전한 재설계 버전. 기존 EF6에서 더 경량화된 버전으로 변경되었으며, .NET Core와 호환.


ORM(Object Relational Mapping)




// https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/required


SQL Server Express LocalDB
https://learn.microsoft.com/ko-kr/sql/database-engine/configure-windows/sql-server-express-localdb?view=sql-server-ver16

SSMS(SQL Server Management Studio)  https://learn.microsoft.com/ko-kr/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16
DBeaver https://dbeaver.io/download/

SqlLocalDB 
https://learn.microsoft.com/ko-kr/sql/tools/sqllocaldb-utility?view=sql-server-ver16



SQL Server
LocalDB는 SQL Server의 경량 버전으로 간편한 데이터베이스 개발과 테스트 용도로 설계되었습니다.

Databases/ 
Security/
    Logins: 서버에 접근할 수 있는 사용자 계정 및 인증 정보를 관리합니다.
    Server Roles: 서버에서 역할 기반의 권한을 설정할 수 있습니다.
    Credentials: 외부 시스템과 연결될 때 사용하는 보안 정보를 관리합니다.
 Server Objects/
    Endpoints: SQL Server와 외부 애플리케이션 간 통신을 설정하는 엔드포인트를 관리합니다.
    Linked Servers: 다른 서버와의 연결 설정을 관리합니다.
    Triggers: 서버 수준에서 발생하는 이벤트에 대한 트리거를 설정할 수 있습니다.
    Backup Devices: 백업 장치와 관련된 설정을 관리합니다.


Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=EfCoreDb;Integrated Security=True;Connect Timeout=30;Encrypt=False;Trust Server Certificate=False;Application Intent=ReadWrite;Multi Subnet Failover=False

Data Source=(localdb)\MSSQLLocalDB
연결할 SQL Server의 인스턴스를 지정합니다.

Initial Catalog=EfCoreDb
의미: 연결하려는 데이터베이스의 이름을 지정합니다.

Integrated Security=True
Windows 인증을 사용하여 SQL Server에 연결하도록 설정합니다.

Connect Timeout=30
30초 동안 SQL Server에 연결을 시도하며, 만약 30초 내에 연결되지 않으면 타임아웃 에러가 발생합니다.

Encrypt=False
데이터베이스 연결 시 데이터 암호화 여부를 설정합니다.

Trust Server Certificate=False
False로 설정되면, 암호화된 연결이 설정된 경우 SQL Server의 인증서가 신뢰할 수 있는 CA(Certificate Authority)에서 발급된 것인지 확인합니다.
True로 설정하면 서버 인증서를 검증하지 않고 연결을 허용합니다.

Application Intent=ReadWrite
ReadWrite // ReadOnly

Multi Subnet Failover=False
다중 서브넷 환경에서 장애 조치(Failover) 성능을 최적화할지 여부를 설정합니다.



``` csharp
internal class AppDbContext : DbContext
{
    public const string CONNECTION_STRING = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=EfCoreDb;Integrated Security=True;Connect Timeout=30;Encrypt=False;Trust Server Certificate=False;Application Intent=ReadWrite;Multi Subnet Failover=False";

    public DbSet<Item> Items { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(CONNECTION_STRING);
        if (_isLogging)
        {
            // optionsBuilder.UseLoggerFactory(
            optionsBuilder.LogTo(
                Console.WriteLine,
                new[] {
                    RelationalEventId.CommandExecuting
                });
        }
    }
    

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // db.DTs_Item.IgnoreQueryFilters() 와 같이 쿼리 필터를 무시할 수 있다.
        modelBuilder.Entity<DTs_Item>().HasQueryFilter(i => i.SoftDeleted == false);
    }
}

static bool InitializeDb(bool forceReset)
{
    using (AppDbContext db = new AppDbContext())
    {
        if (!forceReset)
        {
            RelationalDatabaseCreator? creator = db.GetService<IDatabaseCreator>() as RelationalDatabaseCreator;
            if (creator is null)
            {
                Console.Error.WriteLine("creator is null");
                return false;
            }

            if (creator.Exists())
            {
                Console.Error.WriteLine("creator is exist");
                return false;
            }
        }

        db.Database.EnsureDeleted();
        db.Database.EnsureCreated();

        return true;
    }
}

db.SaveChanges();





DTs_Guild guild = db.DTs_Guild
    .Where(g => g.GuildName == name)
    .Include(g => g.Members)
    .ThenInclude(p => p.Items)
    .AsNoTracking()
    .First();

DTs_Guild guild = db.DTs_Guild
    .Where(x => x.GuildName == name)
    .First();
db.Entry(guild).Collection(x => x.Members).Load();
DTs_Player player = guild.Members.First();
db.Entry(player).Reference(p => p.Items).Load();



EntityState state = db.Entry(player1).State;
Detached | Unchanged | Deleted | Modified | Added
```


Microsoft.EntityFrameworkCore.Sqlite
https://learn.microsoft.com/ko-kr/ef/core/providers/sqlite/limitations


https://learn.microsoft.com/ko-kr/ef/core/modeling/keys?tabs=data-annotations
Key 어트리뷰트: 이 어트리뷰트는 Entity Framework 6 및 Entity Framework Core에서 사용됩니다. 기본 키를 지정하는 가장 일반적인 방법입니다. 예를 들어:
PrimaryKey 어트리뷰트: 이 어트리뷰트는 Entity Framework Core 7.0에서 도입되었습니다 복합 키를 지정할 때 유용합니다. 예를 들어:


AsNoTracking() // readonly
    AsNoTracking은 엔티티 추적을 비활성화하므로, 이후 변경된 데이터는 SaveChanges를 호출하더라도 데이터베이스에 적용되지 않습니다.
    AsNoTracking을 사용하면 동일한 엔티티를 여러 번 조회할 때, EF Core는 이를 서로 다른 객체로 인식합니다. 이는 데이터 일관성에 문제가 될 수 있으며, 추적되지 않은 엔티티를 서로 연결하려고 할 때 예상치 못한 동작이나 예외가 발생할 수 있습니다.
    예시:

- 쿼리 단위 설정
  - AsNoTracking() vs. AsTracking()
  - AsNoTrackingWithIdentityResolution() (EF Core 5)
- Context 인스턴스 수준 설정
  - context.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking
- 전체 기본 동작으로 설정
  - optionBuilder.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking) - (insert/update/delete) 작업 시 개별적으로 AsTracking() 

    csharp
    코드 복사
    var player1 = context.Players.AsNoTracking().FirstOrDefault(p => p.Id == 1);
    var player2 = context.Players.AsNoTracking().FirstOrDefault(p => p.Id == 1);

    // player1과 player2는 서로 다른 객체로 인식됨

.Include() Eager Loading 즉시 로드  - FK를 사용


외래키가 성능을 저하시킬 수 있는 주요 이유는 삽입/삭제 시 참조 무결성 검증 때문이며, 복잡한 트랜잭션에서도 성능 이슈가 발생할 수 있습니다.


foreach (Item item in db.Items.AsNoTracking().Include(i => i.Owner))

db.Items.Include(x => x.Owner).Where(x => x.Owner.Name == name);



[Table("Player", Schema = "sports")]
SQL Server는 항상 스키마를 사용하며, dbo 스키마는 기본값입니다
데이터베이스 객체들을 그룹화하고 논리적으로 구성하는 네임스페이스 개념
dbo는 "Database Owner"의 약자로, SQL Server의 기본 스키마로 설정되어 있습니다. 테이블을 만들 때 별도로 스키마를 지정하지 않으면 기본적으로 dbo 스키마가 적용됩니다.


테이블 이름 - 언더스코어?
복수형 테이블 이름은 예약된 키워드와 충돌할 가능성이 적습니다.
    저는 팀에 속해 있는데, 다른 테이블에서 사용될 때는 기본 키로 id를 사용하고 외래 키로 tablename_id를 사용합니다.
ID에 전체 이름을
https://dev.to/ovid/database-naming-standards-2061






DC_ Client Master DB
DS_ Server DB
Client Logic Class
Common Logic Class
Server Logic Class
유통 DTO(Data Transfer Object)

Q_ request
R_ response

DT // DataTable
DS DTs // Db Server

DC DTc // Db Client
DC_Player DTc_Player


DPlayer D_Player // DbPlayer //
GD_Player // GameDbPlayer
SD_Player // ServerDbPlayer
CB_Player // ClientDbPlayer

Ref_D_Player// public int LinkedPlayerId { get; set; } // Player 테이블과의 연결을 명확히



https://learn.microsoft.com/ko-kr/ef/core/querying/tracking

로깅
https://learn.microsoft.com/ko-kr/ef/core/logging-events-diagnostics/
    https://learn.microsoft.com/ko-kr/ef/core/logging-events-diagnostics/diagnostic-listeners


| .NET     | sqlserver      |
| -------- | -------------- |
| string   | nvarchar(max)  | SQL Server 2019 이상을 사용하고 있고, UTF-8 인코딩을 원하면 varchar를 사용하세요. |
| bool     | bit            |
| byte     | tinyint        |
| int      | int            |
| int64    | bigint         |
| decimal  | decimal(18, 2) |
| datetime | datetime2(7)   | date                                                                              |

datetime2
0001-01-01부터 31.12.99까지
9999년 1월 1일 CE~9999년 12월 31일
00:00:00~23:59:59.99999999

date
0001-01-01 through 9999-12-31


[Table(nameof(DTs_Item))]
[PrimaryKey(nameof(ItemUID))]
[Index(nameof(Id), IsUnique = true)]
[Required]
[Column(TypeName = "varchar(30)")]
[MaxLength(20)]
[NotMapped]
[ForeignKey]
[InverseProperty]
[DbFunction]
[Unicode(false)]
[DatabaseGenerated(DatabaseGeneratedOption.Identity)]

azure cli
https://learn.microsoft.com/en-us/cli/azure/


[DbFunction()]
public static double? GetAverageReviewScore(int itemId)
{
    throw new NotImplementedException("DbFunction");
}

string command =
    @" CREATE FUNCTION GetAverageReviewScore (@itemId INT) RETURN FLOAT
        AS
        BEGIN

        DECLARE @result AS FLOAT

        SELECT @result =  AVG(CAST([Score] AS FLOAT))
        FROM ItemReview As r
        WHERE @itemId = r.ItemId

        RETURN @result

        END
    ";
db.Database.ExecuteSqlRaw(command);


modelBuilder.Entity<DTs_Item>().Property("Property1").HasDefaultValue(1);
modelBuilder.Entity<DTs_Item>().Property("Property1").HasDefaultValueSql("GETDATE()");


``` cs
public class PlayerNameGenerator : ValueGenerator<string>
{
    public override bool GeneratesTemporaryValues => false;

    public override string Next(EntityEntry entry)
    {
        string name = $"Player_{DateTime.Now.ToString("yyyyMMdd")}";
        return name;
    }
}
builder.Entity<Player>()
    .Property(p => p.Name)
    .HasValueGenerator((p, e) => new PlayerNameGenerator());
```

================

## Migration

### 코드 중심

- 마이그레이션 병합
  - dotnet tool install --global dotnet-ef
  - dotnet ef migrations add MigrationName
  - 기존 마이그레이션 삭제
  - dotnet ef database update


#### Migration 생성
Tools > NuGet Package Management > Package Management Console
Microsoft.EntityFrameworkCore.Tools

PM> Add-Migration [Name]
PM> Add-Migration Hello
DbContext -> DB 모델링
~ModelSnapshot.cs를 이용해서 가장 마지막 Migration 상태의 DB 모델링
비교 결과 도출

~ModelSnapshot.cs : ModelSnapshot
222211000000_Hello.cs // up/down
    222211000000_Hello.Designer.cs

#### 

sql 스크립트 생성 후 적용
PM> Script-Migration [From] [To] [Options] 

Command Line 으로 적용
PM> Update-Database [Options]

특정 Migration으로 Sync
PM> Update-Database [Name]

최신 Migration 삭제
PM> Remove-Migration 

### DB 중심
https://learn.microsoft.com/ko-kr/ef/core/extensions/
https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools
    역엔지니어링 db에서 class 생성

====================


트랜잭션 사용
https://learn.microsoft.com/ko-kr/ef/core/saving/transactions

ChangeTracker
https://learn.microsoft.com/ko-kr/ef/core/change-tracking/
상태변화 감지

TrackGraph
Relationship이 있는 Untracked Entity의 State 조작. (ex. 전체 데이터 중 일부만 갱신)

``` cs
using (AppDbContext db = new AppDbContext(true))
{
    DTs_Item item = new DTs_Item();
    item.ItemUID = 1;
    item.TemplateId = 111;

    db.ChangeTracker.TrackGraph(
        item,
        e =>
        {
            if (e.Entry.Entity == item)
            {
                e.Entry.State = EntityState.Unchanged;
                e.Entry.Property(nameof(DTs_Item.TemplateId)).IsModified = true;
            }
        });
    db.SaveChanges();
}
```


``` cs
: DbContext

public override int SaveChanges()
{
    IEnumerable<EntityEntry> entities = ChangeTracker.Entries().Where(e => e.State == EntityState.Added);
    foreach (EntityEntry e in entities)
    {
        object entity = e.Entity;
        DTs_Item item = entity as DTs_Item;
        if (item != null)
        {
        }
    }
    return base.SaveChanges();
}
```

=====

https://learn.microsoft.com/en-us/ef/core/querying/sql-queries

SqlClient.SqlParameter 매개변수 유형, 길이 집적 지정 보안상 보다 안정

EF Core 7 FromSql
    EF Core 6 FromSqlRaw
        런타임 동적 쿼리 구성가능하지만, SQL Injection등 보안문제 취약
    EF Core 6 FromSqlInterpolated
EF Core 7 database.SqlQuery
    스칼라값 반환 - 최종값 별칭은 Value
EF Core 7 ExecuteSql 실행 행수 리턴
    EF Core 6 ExecuteSqlRaw / ExecuteSqlInterpolated
Reload

``` cs
IQueryable<DTs_Item> a = db.DTs_Item.FromSqlRaw("SELECT * FROM dbo.DTs_Item WHERE ItemUID = {0}", uid);
IQueryable<DTs_Item> b = db.DTs_Item.FromSqlInterpolated($"SELECT * FROM dbo.DTs_Item WHERE ItemUID = {uid}");

db.Database.ExecuteSqlInterpolated($"UPDATE dbo.DTs_Item SET TemplateId = {101} WHERE ItemUID = {uid}");
db.Entry(item).Reload();


 SqlParameter p_phone = new SqlParameter("Phone", SqlDbType.VarChar, 30) { Value = "23-768-687-3665" };

 using (SalesContext db = new SalesContext())
 {
     var customers = await db.Customers
         .FromSql($@"SELECT * FROM dbo.Customers AS c WHERE Phone = {p_phone}")
         .AsNoTracking()
         .ToListAsync();
    customers = await db.Customers
        .FromSqlRaw($@"SELECT * FROM dbo.Customers AS c WHERE Phone = @Phone", p_phone)
        .AsNoTracking()
        .ToListAsync();
```

=======================================

``` cs
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // 개별 context 인스턴스에서 Lambda/Linq 사용 시 호출
        if (!optionsBuilder.IsConfigured)
        {
            optionsBuilder
                //.UseLazyLoadingProxies(true)  //테스트를 위해 디폴트로 True 적용
                //.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking)
                .UseSqlServer(connectionString, sqloptions =>
                {
                    //sqloptions.CommandTimeout(30);
                    //sqloptions.MaxBatchSize(42);
                    //sqloptions.EnableRetryOnFailure(maxRetryCount: 5, maxRetryDelay: TimeSpan.FromSeconds(30), errorNumbersToAdd: null);
                    //sqloptions.UseQuerySplittingBehavior(QuerySplittingBehavior.SingleQuery);
                })
                .AddInterceptors(new CommandInterceptorForHint())   //테스트용
#if DEBUG
                //Logging
                .LogTo(
                      Console.WriteLine
                    , LogLevel.Information	//Information < Debug(connection, transaction 등 정보까지) < Trace
                    , DbContextLoggerOptions.DefaultWithLocalTime
                    )

                //쿼리 매개변수 정보도 포함하고 싶은 경우
                //  주의) 아래 옵션은 운영에서 사용하지 말 것(개발 또는 테스팅 환경)
                .EnableSensitiveDataLogging(true)

                // 필드 수준의 상세한 오류 정보
                //.EnableDetailedErrors();  
#endif
                ;
        }
    }
```


SQL Server Profiler
expressprofiler - https://github.com/OleksiiKovalov/expressprofiler

PostgreSQL에서 SQL Server Profiler와 유사한 쿼리 모니터링 도구를 사용하려면 pg_stat_statements와 pgBadger가 가장 추천됩니다


BenchmarkDotNet


### Connection string
Connection Timeout default 15 - azure 30
Encrypt EF Core 7부터 True가 디폴트
Pooling=True;
MultipleActiveResultSets=True;
Application Name


### builder
Command Timeout
DbContextPool(EF Core 2) poolSize default 1024 (EF Core 6)

ServiceCollection services = new ServiceCollection(); // ConfigureServices()에서는 매개변수로 제공
services.AddDbContextPool<SalesContext>(
    options =>
    {
        options.UseSqlServer(connectionString);
    },
    poolSize: 1024
);
```
 static void Main(string[] args)
    {
        // 서비스 컬렉션 생성
        var serviceCollection = new ServiceCollection();
        ConfigureServices(serviceCollection);

        // 서비스 제공자 생성
        var serviceProvider = serviceCollection.BuildServiceProvider();

        // DbContext 사용 예제
        using (var scope = serviceProvider.CreateScope())
        {
            var context = scope.ServiceProvider.GetRequiredService<MyDbContext>();
        }
    }

    private static void ConfigureServices(IServiceCollection services)
    {
        // DbContext 풀링 설정
        services.AddDbContextPool<MyDbContext>(options =>
            options.UseSqlServer("YourConnectionStringHere")); // 실제 연결 문자열로 변경
    }
```


.TagWith("tag1") 로 주석 태그를 남길 수 있다.


// 1 - Adhoc
.Where(c => c.Phone == "30-114-968-4951")
// 2 - Parameter 권고 성능 query tree 의 캐시
.Where(c => c.Phone == p_phone)


ToQueryString(this IQueryable source) // EF Core 5 부터


IQueryable<DTs_Item> q = (new AppDbContext).DTs_Item; // stream
IEnumerable<DTs_Item> q = (new AppDbContext).DTs_Item; // buffering - 이미 이 시점에 쿼리가 실행됨.


PK 열검사 - FindAsync  - AsNoTracking() 비호환
First TOP(1)
SingleOrDefault TOP(2) // 사용금지

EF.Functions
https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbfunctions?view=efcore-8.0

https://learn.microsoft.com/ko-kr/ef/core/providers/sql-server/functions


|                                                                           |           | sqlserver    |
| ------------------------------------------------------------------------- | --------- | ------------ |
| EF. Functions.StandardDeviationSample(group. Select(x => x.Property))     | EF Core 7 | STDEV()      |
| EF. Functions.StandardDeviationPopulation(group. Select(x => x.Property)) | EF Core7  | STDEVP()     |
| EF. Functions.VarianceSample(group. Select(x => x.Property))              | EF Core 7 | VAR()        |
| EF. Functions.VariancePopulation(group. Select(x => x.Property))          | EF Core 7 | VARP()       |
| string.Join(separator, group. Select(x => x.Property))                    | EF Core 7 | STRING_AGG() |
| DateTime.Now                                                              |           | GETDATE()    |
| EF. Functions.Random()                                                    | EF Core 6 | RAND()       |
| EF.Functions.Like(matchExpression, pattern)                               |           | LIKE         |
| collection.Contains(item)                                                 |           | IN           |


int[] custkeys = new int[] { 1, 2, 3, 4, 5 };
List<Customer> customers = await db.Customers…….Where(c => custkeys.Contains(c.CustKey)).ToListAsync();
// WHERE [c].[CustKey] IN (1, 2, 3, 4, 5)


var query = from r in db.Regions.Where(r => r.RegionKey == p_region)
from n in db.Nations
select new
{
r.RegionKey, n.NationKey
};

exec sp_executesql N'SELECT [r].[RegionKey], [n].[NationKey]
FROM [Regions] AS [r]
CROSS JOIN [Nations] AS [n]
WHERE [r].[RegionKey] = @__p_region_0',N'@__p_region_0 int',@__p_region_0=1


var query =
from c in db.Customers
join o in db.Orders on c.CustKey equals o.CustKey
where o.OrderKey <= 2
select new
{
c.CustKey,
c.Name,
o.OrderDate
};
SELECT [c].[CustKey], [c].[Name], [o].[OrderDate]
FROM [Customers] AS [c]
INNER JOIN [Orders] AS [o] ON [c].[CustKey] = [o].[CustKey]
WHERE [o].[OrderKey] <= 2



====

- __금지__ 인덱스 타는걸 방해하는 행위
  - 인덱스 열의 사칙연산
    - Where(o => o.CustKey + 1 == custkey)
  - 인덱스 열의 함수 적용
    - .Where(o => o.OrderDate.AddDays(10) == p_orderdate)
  - 인덱스 열의 암시적 형 변환
    - 문자열 주의
      - ef unicode // db varchar
  - LIKE 조건이 wildcard 문자로 시작하는 경우
    - .Where(c => c.Phone.Contains("%010%")) // '%010%' 앞에 %가 와서 인덱스 못타게됨
    - .Where(c => EF.Functions.Like(c.Phone, "010%")) // EF.Functions 활용 권장
  - 열 간 비교
    - .Where(o => o.CustomerKey == (p_custkey ?? o.CustomerKey))


https://learn.microsoft.com/ko-kr/ef/core/performance/advanced-performance-topics?tabs=with-di%2Cexpression-api-with-constant#compiled-queries
public static Func<SalesContext, int, IAsyncEnumerable<Customer>> getCustByKey =
    EF.CompileAsyncQuery(
        (SalesContext db, int custkey) =>
            db.Customers
                .Include(c => c.Orders)
                .Where(c => c.CustKey == custkey)
);

using (SalesContext db = new SalesContext())
{
    await foreach (Customer c in SalesContext.getCustByKey(db, p_custkey))
    {
        Console.WriteLine($"CustKey: {c.CustKey}, Phone: {c.Phone}");
        Console.WriteLine(new string('-', 20));
    }
}





CancellationTokenSource cancelquery = new CancellationTokenSource();
cancelquery.CancelAfter(10 * 1000);
try
{
    .ToListAsync(cancelquery.Token)
}
catch (TaskCanceledException ex) { }
catch (SqlException ex) //when (ex.Number == -2) {}
catch (Exception ex) {}

===============================

``` cs
// TVF Table Value Function
// https://learn.microsoft.com/en-us/ef/ef6/modeling/designer/advanced/tvfs
// ALTER   FUNCTION [dbo].[GetOrdersList]
// (
//     @p_OrderKey int
// )
// RETURNS TABLE AS RETURN
// (
//     SELECT *
//     FROM dbo.Orders AS o
//     WHERE OrderKey < @p_OrderKey
// )

public IQueryable<Order> GetOrdersList(int OrderKey)
{
    return FromExpression(() => GetOrdersList(OrderKey));
}

```

``` cs
 FormattableString query;
 query = $@"SELECT p.RetailPrice - pp.Average AS Value 
             FROM dbo.Parts AS p
             CROSS JOIN (SELECT AVG(pp.RetailPrice) AS Average FROM dbo.Parts pp) AS pp
             WHERE p.PartKey <= {p_partkey}";

 using (SalesContext db = new SalesContext())
 {
     // A) SqlQuery()
     var prices = await db
         .Database
         .SqlQuery<decimal>(query)
         .OrderBy(p => p)
         .Take(5)
         .ToListAsync()
         ;
```



=======
using (var transaction = context.Database.BeginTransaction())
{
    try
    {
        ...
        transaction.Commit();
    }
    catch (Exception ex)
    {
        transaction.Rollback();
    }
}

using (SalesContext db = new SalesContext())
{
    var order = await db.Orders.FindAsync(orderkey);
    order.TotalPrice -= 10000;

    using (var tx = await db.Database.BeginTransactionAsync())
    {
        tx.CreateSavepoint("BeforeUpdate");

        var rowsAffected = await db.SaveChangesAsync();
        Console.WriteLine($"{rowsAffected} rows affected");

        await tx.RollbackToSavepointAsync("BeforeUpdate");
        await tx.CommitAsync();
    }
}

데이터 대량 변경 ExecuteUpdate / ExecuteDelete 권고. 필요시 sql.


ReadCommitted 기본값
데이터 변경(Update/Delete) 시 동일 리소스 접근하는 작업은 트랜잭션 완료까지 대기
SELECT 잠금은 Column 단위로.

RepeatableRead 격리 수준 상향,
SELECT 잠금이 트랜잭션 완료 시 까지 유지

using (var tx = await db.Database.BeginTransactionAsync(System.Data.IsolationLevel.ReadCommitted))

    public enum IsolationLevel
    {
        Unspecified = -1,
        Chaos = 16,
        ReadUncommitted = 256,
        ReadCommitted = 4096,
        RepeatableRead = 65536,
        Serializable = 1048576,
        Snapshot = 16777216
    }


- 읽기시
  - ReadUncommitted 변경중 데이터 읽기 허용 - 데이터 무결성 문제
    - 가장 많이 사용( 특히 국내 )
  - RCS - READ COMMITED SNAPSHOT 적용
    - 대부분의 RDBMS의 기본 동작
      - Row Versioning or MVCC(Multi-Version Concurrency Control)
    - 변경전 데이터 읽기

``` sql
-- RCS 설정 : SMS 사용해서 설정하거나 sql 사용

-- sql
USE [master]
GO
ALTER DATABASE [SalesSimple]
SET READ_COMMITTED_SNAPSHOT ON
```

================
분산 트렌젝션
/// 분산 트랜잭션 - System.Transactions.TransactionScope
행의 범위 전체 잠금
//비동기 처리 시 TransactionScopeAsyncFlowOption.Enabled 지정

``` cs
static async Task DistributedTransaction_Async_Default()
{
    var p_orderkey = 10;
    List<Order> orders;

    //비동기 처리 시 TransactionScopeAsyncFlowOption.Enabled 지정
    //  Complete/Rollback 전에 Connection이 Logout/Login 후에 처리됨
    using (TransactionScope tx = new TransactionScope(TransactionScopeOption.Required,
                                                      TransactionScopeAsyncFlowOption.Enabled))
    {
        using (SalesContext db = new SalesContext())
        {
            // 비동기 처리 기준
            orders = await db
                .Orders
                .Where(o => o.OrderKey <= p_orderkey)
                .ToListAsync();

        }//using db

        //트랜잭션 완료
        tx.Complete();

        // 세션 격리수준 확인용
        Console.WriteLine($"Completed, any key to continume..."); Console.ReadKey();

    }//using tx
}


```

.Complete하고 scope가 종료되야 비로서 커밋이됨(Ambient Transaciton이기에 )
.Dispose시 롤백





DbCommandInterceptor(IDbCommandInterceptor) : 생성, 실행, 명령 실패, DbDataReader 명령 Dispose
DbConnectionInterceptor(IDbConnectionInterceptor) : 연결 및 닫기, 연결 실패
DbTransactionInterceptor(IDbTransactionInterceptor) : 생성, 사용, 커밋, 롤벡, 생성 및 세이브포인트 사용, 트렌젝션 실패

protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
        optionsBuilder
                .AddInterceptors(new CommandInterceptorForHint())

public partial class CommandInterceptorForHint : DbCommandInterceptor
{
    public override ValueTask<InterceptionResult<DbDataReader>> ReaderExecutingAsync(DbCommand command, CommandEventData eventData, InterceptionResult<DbDataReader> result, CancellationToken cancellationToken = default)
    {
        return base.ReaderExecutingAsync(command, eventData, result, cancellationToken);





modelBuilder.Entity<YourEntity>()
    .Property(e => e.YourDateTimeProperty)
    .HasConversion(
        v => DateTimeOffset.FromUnixTimeSeconds(v).UtcDateTime, 
        v => ((DateTimeOffset)v).ToUnixTimeSeconds()
    );


[AttributeUsage(AttributeTargets.Property, AllowMultiple = false)]
public class UnixTimeAttribute : Attribute
{
}
public class YourEntity
{
    [UnixTime]
    public DateTime YourDateTimeProperty { get; set; }
}
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    foreach (var entityType in modelBuilder.Model.GetEntityTypes())
    {
        foreach (var property in entityType.GetProperties())
        {
            var propertyInfo = property.PropertyInfo;
            if (propertyInfo != null && propertyInfo.GetCustomAttributes(typeof(UnixTimeAttribute), true).Any())
            {
                // long 값을 Unix Time으로 저장하고 DateTime으로 변환하는 변환기
                property.SetValueConverter(new ValueConverter<long, DateTime>(
                    v => DateTimeOffset.FromUnixTimeSeconds(v).UtcDateTime,
                    v => ((DateTimeOffset)v).ToUnixTimeSeconds()
                ));
            }
        }
    }
}
