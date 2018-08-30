---
title: 所有エンティティの型 - EF Core
author: julielerman
ms.date: 2/26/2018
ms.assetid: 2B0BADCE-E23E-4B28-B8EE-537883E16DF3
uid: core/modeling/owned-entities
ms.openlocfilehash: afbc853feab56d31a8ceba0da05fe6dc508b65bb
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/27/2018
ms.locfileid: "42994070"
---
# <a name="owned-entity-types"></a>所有エンティティ型

>[!NOTE]
> この機能は、EF Core 2.0 の新機能です。

EF Core では、その他のエンティティ型のナビゲーション プロパティにのみ表示できるモデルのエンティティ型にできます。 これらと呼ばれます_エンティティ型を所有している_します。 所有エンティティ型を含むエンティティはその_所有者_します。

## <a name="explicit-configuration"></a>明示的な構成

エンティティ型は含まれません EF Core によって、モデルの規則により所有されています。 使用することができます、`OwnsOne`メソッド`OnModelCreating`を持つ型の注釈を設定または`OwnedAttribute`(EF Core 2.1 の新機能) 所有型として型を構成します。

この例では、StreetAddress は identity プロパティは含まずの一種です。 特定の注文の配送先住所を指定するために、Order 型のプロパティとして使用されます。 `OnModelCreating`を使用して、 `OwnsOne` ShippingAddress プロパティが、Order 型の所有エンティティであることを指定するメソッド。

``` csharp
public class StreetAddress
{
    public string Street { get; set; }
    public string City { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public StreetAddress ShippingAddress { get; set; }
}

// OnModelCreating
modelBuilder.Entity<Order>().OwnsOne(p => p.ShippingAddress);
```

ShippingAddress プロパティが Order type でプライベートの場合は、文字列形式を使用することができます、`OwnsOne`メソッド。

``` csharp
modelBuilder.Entity<Order>().OwnsOne(typeof(StreetAddress), "ShippingAddress");
```

この例では使用して、`OwnedAttribute`同じ目標を達成します。

``` csharp
[Owned]
public class StreetAddress
{
    public string Street { get; set; }
    public string City { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public StreetAddress ShippingAddress { get; set; }
}
```

## <a name="implicit-keys"></a>暗黙的なキー

EF Core 2.0 および 2.1 では、参照ナビゲーション プロパティのみが所有型を指定できます。 所有型のコレクションがサポートされていません。 これらの参照は所有型が常に一対一のリレーションシップを持つ所有者と、そのため、独自のキー値が不要です。 前の例ではキー プロパティを定義する、StreetAddress 型は必要ありません。  

EF Core がこれらのオブジェクトを追跡する方法を理解するためにとして主キーが作成されたことを考えるとわかり、[プロパティをシャドウ](xref:core/modeling/shadow-properties)所有型のです。 所有型のインスタンスのキーの値は、所有者のインスタンスのキーの値と同じになります。      

## <a name="mapping-owned-types-with-table-splitting"></a>マッピングは、テーブル分割を持つ型を所有しています。

リレーショナル データベースを使用して、慣例所有型にマップされます、所有者と同じテーブル。 2 つのテーブルを分割する必要があります。 一部の列は、所有者のデータの格納に使用すると、所有エンティティのデータの格納に使用されるいくつかの列。 これは、テーブル分割と呼ばれる一般的な機能です。

> [!TIP]
> テーブル分割と保存されている種類は、所有する EF6 で使用する方法の複合型にも同様に非常に使用します。

規則により、EF Core は、次のパターンの所有エンティティ型のプロパティのデータベース列を名前は_EntityProperty_OwnedEntityProperty_します。 そのため、StreetAddress プロパティは、ShippingAddress_Street と ShippingAddress_City 名前と Orders テーブルに表示されます。

追加することができます、`HasColumnName`それらの列の名前を変更するメソッド。 StreetAddress のパブリック プロパティがある場合は、マッピングになります

``` csharp
modelBuilder.Entity<Order>().OwnsOne(
    o => o.ShippingAddress,
    sa =>
        {
            sa.Property(p=>p.Street).HasColumnName("ShipsToStreet");
            sa.Property(p=>p.City).HasColumnName("ShipsToCity");
        });
```

## <a name="sharing-the-same-net-type-among-multiple-owned-types"></a>複数の所有型の間で同じ .NET 型の共有

所有エンティティ型できます別の所有エンティティ型と同じ .NET 型したがって .NET 型できない可能性があります所有型を識別するために十分な。

その場合、所有者から所有エンティティを指すプロパティになります、_ナビゲーションを定義する_の所有エンティティ型。 EF Core の観点からは、定義となるナビゲーションは、.NET 型と型の id の一部です。   

