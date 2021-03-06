<properties
	pageTitle="Azure Functions における Event Hub のバインド | Microsoft Azure"
	description="Azure Functions で Azure Event Hub のバインドを使用する方法について説明します。"
	services="functions"
	documentationCenter="na"
	authors="wesmc7777"
	manager="erikre"
	editor=""
	tags=""
	keywords="Azure Functions, 関数, イベント処理, 動的コンピューティング, サーバーなしのアーキテクチャ"/>

<tags
	ms.service="functions"
	ms.devlang="multiple"
	ms.topic="reference"
	ms.tgt_pltfrm="multiple"
	ms.workload="na"
	ms.date="05/27/2016"
	ms.author="wesmc"/>

# Azure Functions における Event Hub のバインド

この記事では、Azure Functions で [Azure Event Hub](../event-hubs/event-hubs-overview.md) のバインドを構成したりコーディングしたりする方法について説明します。Azure Functions は、Azure Event Hubs のバインドのトリガーおよび出力をサポートしています。

[AZURE.INCLUDE [intro](../../includes/functions-bindings-intro.md)]


## Azure Event Hub のトリガーのバインド

Azure Event Hub のトリガーは、イベント ハブのイベント ストリームに送信されたイベントの応答に使用できます。トリガーのバインドをセットアップするには、イベント ハブの読み取りアクセスが必要です。

#### Event Hub トリガーのバインドのための function.json

Azure Event Hub トリガーの *function.json* ファイルでは、次のプロパティを指定します。

- `type` : *eventHubTrigger* に設定する必要があります。
- `name` : イベント ハブ メッセージの関数コードで使用される変数名。
- `direction`: *in* に設定する必要があります。
- `path` : イベント ハブの名前。
- `connection` : アプリ設定の名前。イベント ハブが配置される名前空間への接続文字列が含まれます。イベント ハブ自体ではなく、名前空間の **[接続情報]** ボタンをクリックして、この接続文字列をコピーします。この接続文字列には、トリガーをアクティブにするために少なくとも読み取りアクセス許可が必要です。

		{
		  "bindings": [
		    {
		      "type": "eventHubTrigger",
		      "name": "myEventHubMessage",
		      "direction": "in",
		      "path": "MyEventHub",
		      "connection": "myEventHubReadConnectionString"
		    }
		  ],
		  "disabled": false
		}

#### Azure Event Hub のトリガー C# の例
 
上記の function.json の例を使用して、次の C# 関数コードでイベント メッセージの本文を記録します。
 
	using System;
	
	public static void Run(string myEventHubMessage, TraceWriter log)
	{
	    log.Info($"C# Event Hub trigger function processed a message: {myEventHubMessage}");
	}

#### Azure Event Hub のトリガー Node.js の例
 
上記の function.json の例を使用して、次の Node.js 関数コードでイベント メッセージの本文を記録します。
 
	module.exports = function (context, myEventHubMessage) {
	    context.log('Node.js eventhub trigger function processed work item', myEventHubMessage);	
	    context.done();
	};


## Azure Event Hub の出力バインド

Azure Event Hub の出力バインドを使用して、イベント ハブのイベント ストリームにイベントを書き込みます。イベントを書き込むには、イベント ハブへの送信アクセス許可が必要です。

#### Event Hub 出力バインドの function.json

Azure Event Hub 出力バインドの *function.json* ファイルでは、次のプロパティを指定します。

- `type` : *eventHub* に設定する必要があります。
- `name` : イベント ハブ メッセージの関数コードで使用される変数名。
- `path` : イベント ハブの名前。
- `connection` : アプリ設定の名前。イベント ハブが配置される名前空間への接続文字列が含まれます。イベント ハブ自体ではなく、名前空間の **[接続情報]** ボタンをクリックして、この接続文字列をコピーします。この接続文字列には、Event Hub ストリームにメッセージを送信するための送信アクセス許可が必要です。
- `direction` : *out* に設定する必要があります。

	    {
	      "type": "eventHub",
	      "name": "outputEventHubMessage",
	      "path": "myeventhub",
	      "connection": "MyEventHubSend",
	      "direction": "out"
	    }


#### 出力バインドのための Azure Event Hub C# のコード例
 
次の C# 関数コードの例では、Event Hub のイベント ストリームにイベントを書き込みます。この例では、上記の Event Hub 出力バインドを C# タイマー トリガーに適用しています。
 
	using System;
	
	public static void Run(TimerInfo myTimer, out string outputEventHubMessage, TraceWriter log)
	{
	    String msg = $"TimerTriggerCSharp1 executed at: {DateTime.Now}";
	
	    log.Verbose(msg);   
	    
	    outputEventHubMessage = msg;
	}

#### 出力バインドのための Azure Event Hub Node.js のコード例
 
次の Node.js 関数コードの例では、Event Hub のイベント ストリームにイベントを書き込みます。この例では、上記の Event Hub 出力バインドを Node.js タイマー トリガーに適用しています。
 
	module.exports = function (context, myTimer) {
	    var timeStamp = new Date().toISOString();
	    
	    if(myTimer.isPastDue)
	    {
	        context.log('TimerTriggerNodeJS1 is running late!');
	    }

	    context.log('TimerTriggerNodeJS1 function ran!', timeStamp);   
	    
	    context.bindings.outputEventHubMessage = "TimerTriggerNodeJS1 ran at : " + timeStamp;
	
	    context.done();
	};

## 次のステップ

[AZURE.INCLUDE [次のステップ](../../includes/functions-bindings-next-steps.md)]

<!---HONumber=AcomDC_0713_2016-->