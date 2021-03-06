<properties
   pageTitle="Azure Active Directory 監査レポートのイベント | Microsoft Azure"
   description="Azure Active Directory から表示およびダウンロードできる監査対象イベントについて説明します。"
   services="active-directory"
   documentationCenter=""
   authors="dhanyahk"
   manager="stevenpo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="07/25/2016"
   ms.author="dhanyahk"/>

# Azure Active Directory 監査レポートのイベント

*このドキュメントは、[Azure Active Directory レポート ガイド](active-directory-reporting-guide.md)の一部です*。

Azure Active Directory 監査レポートを利用すると、Azure Active Directory で発生した特権アクションを識別できます。特権アクションとしては、昇格の変更 (例: ロールの作成、パスワードのリセット)、ポリシー構成の変更 (例: パスワード ポリシー)、ディレクトリ構成の変更 (例: ドメインのフェデレーション設定の変更) などがあります。レポートでは、イベント名、アクションを実行したアクター、変更によって影響を受けた対象リソース、日時 (UTC) に関する監査レコードが提供されます。「[アクセス レポートと状況レポートの表示](active-directory-view-access-usage-reports.md)」で説明したように、ユーザーは、[Microsoft Azure 管理ポータル](https://manage.windowsazure.com/)を使用して Azure Active Directory に対する監査イベントのリストを取得できます。アクセスおよび使用状況レポートを表示します。


## 監査レポートのイベントの一覧
<!--- audit event descriptions should be in the past tense --->

イベント | イベントの説明
------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**ユーザー イベント** |
ユーザーの追加 | ディレクトリにユーザーが追加されました。
ユーザーの削除 | ディレクトリからユーザーが削除されました。
ライセンスのプロパティの設定 | ディレクトリでユーザーのライセンス プロパティが設定されました。
ユーザー パスワードのリセット | ディレクトリでユーザーのパスワードがリセットされました。
ユーザー パスワードの変更 | ディレクトリでユーザーのパスワードが変更されました。
ユーザー ライセンスの変更 | ディレクトリでユーザーに割り当てられたライセンスが変更されました。更新されたライセンスについては、このイベントの直前または直後の「ユーザーの更新」イベントを参照してください。
ユーザーの更新 | ディレクトリでユーザーが更新されました。更新可能な属性については、[以下を参照](#quotupdate-userquot-attributes)してください。
ユーザー パスワードの強制変更の設定 | ログイン時にパスワードの変更をユーザーに強制するプロパティが設定されました。
**グループ イベント** |
グループの追加 | ディレクトリ内にグループが作成されました。
グループの更新 | ディレクトリ内のグループが更新されました。
グループの削除 | ディレクトリからグループが削除されました。
グループへのメンバーの追加 | ディレクトリ内のグループにメンバーが追加されました。
グループからのメンバーの削除 | ディレクトリ内のグループからメンバーが削除されました。
**アプリケーション イベント** |
サービス プリンシパルの追加 | ディレクトリにサービス プリンシパルが追加されました。
サービス プリンシパルの削除 | ディレクトリからサービス プリンシパルが削除されました。
サービス プリンシパルの資格情報の追加 | サービス プリンシパルに資格情報が追加されました。
サービス プリンシパルの資格情報の削除 | サービス プリンシパルから資格情報が削除されました。
委任エントリの追加 | ディレクトリに [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permissiongrant-entity) が作成されました。
委任エントリの設定 | ディレクトリの [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permissiongrant-entity) が更新されました。
委任エントリの削除 | ディレクトリの [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permissiongrant-entity) が削除されました。
**ロール イベント** |
ロールへのロール メンバーの追加 | ディレクトリのロールにユーザーが追加されました。
ロールからのロール メンバーの削除 | ディレクトリのロールからユーザーが削除されました。
会社の連絡先情報の設定 | 会社レベルで優先連絡先が設定されました。これには、マーケティングに対する電子メール アドレスおよび Microsoft オンライン サービスに関する技術的通知が含まれます。
**B2B イベント** |
バッチ招待がアップロードされました。 | 管理者が、パートナー ユーザーに送信する複数の招待を含むファイルをアップロードしました。
バッチ招待が処理されました。 | パートナー ユーザーに対する招待を含むファイルが処理されました。
外部ユーザーを招待します。 | 外部ユーザーがディレクトリに招待されました。
外部ユーザーの招待を利用します。 | 外部ユーザーは、ディレクトリへの招待を利用しました。
外部ユーザーをグループに追加します。 | 外部ユーザーには、ディレクトリ内のグループのメンバーシップが割り当てられました。
外部ユーザーをアプリケーションに割り当てます。 | 外部ユーザーには、アプリケーションへの直接アクセスが割り当てられました。
バイラル テナントの作成。 | 招待の利用によって、Azure AD に新しいテナントが作成されました。
バイラル ユーザーの作成。 | 招待の利用によって、Azure AD の既存のテナントにユーザーが作成されました。
**ディレクトリ イベント** |
会社へのパートナーの追加 | ディレクトリにパートナーが追加されました。
会社からのパートナーの削除 | ディレクトリからパートナーが削除されました。
会社へのドメインの追加 | ディレクトリにドメインが追加されました。
会社からのドメインの削除 | ディレクトリからドメインが削除されました。
ドメインの更新 | ディレクトリでドメインが更新されました。
ドメインの認証の設定 | 会社の既定のドメイン設定が変更されました。
ドメインのフェデレーションの設定 | ドメインのフェデレーション設定が更新されました。
ドメインの検証 | ディレクトリのドメインが検証されました。
電子メール検証済みドメインの検証 | 電子メールの検証を使用してディレクトリのドメインが検証されました。
会社への DirSyncEnabled フラグの設定 | ディレクトリの Azure AD Sync を有効にするプロパティが設定されました。
パスワード ポリシーの設定 | ユーザーのパスワードの長さと文字の制約が設定されました。
会社情報の設定 | 会社レベルの情報が更新されました。詳細については、[Get-MsolCompanyInformation](https://msdn.microsoft.com/library/azure/dn194126.aspx) PowerShell コマンドレットを参照してください。

<!---

List of events that still need descriptions:

Restore Application
Set String Auth Policy
Promote tenant to partner

--->

## 監査レポートの保持
Azure AD の監査レポートのイベントは、180 日間保持されます。レポートの保持の詳細については、「[Azure Active Directory Report Retention Policies (Azure Active Directory レポートの保持ポリシー)](active-directory-reporting-retention.md)」をご覧ください。

監査イベントを長期間保存する目的のユーザーの場合は、Reporting API を使用して監査イベントを定期的に別のデータ ストアにプルできます。詳細については、「[Getting Started with the Reporting API (Reporting API の概要)](active-directory-reporting-api-getting-started.md)」をご覧ください。

## 各監査イベントに含まれるプロパティ

プロパティ | 説明
------------- | --------------------------------------------------------------
日付と時刻 | 監査イベントが発生した日時です
アクター | アクションを実行したユーザーまたはサービス プリンシパルです
アクション | 実行されたアクションです
ターゲット | アクションの実行対象になったユーザーまたはサービス プリンシパルです


## 「ユーザーの更新」属性
「ユーザーの更新」監査イベントには、更新されたユーザー属性に関する追加情報が含まれます。各属性について、更新前の値と新しい値の両方が含まれます。

属性 | 説明
------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------
AccountEnabled | ユーザーはサインインを許可されています。
AssignedLicense | ユーザーに割り当てられているすべてのライセンスです。
AssignedPlan | ユーザーに割り当てられたライセンスによるサービス プランです。
LicenseAssignmentDetail | ユーザーに割り当てられているライセンスの詳細です。たとえば、グループ ベースのライセンスが含まれていた場合、これにはライセンスを許可したグループが含まれます。
Mobile | ユーザーの携帯電話の番号です。
OtherMail | ユーザーの代替電子メール アドレスです。
OtherMobile | ユーザーの代替携帯電話番号です。
StrongAuthenticationMethod | ユーザーによって構成された Multi-Factor Authentication の検証方法のリストです。音声通話、SMS、モバイル アプリからの検証コードなど。
StrongAuthenticationRequirement | このユーザーに対して、Multi-Factor Authentication が適用されている、有効になっている、無効になっているかどうかです。
StrongAuthenticationUserDetails | Multi-Factor Authentication およびパスワード リセットの検証に使用されるユーザーの電話番号、代替電話番号、電子メール アドレスです。
TelephoneNumber | ユーザーの電話番号です。

監査レコードは、多くの法令遵守に必要な管理です。Azure Active Directory 監査レポートを使用してコンプライアンス規制に対応するお客様の場合、お客様がエクスポートした監査レポートのコピーにこのヘルプ トピックのコピーを添えて提出し、レポートの詳細な説明に役立てることをお勧めします。Azure が現在を満たしているコンプライアンス規制についての情報を監査担当者が必要としている場合は、監査担当者に Microsoft Azure セキュリティ センターの[コンプライアンス ページ](https://azure.microsoft.com/support/trust-center/compliance/)を紹介してください。

<!---HONumber=AcomDC_0727_2016-->