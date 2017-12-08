---
title: "カスタムの移行操作 - EF コア"
author: bricelam
ms.author: bricelam
ms.date: 11/7/2017
ms.technology: entity-framework-core
ms.openlocfilehash: d41409dee034e84d22092a5f9111dd79c87dcec3
ms.sourcegitcommit: b467368cc350e6059fdc0949e042a41cb11e61d9
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/15/2017
---
<a name="custom-migrations-operations"></a><span data-ttu-id="e79c4-102">カスタムの移行操作</span><span class="sxs-lookup"><span data-stu-id="e79c4-102">Custom Migrations Operations</span></span>
============================
<span data-ttu-id="e79c4-103">MigrationBuilder API では、移行中にさまざまな種類の操作を実行することができますが、排他的なかけ離れていること。</span><span class="sxs-lookup"><span data-stu-id="e79c4-103">The MigrationBuilder API allows you to perform many different kinds of operations during a migration, but it's far from exhaustive.</span></span> <span data-ttu-id="e79c4-104">ただし、また API は、拡張を独自の操作を定義できるようにします。</span><span class="sxs-lookup"><span data-stu-id="e79c4-104">However, the API is also extensible allowing you to define your own operations.</span></span> <span data-ttu-id="e79c4-105">API を拡張する 2 つの方法があります: を使用して、`Sql()`メソッド、またはカスタムを定義して`MigrationOperation`オブジェクト。</span><span class="sxs-lookup"><span data-stu-id="e79c4-105">There are two ways to extend the API: Using the `Sql()` method, or by defining custom `MigrationOperation` objects.</span></span>

<span data-ttu-id="e79c4-106">理解するには、それぞれのアプローチを使用してデータベース ユーザーを作成する操作の実装を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="e79c4-106">To illustrate, let's look at implementing an operation that creates a database user using each approach.</span></span> <span data-ttu-id="e79c4-107">この移行で、次のコードを記述できるようにします。</span><span class="sxs-lookup"><span data-stu-id="e79c4-107">In our migrations, we want to enable writing the following code:</span></span>

``` csharp
migrationBuilder.CreateUser("SQLUser1", "Password");
```

<a name="using-migrationbuildersql"></a><span data-ttu-id="e79c4-108">MigrationBuilder.Sql() を使用します。</span><span class="sxs-lookup"><span data-stu-id="e79c4-108">Using MigrationBuilder.Sql()</span></span>
----------------------------
<span data-ttu-id="e79c4-109">呼び出す拡張メソッドを定義するカスタム操作を実装する最も簡単な方法は、`MigrationBuilder.Sql()`です。</span><span class="sxs-lookup"><span data-stu-id="e79c4-109">The easiest way to implement a custom operation is to define an extension method that calls `MigrationBuilder.Sql()`.</span></span>
<span data-ttu-id="e79c4-110">適切な Transact SQL を生成する例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="e79c4-110">Here is an example that generates the appropriate Transact-SQL.</span></span>

``` csharp
static MigrationBuilder CreateUser(
    this MigrationBuilder migrationBuilder,
    string name,
    string password)
    => migrationBuilder.Sql($"CREATE USER {name} WITH PASSWORD '{password}';");
```

<span data-ttu-id="e79c4-111">複数のデータベース プロバイダーをサポートするために、移行する場合を使えば、`MigrationBuilder.ActiveProvider`プロパティです。</span><span class="sxs-lookup"><span data-stu-id="e79c4-111">If your migrations need to support multiple database providers, you can use the `MigrationBuilder.ActiveProvider` property.</span></span> <span data-ttu-id="e79c4-112">Microsoft SQL Server と PostreSQL の両方をサポートする例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="e79c4-112">Here's an example supporting both Microsoft SQL Server and PostreSQL.</span></span>

``` csharp
static MigrationBuilder CreateUser(
    this MigrationBuilder migrationBuilder,
    string name,
    string password)
{
    switch (migrationBuilder.ActiveProvider)
    {
        case "Npgsql.EntityFrameworkCore.PostgreSQL":
            return migrationBuilder
                .Sql($"CREATE USER {name} WITH PASSWORD '{password}';");

        case "Microsoft.EntityFrameworkCore.SqlServer":
            return migrationBuilder
                .Sql($"CREATE USER {name} WITH PASSWORD = '{password}';");
    }
}
```

