<properties 
   pageTitle="Log Analytics のアラートでの Webhook のサンプル"
   description="Log Analytics のアラートに対して実行できるアクションの 1 つが *Webhook* であり、1 つの HTTP 要求を使用して外部プロセスを呼び出すことができます。この記事では、Slack を使用して、Log Analytics のアラートで Webhook アクションを作成する例を見ていきます。"
   services="log-analytics"
   documentationCenter=""
   authors="bwren"
   manager="jwhit"
   editor="tysonn" />
<tags 
   ms.service="log-analytics"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="05/12/2016"
   ms.author="bwren" />

# Log Analytics のアラートでの Webhook

[Log Analytics のアラート](log-analytics-alerts.md)に対して実行できるアクションの 1 つが *Webhook* であり、1 つの HTTP 要求を使用して外部プロセスを呼び出すことができます。アラートと Webhook の詳細については、「[Log Analytics のアラート](log-analytics-alerts.md)」を参照してください。

この記事では、メッセージング サービスである Slack を使用して、Log Analytics のアラートで Webhook アクションを作成する例を見ていきます。

>[AZURE.NOTE] このサンプルを完了するには、Slack のアカウントが必要です。[slack.com](http://slack.com) で、無料のアカウントにサインアップすることができます。

## 手順 1. Slack で Webhook を有効にする
2.	[slack.com](http://slack.com) で Slack にサインインします。
3.	左側のウィンドウの **[Channels]** (チャネル) セクションで、チャネルを選択します。これは、メッセージが送信されるチャネルとなります。"**general**"、"**random**" などの既定のチャネルを選択できます。実際のシナリオでは、多くの場合、"**criticalservicealerts**" などの特別なチャネルを作成します。<br>

	![Slack channels](media/log-analytics-alerts-webhooks/oms-webhooks01.png)

3. **[Add an app or custom integration]** (アプリまたはカスタム統合を追加する) をクリックして、App Directory を開きます。
3.	検索ボックスに「*webhooks*」と入力し、**[Incoming WebHooks]** (受信 Webhook) を選択します。<br>
	
	![Slack channels](media/log-analytics-alerts-webhooks/oms-webhooks02.png)

4.	チーム名の横にある **[Install]** (インストール) をクリックします。
5.	**[Add Configuration]** (構成の追加) をクリックします。
6.	この例で使用するチャネルを選択し、**[Add Incoming WebHooks integration]** (受信 Webhook 統合の追加) をクリックします。  
6. **Webhook URL** をコピーします。後でこれをアラートの構成に貼り付けます。<br>

	![Slack channels](media/log-analytics-alerts-webhooks/oms-webhooks05.png)

## 手順 2. Log Analytics でアラート ルールを作成する
1.	次の設定で[アラート ルールを作成](log-analytics-alerts.md)します。
    - クエリ: ```	Type=Event EventLevelName=error ```
    - このアラートのチェック間隔: 5 分
    - 結果の数: 10 より大きい
    - 時間枠: 60 分
    - **[Webhook]** には **[Yes]** (はい)、他のアクションには **[No]** (いいえ) を選択
7. **[Webhook URL]** フィールドに Slack の URL を貼り付けます。 
8. **[Include custom JSON payload]** (カスタム JSON ペイロードを含める) オプションを選択します。
9. Slack では、"*text*" という名前のパラメーターを持つ JSON 形式のペイロードが想定されます。これは、作成されたメッセージに表示されるテキストです。次の例に示すように、*#* シンボルを使用して 1 つ以上のアラート パラメーターを指定できます。

    ```
    {
    "text":"#alertrulename fired with #searchresultcount records which exceeds the over threshold of #thresholdvalue ."
    }	
    ```

	![example JSON payload](media/log-analytics-alerts-webhooks/oms-webhooks07.png)

9.	**[Save]** (保存) をクリックして、アラート ルールを保存します。
    
10. アラートが作成されるまで十分な時間だけ待ってから、Slack で次のようなメッセージが表示されるかどうかを確認します。
    
	![example webhook in Slack](media/log-analytics-alerts-webhooks/oms-webhooks08.png)


### Slack 用の高度な Webhook ペイロード

Slack では、受信メッセージを広範囲にカスタマイズできます。詳細については、Slack の Web サイトで「[Incoming Webhooks (受信 Webhook)](https://api.slack.com/incoming-webhooks)」を参照してください。書式設定を含むリッチなメッセージを作成する、より複雑なペイロードを次に示します。

	{
		"attachments": [
			{
				"title":"OMS Alerts Custom Payload",
				"fields": [
					{
						"title": "Alert Rule Name",
						"value": "#alertrulename"},
					{
						"title": "Link To SearchResults",
						"value": "<#linktosearchresults|OMS Search Results>"},
					{
						"title": "Search Interval",
						"value": "#searchinterval"},
					{
						"title": "Threshold Operator",
						"value": "#thresholdoperator"},	
					{
						"title": "Threshold Value",
						"value": "#thresholdvalue"}
				],
				"color": "#F35A00"
			}
		]
	}


この場合、Slack に次のようなメッセージが生成されます。

![example message in Slack](media/log-analytics-alerts-webhooks/oms-webhooks09.png)

## 概要

このアラート ルールを設定すると、条件が満たされるたびに Slack にメッセージが送信されます。

これは、アラートへの応答として作成できるアクションの一例にすぎません。別の外部サービスを呼び出す Webhook アクション、Azure Automation で Runbook を開始する Runbook アクション、自分または他の受信者に電子メールを送信する電子メール アクションなども作成できます。

## 次のステップ

- 他のアクションなど、[Log Analytics のアラート](log-analytics-alerts.md)の詳細を学習します。
- Webhook から呼び出すことができる [Azure Automation の Runbook を作成](../automation/automation-webhooks.md)します。

<!---HONumber=AcomDC_0518_2016-->