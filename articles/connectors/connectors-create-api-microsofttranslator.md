<properties
    pageTitle="PowerApps Enterprise またはロジック アプリに Microsoft Translator を追加する | Microsoft Azure"
    description="Microsoft Translator コネクタと REST API パラメーターの概要"
    services=""
    suite=""
    documentationCenter="" 
    authors="MandiOhlinger"
    manager="erikre"
    editor=""
    tags="connectors"/>

<tags
   ms.service="multiple"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na" 
   ms.date="05/18/2016"
   ms.author="mandia"/>

# Microsoft Translator コネクタの使用
Microsoft Translator に接続して、テキストの翻訳、言語の検出などを行います。Microsoft Translator コネクタは、次のツールから使用できます。

- Logic Apps 
- PowerApps

> [AZURE.SELECTOR]
- [Logic Apps](../articles/connectors/connectors-create-api-microsofttranslator.md)
- [PowerApps Enterprise](../articles/power-apps/powerapps-create-api-microsofttranslator.md)

Microsoft Translator では、次の操作を実行できます。

- Microsoft Translator から取得したデータに基づいてビジネス フローを構築できます。 
- アクションを使用して、テキストの翻訳、言語の検出などを行うことができます。また、これらのアクションで応答を取得すると、他のアクションから出力を使用できます。たとえば、Dropbox で新しいファイルが作成されたときに、Microsoft Translator を使用してファイル内のテキストを別の言語に翻訳することができます。
- PowerApps Enterprise に Microsoft Translator コネクタを追加できます。追加すると、ユーザーはアプリ内でコネクタを使用できるようになります。 

PowerApps Enterprise にコネクタを追加する方法については、[PowerApps でのコネクタの登録](../power-apps/powerapps-register-from-available-apis.md)に関するページを参照してください。

Logic Apps に操作を追加する方法については、「[SaaS サービスを接続する新しいロジック アプリの作成](../app-service-logic/app-service-logic-create-a-logic-app.md)」をご覧ください。

## トリガーとアクション
Microsoft Translator には、次のアクションがあります。トリガーはありません。

トリガー | アクション
--- | ---
なし | <ul><li>言語を検出する</li><li>テキストを音声に変換する</li><li>テキストを翻訳する</li><li>言語を取得する</li><li>音声の言語を取得する</li></ul>

すべてのコネクタは、JSON および XML 形式のデータに対応します。


## Microsoft Translator への接続を作成する

>[AZURE.INCLUDE [Microsoft Translator への接続を作成する手順](../../includes/connectors-create-api-microsofttranslator.md)]


## Swagger REST API リファレンス
適用されるバージョン: 1.0。

### 言語を検出する    
指定されたテキストのソース言語を検出します。```GET: /Detect```

| 名前| データ型|必須|場所|既定値|説明|
| ---|---|---|---|---|---|
|query|string|○|query|なし |言語を特定するテキスト|

#### Response
|名前|説明|
|---|---|
|200|OK|
|default|操作に失敗しました。|


### テキストを音声に変換する    
指定されたテキストを wave 形式のオーディオ ストリームとして音声に変換します。```GET: /Speak```

| 名前| データ型|必須|場所|既定値|説明|
| ---|---|---|---|---|---|
|query|string|○|query|なし |変換するテキスト|
|言語|string|○|query|なし |音声を生成する言語コード (例: 'ja-JP')|

#### Response
|名前|説明|
|---|---|
|200|OK|
|default|操作に失敗しました。|


### テキストを翻訳する    
Microsoft Translator を使用して、指定された言語にテキストを翻訳します。```GET: /Translate```

| 名前| データ型|必須|場所|既定値|説明|
| ---|---|---|---|---|---|
|query|string|○|query|なし |翻訳するテキスト|
|languageTo|string|○|query| なし|ターゲット言語コード (例: "fr")|
|languageFrom|string|×|query|なし |ソース言語。指定しない場合、Microsoft Translator で自動検出が試行されます (例: en)。|
|カテゴリ|string|×|query|全般 |翻訳のカテゴリ (既定値: 'general')|

#### Response
|名前|説明|
|---|---|
|200|OK|
|default|操作に失敗しました。|


### 言語を取得する    
Microsoft Translator がサポートしているすべての言語を取得します。```GET: /TranslatableLanguages```

この呼び出しには、パラメーターはありません。

#### Response
|名前|説明|
|---|---|
|200|OK|
|default|操作に失敗しました。|


### 音声の言語を取得する    
音声合成に使用できる言語を取得します。```GET: /SpeakLanguages```

この呼び出しには、パラメーターはありません。

#### Response
|名前|説明|
|---|---|
|200|OK|
|default|操作に失敗しました。|

## オブジェクト定義

#### Language: Microsoft Translator で翻訳できる言語の言語モデル

|プロパティ名 | データ型 | 必須|
|---|---|---|
|コード|string|×|
|名前|string|×|


## 次のステップ

[ロジック アプリを作成](../app-service-logic/app-service-logic-create-a-logic-app.md)します。

[API リスト](apis-list.md)に戻ります。


<!--References-->
[5]: https://datamarket.azure.com/developer/applications/
[6]: ./media/connectors-create-api-microsofttranslator/register-your-application.png

<!---HONumber=AcomDC_0525_2016-->