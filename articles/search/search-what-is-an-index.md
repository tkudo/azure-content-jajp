<properties
	pageTitle="Azure Search インデックスの作成 | Microsoft Azure | ホステッド クラウド検索サービス"
	description="Azure Search のインデックスとその使用方法について説明します。"
	services="search"
	documentationCenter=""
authors="ashmaka"
/>

<tags
	ms.service="search"
	ms.devlang="na"
	ms.workload="search"
	ms.topic="get-started-article"
	ms.tgt_pltfrm="na"
	ms.date="05/31/2016"
	ms.author="ashmaka"/>

# Azure Search インデックスの作成
> [AZURE.SELECTOR]
- [概要](search-what-is-an-index.md)
- [ポータル](search-create-index-portal.md)
- [.NET](search-create-index-dotnet.md)
- [REST ()](search-create-index-rest-api.md)

## インデックスとは

*インデックス* とは、Azure Search サービスで使用される*ドキュメント*などの構成要素の永続的なストアです。ドキュメントは、インデックス内の 1 単位の検索可能なデータです。たとえば、e コマースの小売業者であれば販売品目ごとにドキュメントがあり、報道機関であれば記事ごとにドキュメントがあります。これらの概念をなじみのあるデータベースの同等のものに対応させるなら、*インデックス*は概念的には*テーブル*に似ており、*ドキュメント*はテーブルにおける*行*とほぼ同じです。

ドキュメントを追加/アップロードして検索クエリを Azure Search に送信する場合、検索サービス内の特定のインデックスへの要求を送信していることになります。

## Azure Search インデックスのフィールドの種類と属性

スキーマを定義する際に、インデックスの各フィールドの名前、型、属性を指定する必要があります。フィールドの型によって、そのフィールドに格納されているデータが分類されます。属性は個々のフィールドに設定されてフィールドの使用方法を指定します。次の表に、指定できる型と属性をまとめます。


### フィールドの型
|種類|説明|
|------------|-----------|
|*Edm.String*|フルテキスト検索 (単語区切り、ステミングなど) のために必要に応じてトークン化することのできるテキスト。|
|*Collection(Edm.String)*|フルテキスト検索のために必要に応じてトークン化することのできる一連の文字列。コレクション内の項目の数に理論上の上限はありませんが、ペイロードのサイズに対する 16 MB の上限がコレクションに適用されます。|
|*Edm.Boolean*|true または false の値が含まれます。|
|*Edm.Int32*|32 ビット整数値です。|
|*Edm.Int64*|64 ビット整数値です。|
|*Edm.Double*|倍精度数値データです。|
|*Edm.DateTimeOffset*|OData V4 形式で表された日時の値です (例: `yyyy-MM-ddTHH:mm:ss.fffZ` または `yyyy-MM-ddTHH:mm:ss.fff[+/-]HH:mm`)。|
|*Edm.GeographyPoint*|地球上の地理的な場所を表すポイントです。|

Azure Search の[サポートされるデータ型について詳しくは、MSDN をご覧ください](https://msdn.microsoft.com/library/azure/dn798938.aspx)。



### フィールド属性
|属性|説明|
|------------|-----------|
|*キー*|ドキュメント検索に使用される各ドキュメントの一意の ID を提供する文字列です。各インデックスに、1 つのキーが必要です。1 つのフィールドだけをキーにすることができ、その型を Edm.String に設定する必要があります。|
|*Retrievable*|検索結果でフィールドを返すことができるかどうかを設定します。|
|*Filterable*|フィルター クエリでフィールドを使用できるようにします。|
|*Sortable*|このフィールドを使ってクエリで検索結果を並べ替えられるようにします。|
|*Facetable*|ユーザー自律フィルター処理の[ファセット ナビゲーション](search-faceted-navigation.md)構造でフィールドを使用できるようにします。通常は、複数のドキュメント (たとえば、1 つのブランドやサービス カテゴリに属する複数のドキュメント) をまとめてグループ化するために使用できる、反復する値があるフィールドが、ファセットとして最適です。|
|*Searchable*|フィールドをフルテキスト検索可能としてマークします。|

Azure Search の[インデックス属性について詳しくは、MSDN をご覧ください](https://msdn.microsoft.com/library/azure/dn798941.aspx)。



## インデックス スキーマの定義に関するガイダンス

インデックスを設計する際は、計画フェーズに時間をかけ、それぞれの決定項目を十分に検討してください。各フィールドには[適切な属性](https://msdn.microsoft.com/library/azure/dn798941.aspx)を割り当てる必要があるため、インデックスを設計する際は、検索のユーザー エクスペリエンスとビジネス ニーズに留意することが重要です。デプロイ後にインデックスを変更するには、再構築とデータの再読み込みが必要になります。


データ ストレージの要件が随時変更される環境でも、パーティションを追加または削除することで容量を増減できます。詳細については、「[Microsoft Azure で Search サービスを管理する](search-manage.md)」または「[サービスの制限](search-limits-quotas-capacity.md)」を参照してください。

<!---HONumber=AcomDC_0601_2016-->