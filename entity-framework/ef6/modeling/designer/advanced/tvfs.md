---
title: テーブル値関数 (Tvf) - EF6
author: divega
ms.date: 2016-10-23
ms.assetid: f019c97b-87b0-4e93-98f4-2c539f77b2dc
ms.openlocfilehash: f4b8c1bd442bac67a06584bd23c040381374f303
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993248"
---
# <a name="table-valued-functions-tvfs"></a>テーブル値関数 (Tvf)
> [!NOTE]
> **EF5 以降のみ**-機能、Api、Entity Framework 5 で導入されたなどのこのページで説明します。 以前のバージョンを使用している場合、一部またはすべての情報は適用されません。

ビデオ、およびステップ バイ ステップ チュートリアルでは、テーブル値関数 (Tvf、Entity Framework デザイナーを使用して) マップする方法を示します。 LINQ クエリから TVF を呼び出す方法も示します。

Tvf は現在、データベースの最初のワークフローでのみサポートされます。

TVF のサポートは、Entity Framework バージョン 5 で導入されました。 テーブル値関数、列挙型、および .NET Framework 4.5 をターゲットする必要がありますの空間型などの新機能を使用するように注意してください。 Visual Studio 2012 では、既定では .NET 4.5 を対象とします。

Tvf が 1 つの主な違いを使用したストアド プロシージャとよく似ています。 TVF の結果はコンポーザブルです。 つまり、TVF の結果は、ストアド プロシージャの結果ができないときに、LINQ クエリで使用できます。

## <a name="watch-the-video"></a>ビデオを見る

**によって提示される**: Julia Kornich

[WMV](http://download.microsoft.com/download/6/0/A/60A6E474-5EF3-4E1E-B9EA-F51D2DDB446A/HDI-ITPro-MSDN-winvideo-tvf.wmv) | [MP4](http://download.microsoft.com/download/6/0/A/60A6E474-5EF3-4E1E-B9EA-F51D2DDB446A/HDI-ITPro-MSDN-mp4video-tvf.m4v) | [WMV (ZIP)](http://download.microsoft.com/download/6/0/A/60A6E474-5EF3-4E1E-B9EA-F51D2DDB446A/HDI-ITPro-MSDN-winvideo-tvf.zip)

## <a name="pre-requisites"></a>前提条件

このチュートリアルを完了する必要があります。

- インストール、 [School データベース](~/ef6/resources/school-database.md)します。

- Visual Studio の最新バージョンします。

## <a name="set-up-the-project"></a>プロジェクトを設定します。

1.  Visual Studio を開く
2.  **ファイル**メニューで、**新規**、 をクリックし、**プロジェクト**
3.  左側のウィンドウで次のようにクリックします**Visual C\#** を選び、**コンソール**テンプレート。
4.  入力**TVF**をクリックして、プロジェクトの名前として**OK**

## <a name="add-a-tvf-to-the-database"></a>TVF をデータベースに追加します。

-   選択**ビュー -&gt; SQL Server オブジェクト エクスプ ローラー**
-   LocalDB はサーバーの一覧にない場合: を右クリックし**SQL Server**選択**SQL Server の追加**既定値を使用して、 **Windows 認証**LocalDB サーバーに接続するには
-   LocalDB のノードを展開します。
-   [データベース] ノードでは、School データベース ノードを右クリックして**新しいクエリ.**
-   T-SQL エディターでは、次の TVF 定義を貼り付け

``` SQL
CREATE FUNCTION [dbo].[GetStudentGradesForCourse]

(@CourseID INT)

RETURNS TABLE

RETURN
    SELECT [EnrollmentID],
           [CourseID],
           [StudentID],
           [Grade]
    FROM   [dbo].[StudentGrade]
    WHERE  CourseID = @CourseID
```

-   T-SQL エディターでマウスの右ボタンをクリックし、選択**Execute**
-   GetStudentGradesForCourse 関数は、School データベースに追加されます。

 

## <a name="create-a-model"></a>モデルを作成します。

1.  ソリューション エクスプ ローラーでプロジェクト名を右クリックし、[**追加**、] をクリックし、**新しい項目**
2.  選択**データ**選択し、左側のメニューから**ADO.NET Entity Data Model**で、**テンプレート**ウィンドウ
3.  入力**TVFModel.edmx**のファイル名、およびクリック**追加**
4.  モデルのコンテンツの選択 ダイアログ ボックスで、次のように選択します**データベースから生成**、 をクリックし、 **次へ。**
5.  クリックして**新しい接続**」と入力 **(localdb)\\mssqllocaldb**サーバー名テキスト ボックスに、Enter**学校**データベースの名前をクリックします **[ok]**
6.  選択 で、データベース オブジェクト ダイアログ ボックスで、**テーブル**ノードを選択、 **Person**、 **StudentGrade**、および**コース**テーブル
7.  選択、 **GetStudentGradesForCourse**関数の下にある、**ストアド プロシージャおよび関数**注、Visual Studio 2012 では、その開始ノード、エンティティ デザイナーを使用するバッチのインポート をストアド プロシージャおよび関数
8.  クリックして**完了**
9.  モデルを編集するため、デザイン サーフェイスを提供するエンティティ デザイナーが表示されます。 選択したすべてのオブジェクト、**データベース オブジェクトの選択** ダイアログ ボックスは、モデルに追加されます。
10. 既定では、各インポートされたストアド プロシージャまたは関数の結果の形式は、エンティティ モデル内の新しい複合型に自動的になります。 GetStudentGradesForCourse 関数の結果を StudentGrade エンティティにマップします選択し、画面のデザインを右クリック**モデル ブラウザー**モデル ブラウザーで、選択**関数インポート**。、し、ダブルクリック、 **GetStudentGradesForCourse**関数で、関数インポートの編集] ダイアログ ボックス、[**エンティティ**選択**StudentGrade**

## <a name="persist-and-retrieve-data"></a>永続化およびデータの取得

Main メソッドが定義されているファイルを開きます。 Main 関数には、次のコードを追加します。

次のコードでは、テーブル値関数を使用するクエリを作成する方法を示します。 クエリでは、関連のコースのタイトルと 3.5 以上のレベルが関連の受講者を含む匿名型に結果を射影します。

``` csharp
using (var context = new SchoolEntities())
{
    var CourseID = 4022;
    var Grade = 3.5M;

    // Return all the best students in the Microeconomics class.
    var students = from s in context.GetStudentGradesForCourse(CourseID)
                            where s.Grade >= Grade
                            select new
                            {
                                s.Person,
                                s.Course.Title
                            };

    foreach (var result in students)
    {
        Console.WriteLine(
            "Couse: {0}, Student: {1} {2}",
            result.Title,  
            result.Person.FirstName,  
            result.Person.LastName);
    }
}
```

アプリケーションをコンパイルして実行します。 このプログラムの出力は、次のようになります。

```
Couse: Microeconomics, Student: Arturo Anand
Couse: Microeconomics, Student: Carson Bryant
```

## <a name="summary"></a>まとめ

このチュートリアルでは、テーブル値関数 (Tvf、Entity Framework デザイナーを使用して) マップする方法を説明しました。 LINQ クエリから TVF を呼び出す方法も示されています。