<span data-ttu-id="e79c4-113">このアプローチだけはすべてのプロバイダーがわかっている場合、カスタムの操作が適用されます。</span><span class="sxs-lookup"><span data-stu-id="e79c4-113">This approach only works if you know every provider where your custom operation will be applied.</span></span>

<a name="using-a-migrationoperation"></a><span data-ttu-id="e79c4-114">使用して、MigrationOperation</span><span class="sxs-lookup"><span data-stu-id="e79c4-114">Using a MigrationOperation</span></span>
---------------------------
<span data-ttu-id="e79c4-115">SQL からカスタムの操作を切り離すことを定義できます。 自分`MigrationOperation`それを表します。</span><span class="sxs-lookup"><span data-stu-id="e79c4-115">To decouple the custom operation from the SQL, you can define your own `MigrationOperation` to represent it.</span></span> <span data-ttu-id="e79c4-116">操作は、ことを確認し、適切な SQL を生成するために、プロバイダーに渡されます。</span><span class="sxs-lookup"><span data-stu-id="e79c4-116">The operation is then passed to the provider so it can determine the appropriate SQL to generate.</span></span>

``` csharp
class CreateUserOperation : MigrationOperation
{
    public string Name { get; set; }
    public string Password { get; set; }
}
```

<span data-ttu-id="e79c4-117">この方法は、だけ、拡張メソッドはこれらの操作を 1 つ追加する必要があります。`MigrationBuilder.Operations`です。</span><span class="sxs-lookup"><span data-stu-id="e79c4-117">With this approach, the extension method just needs to add one of these operations to `MigrationBuilder.Operations`.</span></span>

``` csharp
static MigrationBuilder CreateUser(
    this MigrationBuilder migrationBuilder,
    string name,
    string password)
{
    migrationBuilder.Operations.Add(
        new CreateUserOperation
        {
            Name = name,
            Password = password
        });

    return migrationBuilder;
}
```

<span data-ttu-id="e79c4-118">この方法には、SQL では、この操作を生成する方法を理解するには、各プロバイダーが必要があります、`IMigrationsSqlGenerator`サービス。</span><span class="sxs-lookup"><span data-stu-id="e79c4-118">This approach requires each provider to know how to generate SQL for this operation in their `IMigrationsSqlGenerator` service.</span></span> <span data-ttu-id="e79c4-119">新しい操作を処理する SQL Server のジェネレーターをオーバーライドする例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="e79c4-119">Here is an example overriding the SQL Server's generator to handle the new operation.</span></span>

``` csharp
class MyMigrationsSqlGenerator : SqlServerMigrationsSqlGenerator
{
    public MyMigrationsSqlGenerator(
        MigrationsSqlGeneratorDependencies dependencies,
        IMigrationsAnnotationProvider migrationsAnnotations)
        : base(dependencies, migrationsAnnotations)
    {
    }

    protected override void Generate(
        MigrationOperation operation,
        IModel model,
        MigrationCommandListBuilder builder)
    {
        if (operation is CreateUserOperation createUserOperation)
        {
            Generate(createUserOperation, builder);
        }
        else
        {
            base.Generate(operation, model, builder);
        }
    }

    private void Generate(
        CreateUserOperation operation,
        MigrationCommandListBuilder builder)
    {
        var sqlHelper = Dependencies.SqlGenerationHelper;
        var stringMapping = Dependencies.TypeMapper.GetMapping(typeof(string));

        builder
            .Append("CREATE USER ")
            .Append(sqlHelper.DelimitIdentifier(name))
            .Append(" WITH PASSWORD = ")
            .Append(stringMapping.GenerateSqlLiteral(password))
            .AppendLine(sqlHelper.StatementTerminator)
            .EndCommand();
    }
}
```

<span data-ttu-id="e79c4-120">1 つずつ更新で、既定の移行 sql ジェネレーターのサービスを置き換えます。</span><span class="sxs-lookup"><span data-stu-id="e79c4-120">Replace the default migrations sql generator service with the updated one.</span></span>

``` csharp
protected override void OnConfiguring(DbContextOptionsBuilder options)
    => options
        .UseSqlServer(connectionString)
        .ReplaceService<IMigrationsSqlGenerator, MyMigrationsSqlGenerator>();
```