<properties
   pageTitle="SQL Server から Azure SQL Data Warehouse へのデータの読み込み (PolyBase) | Microsoft Azure"
   description="bcp を使用して SQL Server からフラット ファイルにデータをエクスポートし、AZCopy を使用してデータを Azure Blob Storage にインポートし、PolyBase を使用してデータを Azure SQL Data Warehouse に取り込みます。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="06/30/2016"
   ms.author="lodipalm;barbkess;sonyama"/>


# SQL Server から Azure SQL Data Warehouse へのデータの読み込み (AZCopy)

SQL Server から Azure Blob Storage にデータを読み込むには、bcp および AZCopy コマンド ライン ユーティリティを使用します。その後、データを Azure SQL Data Warehouse に読み込むには、PolyBase または Azure Data Factory を使用します。


## 前提条件

このチュートリアルを進めるには、次が必要です。

- SQL Data Warehouse データベース
- インストールされた bcp コマンド ライン ユーティリティ
- インストールされた SQLCMD コマンド ライン ユーティリティ

>[AZURE.NOTE] bcp および sqlcmd ユーティリティは [Microsoft ダウンロード センター][]からダウンロードできます。

## SQL Data Warehouse へのデータのインポート

このチュートリアルでは、Azure SQL Data Warehouse でテーブルを作成し、そのデータをテーブルにインポートします。

### 手順 1: Azure SQL Data Warehouse でテーブルを作成する

コマンド プロンプトで sqlcmd を使用して次のクエリを実行し、インスタンスにテーブルを作成します。

```sql
sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "
    CREATE TABLE DimDate2
    (
        DateId INT NOT NULL,
        CalendarQuarter TINYINT NOT NULL,
        FiscalQuarter TINYINT NOT NULL
    )
    WITH
    (
        CLUSTERED COLUMNSTORE INDEX,
        DISTRIBUTION = ROUND_ROBIN
    );
"
```

>[AZURE.NOTE] SQL Data Warehouse でのテーブルの作成と、WITH 句で使用できるオプションの詳細については、[テーブルの概要][]に関する記事または [CREATE TABLE 構文][]に関するページを参照してください。

### 手順 2: ソース データ ファイルを作成する

メモ帳を開き、データの以下の行を新しいテキスト ファイルにコピーして、このファイルをローカルの一時ディレクトリに保存します (C:\\Temp\\DimDate2.txt)。

```
20150301,1,3
20150501,2,4
20151001,4,2
20150201,1,3
20151201,4,2
20150801,3,1
20150601,2,4
20151101,4,2
20150401,2,4
20150701,3,1
20150901,3,1
20150101,1,3
```

> [AZURE.NOTE] bcp.exe は、UTF-8 のファイルのエンコーディングをサポートしていないことに注意してください。bcp.exe を使用する場合は、ASCII ファイルまたは UTF-16 エンコード ファイルを使用してください。

### 手順 3: データに接続し、インポートする
bcp を使用して、次のコマンドでデータに接続し、インポートできます。値は適宜置き換えて使用してください。

```sql
bcp DimDate2 in C:\Temp\DimDate2.txt -S <Server Name> -d <Database Name> -U <Username> -P <password> -q -c -t  ','
```

sqlcmd を使用して次のクエリを実行すると、データが読み込まれたかどうかを確認することができます。

```sql
sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "SELECT * FROM DimDate2 ORDER BY 1;"
```

これにより、次の結果が得られます。

DateId |CalendarQuarter |FiscalQuarter
----------- |--------------- |-------------
20150101 |1 |3
20150201 |1 |3
20150301 |1 |3
20150401 |2 |4
20150501 |2 |4
20150601 |2 |4
20150701 |3 |1
20150801 |3 |1
20150801 |3 |1
20151001 |4 |2
20151101 |4 |2
20151201 |4 |2

### 手順 4: 新しくロードしたデータの統計を作成する

Azure SQL Data Warehouse は、統計の自動作成または自動更新をまだサポートしていません。クエリから最高のパフォーマンスを取得するには、最初の読み込み後またはそれ以降のデータの変更後に、すべてのテーブルのすべての列で統計を作成することが重要です。統計の詳細については、開発トピック グループの「[統計][]」トピックを参照してください。この例でロードしたテーブルの統計を作成する方法の簡単な例を次に示します

sqlcmd プロンプトから次の CREATE STATISTICS ステートメントを実行します。

```sql
sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "
    create statistics [DateId] on [DimDate2] ([DateId]);
    create statistics [CalendarQuarter] on [DimDate2] ([CalendarQuarter]);
    create statistics [FiscalQuarter] on [DimDate2] ([FiscalQuarter]);
"
```

## SQL Data Warehouse からのデータのエクスポート
このチュートリアルでは SQL Data Warehouse 内のテーブルからデータ ファイルを作成します。上記で作成したデータを DimDate2\_export.txt という名前の新しいデータ ファイルにエクスポートします。

### 手順 1: データをエクスポートする

bcp ユーティリティを使用して、次のコマンドでデータに接続し、エクスポートできます。値は適宜置き換えて使用してください。

```sql
bcp DimDate2 out C:\Temp\DimDate2_export.txt -S <Server Name> -d <Database Name> -U <Username> -P <password> -q -c -t ','
```
新しいファイルを開き、データが正しくエクスポートされたことを確認できます。ファイル内のデータは、次のテキストと一致する必要があります。

```
20150301,1,3
20150501,2,4
20151001,4,2
20150201,1,3
20151201,4,2
20150801,3,1
20150601,2,4
20151101,4,2
20150401,2,4
20150701,3,1
20150901,3,1
20150101,1,3
```

>[AZURE.NOTE] 分散システムの性質上、データの順序は SQL Data Warehouse データベース全体で異なる場合があります。別の方法として、bcp の **queryout** 関数を使用して、テーブル全体をエクスポートするのではなく、クエリによる抽出を記述することもできます。

## 次のステップ
読み込みの概要については、「[Load data into SQL Data Warehouse (SQL Data Warehouse へのデータの読み込み)][]」を参照してください。開発に関するその他のヒントについては、「[SQL Data Warehouse development overview (SQL Data Warehouse の開発の概要)][]」をご覧ください。

<!--Image references-->

<!--Article references-->

[Load data into SQL Data Warehouse (SQL Data Warehouse へのデータの読み込み)]: ./sql-data-warehouse-overview-load.md
[SQL Data Warehouse development overview (SQL Data Warehouse の開発の概要)]: ./sql-data-warehouse-overview-develop.md
[テーブルの概要]: ./sql-data-warehouse-tables-overview.md
[統計]: ./sql-data-warehouse-tables-statistics.md

<!--MSDN references-->
[bcp]: https://msdn.microsoft.com/library/ms162802.aspx
[CREATE TABLE 構文]: https://msdn.microsoft.com/library/mt203953.aspx

<!--Other Web references-->
[Microsoft ダウンロード センター]: https://www.microsoft.com/download/details.aspx?id=36433

<!---HONumber=AcomDC_0706_2016-->