たとえば、次のクラスで ShippingAddress および BillingAddress は同じ .NET 型、StreetAddress いずれです。

``` csharp
public class Order
{
    public int Id { get; set; }
    public StreetAddress ShippingAddress { get; set; }
    public StreetAddress BillingAddress { get; set; }
}
```

EF Core がこれらのオブジェクトの追跡対象のインスタンスを区別する方法を理解するためには、定義となるナビゲーションが所有者のキーの値と共にインスタンスのキーの一部と所有型の .NET 型になったことを考えるとわかり場合があります。

## <a name="nested-owned-types"></a>入れ子にされた所有型

この例では、OrderDetails は、BillingAddress と ShippingAddress、これらは両方の StreetAddress 型を所有します。 OrderDetails Order type が所有します。

``` csharp
public class Order
{
    public int Id { get; set; }
    public OrderDetails OrderDetails { get; set; }
    public OrderStatus Status { get; set; }
}

public class OrderDetails
{
    public StreetAddress BillingAddress { get; set; }
    public StreetAddress ShippingAddress { get; set; }
}

public enum OrderStatus
{
    Pending,
    Shipped
}

public class StreetAddress
{
    public string Street { get; set; }
    public string City { get; set; }
}
```

チェーンすることができます、`OwnsOne`メソッドでこのモデルを構成する fluent マッピング。

``` csharp
modelBuilder.Entity<Order>().OwnsOne(p => p.OrderDetails, od =>
    {
        od.OwnsOne(c => c.BillingAddress);
        od.OwnsOne(c => c.ShippingAddress);
    });
```

使用して同じことを実現することは`OwnedAttribute`OrderDetails と StreetAdress の両方でします。

入れ子になったの所有型だけでなく、所有型は正規のエンティティを参照できます。 次の例では、国は、通常の非所有エンティティは。

``` csharp
public class StreetAddress
{
    public string Street { get; set; }
    public string City { get; set; }
    public Country Country { get; set; }
}
```

この機能は、EF6 の複合型とは別の所有エンティティ型を設定します。

## <a name="storing-owned-types-in-separate-tables"></a>別のテーブルに所有型を格納します。

また、EF6 の複合型とは異なりは、所有型を所有者から別のテーブルに格納できます。 所有型を所有者と同じテーブルにマップする規則を上書きするために呼び出すだけでできます`ToTable`別のテーブルの名前を指定します。 次の例では、OrderDetails と、その 2 つのアドレスにマップされます、別のテーブルの順序。

``` csharp
modelBuilder.Entity<Order>().OwnsOne(p => p.OrderDetails, od =>
    {
        od.OwnsOne(c => c.BillingAddress);
        od.OwnsOne(c => c.ShippingAddress);
        od.ToTable("OrderDetails");
    });
```

## <a name="querying-owned-types"></a>所有型を問い合わせています

所有者に問い合わせるとき、所有されている型が既定で含まれます。 使用する必要はありません、`Include`所有型が別のテーブルに格納されている場合でも、メソッド。 前に説明したモデルに基づいて、次のクエリは Order、OrderDetails およびデータベースからすべての保留中の注文の 2 つの所有 StreetAddresses をプルします。

``` csharp
var orders = context.Orders.Where(o => o.Status == OrderStatus.Pending);
```  

## <a name="limitations"></a>制限事項

これらの制限の一部は所有されているどのエンティティ型の作業を基本的な機能がいくつか他のユーザー制限は今後のリリースを削除するいただく可能性があります。

### <a name="shortcomings-in-previous-versions"></a>以前のバージョンの欠点
- EF Core 2.0 では、ナビゲーションには、所有エンティティが別のテーブルに所有者の階層から明示的にマップされていない限り、派生エンティティ型のエンティティ型を宣言できませんを所有します。 EF Core 2.1 でこの制限が削除されました

### <a name="current-shortcomings"></a>現在の欠点
- 継承階層を含む所有エンティティ型はサポートされていません
- 所有エンティティ型がで、コレクション ナビゲーション プロパティ (ナビゲーションは現在サポートされている唯一の参照) によって指されることはできません。
- 明示的に、所有者から別のテーブルにマップされていないエンティティ型を null にすることはできませんを所有する可能性のあるナビゲーション
- (これは、所有エンティティ型を使用して実装することはできません。 値オブジェクトのよく知られているシナリオ) 複数の所有者で所有エンティティ型のインスタンスを共有することはできません。

### <a name="by-design-restrictions"></a>設計による制限
- 作成することはできません、 `DbSet<T>`
- 呼び出すことはできません`Entity<T>()`所有型の使用 `ModelBuilder`
