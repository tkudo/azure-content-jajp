<properties
	pageTitle="Azure Notification Hubs から Android へのプッシュ通知の送信 | Microsoft Azure"
	description="このチュートリアルでは、Azure Notification Hubs を使用して Android デバイスにプッシュ通知を送信する方法について学習します。"
	services="notification-hubs"
	documentationCenter="android"
	keywords="プッシュ通知,プッシュ通知,Android プッシュ通知"
	authors="wesmc7777"
	manager="erikre"
	editor=""/>
<tags
	ms.service="notification-hubs"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-android"
	ms.devlang="java"
	ms.topic="hero-article"
	ms.date="07/05/2016"
	ms.author="wesmc"/>

# Azure Notification Hubs から Android へのプッシュ通知の送信

[AZURE.INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

##Overview

> [AZURE.IMPORTANT] このトピックでは、Google Cloud Messaging (GCM) を使用したプッシュ通知について説明します。Google の Firebase Cloud Messaging (FCM) を使用している場合は、[Azure Notification Hubs と FCM を使用した Android へのプッシュ通知の送信](notification-hubs-android-push-notification-google-fcm-get-started.md)に関する記事を参照してください。

このチュートリアルでは、Azure Notification Hubs を使用して Android アプリケーションにプッシュ通知を送信する方法について説明します。Google Cloud Messaging (GCM) を使用してプッシュ通知を受信する空の Android アプリケーションを作成します。

[AZURE.INCLUDE [notification-hubs-hero-slug](../../includes/notification-hubs-hero-slug.md)]

このチュートリアルの完成したコードについては、GitHub の[こちら](https://github.com/Azure/azure-notificationhubs-samples/tree/master/Android/GetStarted)からダウンロードできます。


##前提条件

> [AZURE.IMPORTANT] このチュートリアルを完了するには、アクティブな Azure アカウントが必要です。アカウントがない場合は、無料試用アカウントを数分で作成することができます。詳細については、[Azure の無料試用版サイト](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A643EE910&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fja-JP%2Fdocumentation%2Farticles%2Fnotification-hubs-android-get-started)を参照してください。

上記のアクティブな Azure アカウント以外に、このチュートリアルで必要となるのは、最新版の [Android Studio](http://go.microsoft.com/fwlink/?LinkId=389797) のみです。

このチュートリアルを完了することは、Android アプリケーションの他のすべての Notification Hubs チュートリアルの前提条件です。

##Google Cloud Messaging をサポートするプロジェクトを作成する

[AZURE.INCLUDE [mobile-services-enable-Google-cloud-messaging](../../includes/mobile-services-enable-google-cloud-messaging.md)]


##新しい Notification Hub を構成する


[AZURE.INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]


&emsp;&emsp;6.**[設定]** ブレードで、**[Notification Services]**、[**Google (GCM)]** の順に選択します。API キーを入力して、**[保存]** をクリックします。

&emsp;&emsp;![Azure Notification Hubs - Google (GCM)](./media/notification-hubs-android-get-started/notification-hubs-gcm-api.png)

これで、通知ハブが GCM と連動するように構成されました。接続文字列を使用して、プッシュ通知の受信と送信の両方にアプリを登録できます。

##<a id="connecting-app"></a>通知ハブにアプリを接続する

###新しい Android プロジェクトを作成する

1. Android Studio で新しい Android Studio プロジェクトを開始する

   	![Android Studio - new project][13]

2. **[電話とタブレット]** フォーム ファクターを選択し、サポートする **[最低限の SDK]** を選択します。その後、**[次へ]** をクリックします。

   	![Android Studio - project creation workflow][14]

3. メイン アクティビティに **[Empty Activity (空のアクティビティ)]** を選択し、**[Next (次へ)]** をクリックして、**[Finish (完了)]** をクリックします。

###プロジェクトへの Google Play Services の追加

[AZURE.INCLUDE [Play サービスの追加](../../includes/notification-hubs-android-studio-add-google-play-services.md)]

###Azure Notification Hubs ライブラリを追加する


1. **[app (アプリ)]** の `Build.Gradle` ファイルの **dependencies** セクションに次の行を追加します。

		compile 'com.microsoft.azure:notification-hubs-android-sdk:0.4@aar'
		compile 'com.microsoft.azure:azure-notifications-handler:1.0.1@aar'

2. **[dependencies]** セクションの後に次のリポジトリを追加します。

		repositories {
		    maven {
		        url "http://dl.bintray.com/microsoftazuremobile/SDK"
		    }
		}

### AndroidManifest.xml ファイルを更新する


1. GCM をサポートするには、[Google のインスタンス ID API](https://developers.google.com/instance-id/) を利用し、[登録トークンの取得](https://developers.google.com/cloud-messaging/android/client#sample-register)に使用されるインスタンス ID リスナー サービスをコードに実装する必要があります。このチュートリアルでは、クラスに `MyInstanceIDService` という名前を付けます。
 
	次のサービス定義を AndroidManifest.xml ファイルの `<application>` タグ内に追加します。`<your package>` プレースホルダーを、`AndroidManifest.xml` ファイルの先頭にある実際のパッケージ名に置き換えます。

		<service android:name="<your package>.MyInstanceIDService" android:exported="false">
		    <intent-filter>
		        <action android:name="com.google.android.gms.iid.InstanceID"/>
		    </intent-filter>
		</service>


2. インスタンス ID API から GCM 登録トークンを受け取ったら、それを使用して [Azure 通知ハブに登録](notification-hubs-push-notification-registration-management.md)します。この登録はバックグラウンドで `RegistrationIntentService` という名前の `IntentService` を利用してサポートします。このサービスは [GCM 登録トークンの更新](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens)も行います。
 
	次のサービス定義を AndroidManifest.xml ファイルに追加します。`<application>` タグの中に追加します。`<your package>` プレースホルダーを、`AndroidManifest.xml` ファイルの先頭にある実際のパッケージ名に置き換えます。

        <service
            android:name="<your package>.RegistrationIntentService"
            android:exported="false">
        </service>



3. 通知を受信するレシーバーも定義します。次のレシーバー定義を AndroidManifest.xml ファイルの `<application>` タグ内に追加します。`<your package>` プレースホルダーを、`AndroidManifest.xml` ファイルの先頭にある実際のパッケージ名に置き換えます。

		<receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
		    android:permission="com.google.android.c2dm.permission.SEND">
		    <intent-filter>
		        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
		        <category android:name="<your package name>" />
		    </intent-filter>
		</receiver>



4. 次の必要な GCM 関連のアクセス許可を `</application>` タグの下に追加します。`<your package>` は、`AndroidManifest.xml` ファイルの先頭にあるパッケージ名に必ず置き換えてください。

	これらのアクセス許可に関する詳細については、[Android 用 GCM Client アプリの設定](https://developers.google.com/cloud-messaging/android/client#manifest)に関するページを参照してください。

		<uses-permission android:name="android.permission.INTERNET"/>
		<uses-permission android:name="android.permission.GET_ACCOUNTS"/>
		<uses-permission android:name="android.permission.WAKE_LOCK"/>
		<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

		<permission android:name="<your package>.permission.C2D_MESSAGE" android:protectionLevel="signature" />
		<uses-permission android:name="<your package>.permission.C2D_MESSAGE"/>


### コードを追加する


1. [プロジェクト] ビューで、**[アプリ]**、**[src]**、** [main]**、**[java]** と展開します。**[java]** のパッケージ フォルダーを右クリックして、**[新規]**、**[Java Class (Java クラス)]** と順にクリックします。`NotificationSettings` という名前の新しいクラスを追加します。

	![Android Studio - new Java class][6]

	以下の `NotificationSettings` クラスのコード内にある次の 3 つのプレースホルダーを忘れずに更新してください。
	* **SenderId**: [Google Cloud Console](http://cloud.google.com/console) で取得しておいたプロジェクト番号。
	* **HubListenConnectionString**: ハブの **DefaultListenAccessSignature** 接続文字列。接続文字列をコピーするには、[Azure ポータル]で、ハブの **[設定]** ブレードにある **[アクセス ポリシー]** をクリックします。
	* **HubName**: [Azure ポータル]のハブ ブレードに表示される通知ハブの名前を使用します。

	`NotificationSettings` のコードは次のとおりです。

		public class NotificationSettings {
		    public static String SenderId = "<Your project number>";
		    public static String HubName = "<Your HubName>";
		    public static String HubListenConnectionString = "<Your default listen connection string>";
		}

2. 上記の手順で `MyInstanceIDService` という名前の別の新しいクラスを追加します。これはインスタンス ID リスナー サービスの実装になります。

	このクラスのコードでは `IntentService` を呼び出し、バックグラウンドで [GCM トークンを更新](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens)します。

		import android.content.Intent;
		import android.util.Log;
		import com.google.android.gms.iid.InstanceIDListenerService;
		
		
		public class MyInstanceIDService extends InstanceIDListenerService {
		
		    private static final String TAG = "MyInstanceIDService";
		
		    @Override
		    public void onTokenRefresh() {
		
		        Log.i(TAG, "Refreshing GCM Registration Token");
		
		        Intent intent = new Intent(this, RegistrationIntentService.class);
		        startService(intent);
		    }
		};


3. `RegistrationIntentService` という名前の別の新しいクラスをプロジェクトに追加します。これが [GCM トークンの更新](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens)と[通知ハブへの登録](notification-hubs-push-notification-registration-management.md)を処理する `IntentService` の実装になります。

	次のコードをこのクラスに使用します。

		import android.app.IntentService;
		import android.content.Intent;
		import android.content.SharedPreferences;
		import android.preference.PreferenceManager;
		import android.util.Log;
		
		import com.google.android.gms.gcm.GoogleCloudMessaging;
		import com.google.android.gms.iid.InstanceID;
		import com.microsoft.windowsazure.messaging.NotificationHub;
		
		public class RegistrationIntentService extends IntentService {
		
		    private static final String TAG = "RegIntentService";
		
		    private NotificationHub hub;
		
		    public RegistrationIntentService() {
		        super(TAG);
		    }
		
		    @Override
		    protected void onHandleIntent(Intent intent) {		
		        SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
		        String resultString = null;
		        String regID = null;
		
		        try {
		            InstanceID instanceID = InstanceID.getInstance(this);
		            String token = instanceID.getToken(NotificationSettings.SenderId,
		                    GoogleCloudMessaging.INSTANCE_ID_SCOPE);		
		            Log.i(TAG, "Got GCM Registration Token: " + token);
		
		            // Storing the registration id that indicates whether the generated token has been
		            // sent to your server. If it is not stored, send the token to your server,
		            // otherwise your server should have already received the token.
		            if ((regID=sharedPreferences.getString("registrationID", null)) == null) {		
		                NotificationHub hub = new NotificationHub(NotificationSettings.HubName,
		                        NotificationSettings.HubListenConnectionString, this);
		                Log.i(TAG, "Attempting to register with NH using token : " + token);

		                regID = hub.register(token).getRegistrationId();

		                // If you want to use tags...
						// Refer to : https://azure.microsoft.com/ja-JP/documentation/articles/notification-hubs-routing-tag-expressions/
		                // regID = hub.register(token, "tag1", "tag2").getRegistrationId();

		                resultString = "Registered Successfully - RegId : " + regID;
		                Log.i(TAG, resultString);		
		                sharedPreferences.edit().putString("registrationID", regID ).apply();
		            } else {
		                resultString = "Previously Registered Successfully - RegId : " + regID;
		            }
		        } catch (Exception e) {
		            Log.e(TAG, resultString="Failed to complete token refresh", e);
		            // If an exception happens while fetching the new token or updating our registration data
		            // on a third-party server, this ensures that we'll attempt the update at a later time.
		        }
		
		        // Notify UI that registration has completed.
		        if (MainActivity.isVisible) {
		            MainActivity.mainActivity.ToastNotify(resultString);
		        }
		    }
		}


		
4. `MainActivity` クラスで、クラス宣言の上に次の `import` ステートメントを追加します。

		import com.google.android.gms.common.ConnectionResult;
		import com.google.android.gms.common.GoogleApiAvailability;
		import com.google.android.gms.gcm.*;
		import com.microsoft.windowsazure.notifications.NotificationsManager;
		import android.util.Log;
		import android.widget.TextView;
		import android.widget.Toast;

5. クラスの最上位に、次のプライベート メンバーを追加します。これらを利用し、[Google の推奨事項に従って Google Play サービスが利用可能かどうかを確認](https://developers.google.com/android/guides/setup#ensure_devices_have_the_google_play_services_apk)します。

	    public static MainActivity mainActivity;
    	public static Boolean isVisible = false;	
		private GoogleCloudMessaging gcm;
	    private static final int PLAY_SERVICES_RESOLUTION_REQUEST = 9000;

6. `MainActivity` クラスに、Google Play Services が利用可能かどうかを確認する次のメソッドを追加します。

	    /**
	     * Check the device to make sure it has the Google Play Services APK. If
	     * it doesn't, display a dialog that allows users to download the APK from
	     * the Google Play Store or enable it in the device's system settings.
	     */
	    private boolean checkPlayServices() {
	        GoogleApiAvailability apiAvailability = GoogleApiAvailability.getInstance();
	        int resultCode = apiAvailability.isGooglePlayServicesAvailable(this);
	        if (resultCode != ConnectionResult.SUCCESS) {
	            if (apiAvailability.isUserResolvableError(resultCode)) {
	                apiAvailability.getErrorDialog(this, resultCode, PLAY_SERVICES_RESOLUTION_REQUEST)
	                        .show();
	            } else {
	                Log.i(TAG, "This device is not supported by Google Play Services.");
	                ToastNotify("This device is not supported by Google Play Services.");
	                finish();
	            }
	            return false;
	        }
	        return true;
	    }


7. `MainActivity` クラスに次のコードを追加します。このコードは、Google Play Services を確認してから、`IntentService` を呼び出して GCM 登録トークンを取得し、通知ハブに登録します。

	    public void registerWithNotificationHubs()
	    {
	        Log.i(TAG, " Registering with Notification Hubs");
	
	        if (checkPlayServices()) {
	            // Start IntentService to register this application with GCM.
	            Intent intent = new Intent(this, RegistrationIntentService.class);
	            startService(intent);
	        }
	    }


8. `MainActivity` クラスの `OnCreate` メソッドで、アクティビティの作成時に登録プロセスを開始するために次のコードを追加します。

	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	
	        mainActivity = this;
	        NotificationsManager.handleNotifications(this, NotificationSettings.SenderId, MyHandler.class);
	        registerWithNotificationHubs();
	    }


9. 次の追加メソッドを `MainActivity` に追加して、アプリの状態を確認してアプリ内で報告します。

	    @Override
	    protected void onStart() {
	        super.onStart();
	        isVisible = true;
	    }
	
	    @Override
	    protected void onPause() {
	        super.onPause();
	        isVisible = false;
	    }
	
	    @Override
	    protected void onResume() {
	        super.onResume();
	        isVisible = true;
	    }
	
	    @Override
	    protected void onStop() {
	        super.onStop();
	        isVisible = false;
	    }
	
	    public void ToastNotify(final String notificationMessage) {
	        runOnUiThread(new Runnable() {
	            @Override
	            public void run() {
	                Toast.makeText(MainActivity.this, notificationMessage, Toast.LENGTH_LONG).show();
	                TextView helloText = (TextView) findViewById(R.id.text_hello);
	                helloText.setText(notificationMessage);
	            }
	        });
	    }


10. `ToastNotify` メソッドでは *"Hello World"* `TextView` コントロールを使用し、アプリで状態と通知を継続的に報告します。activity\_main.xml レイアウトでは、そのコントロールに次の ID を追加します。

        android:id="@+id/text_hello"

11. 次に、AndroidManifest.xml で定義したレシーバーのサブクラスを追加します。`MyHandler` という名前の別の新しいクラスをプロジェクトに追加します。

12. 次の import ステートメントを `MyHandler.java` の先頭に追加します。

		import android.app.NotificationManager;
		import android.app.PendingIntent;
		import android.content.Context;
		import android.content.Intent;
		import android.os.Bundle;
		import android.support.v4.app.NotificationCompat;
		import com.microsoft.windowsazure.notifications.NotificationsHandler;

13. `MyHandler` クラス用に次のコードを追加して、`com.microsoft.windowsazure.notifications.NotificationsHandler` のサブクラスにします。

	このコードによって `OnReceive` メソッドがオーバーライドされるため、ハンドラーは受信した通知を報告します。ハンドラーは `sendNotification()` メソッドを使用して Android の通知マネージャーにもプッシュ通知を送信します。`sendNotification()` メソッドは、アプリが動作していないときに通知を受信した場合に実行される必要があります。

		public class MyHandler extends NotificationsHandler {
		    public static final int NOTIFICATION_ID = 1;
		    private NotificationManager mNotificationManager;
		    NotificationCompat.Builder builder;
		    Context ctx;
		
		    @Override
		    public void onReceive(Context context, Bundle bundle) {
		        ctx = context;
		        String nhMessage = bundle.getString("message");
		        sendNotification(nhMessage);
		        if (MainActivity.isVisible) {
		            MainActivity.mainActivity.ToastNotify(nhMessage);
		        }
		    }
		
		    private void sendNotification(String msg) {
		
		        Intent intent = new Intent(ctx, MainActivity.class);
		        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
		
		        mNotificationManager = (NotificationManager)
		                ctx.getSystemService(Context.NOTIFICATION_SERVICE);
		
		        PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
		                intent, PendingIntent.FLAG_ONE_SHOT);
		
		        Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
		        NotificationCompat.Builder mBuilder =
		                new NotificationCompat.Builder(ctx)
		                        .setSmallIcon(R.mipmap.ic_launcher)
		                        .setContentTitle("Notification Hub Demo")
		                        .setStyle(new NotificationCompat.BigTextStyle()
		                                .bigText(msg))
		                        .setSound(defaultSoundUri)
		                        .setContentText(msg);
		
		        mBuilder.setContentIntent(contentIntent);
		        mNotificationManager.notify(NOTIFICATION_ID, mBuilder.build());
		    }
		}


14. Android Studio のメニュー バーで、**[ビルド]**、**[プロジェクトのリビルド]** の順にクリックし、コードにエラーがないことを確認します。

##プッシュ通知を送信する

[Azure ポータル]から通知を送信して、アプリでプッシュ通知の受信テストを行うことができます。以下に示す、ハブ ブレードの **[トラブルシューティング]** セクションを参照してください。

![Azure Notification Hubs - Test Send](./media/notification-hubs-android-get-started/notification-hubs-test-send.png)

[AZURE.INCLUDE [notification-hubs-sending-notifications-from-the-portal](../../includes/notification-hubs-sending-notifications-from-the-portal.md)]

## (省略可能) アプリから直接プッシュ通知を送信する

通常は、バックエンド サーバーを使用して通知を送信します。場合によっては、クライアント アプリケーションから直接プッシュ通知を送信できることが必要になります。このセクションでは、[Azure Notification Hubs REST API](https://msdn.microsoft.com/library/azure/dn223264.aspx) を使用してクライアントから通知を送信する方法を説明します。

1. Android Studio の [プロジェクト] ビューで **[アプリ]**、**[src]**、**[main]**、**[res]**、**[layout]** の順に展開します。`activity_main.xml` レイアウト ファイルを開き、**[Text (テキスト)]** タブをクリックしてファイルのテキストの内容を更新します。次のコードで更新します。これにより、通知ハブにプッシュ通知メッセージを送信するための新しい `Button` コントロールと `EditText` コントロールが追加されます。このコードは一番下の `</RelativeLayout>` のすぐ前に追加します。

	    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/send_button"
        android:id="@+id/sendbutton"
        android:layout_centerVertical="true"
        android:layout_centerHorizontal="true"
        android:onClick="sendNotificationButtonOnClick" />

	    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/editTextNotificationMessage"
        android:layout_above="@+id/sendbutton"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="42dp"
        android:hint="@string/notification_message_hint" />

2. Android Studio の [Project] ビューで **[App]**、**[src]**、**[main]**、**[res]**、**[values]** の順に展開します。`strings.xml` ファイルを開き、新しい `Button` コントロールと `EditText` コントロールで参照される文字列の値を追加します。これらはファイルの一番下の `</resources>` のすぐ前に追加します。

        <string name="send_button">Send Notification</string>
        <string name="notification_message_hint">Enter notification message text</string>


3. `NotificationSetting.java` ファイルで、次の設定を `NotificationSettings` クラスに追加します。

	`HubFullAccess` をハブの **DefaultFullSharedAccessSignature** 接続文字列で更新します。この接続文字列は [Azure ポータル]からコピーできます。これには、通知ハブの **[設定]** ブレードで **[アクセス ポリシー]** をクリックします。

		public static String HubFullAccess = "<Enter Your DefaultFullSharedAccess Connection string>";

4. `MainActivity.java` ファイルで、`MainActivity` クラスの上に次の `import` ステートメントを追加します。

		import java.io.BufferedOutputStream;
		import java.io.BufferedReader;
		import java.io.InputStreamReader;
		import java.io.OutputStream;
		import java.net.HttpURLConnection;
		import java.net.URL;
		import java.net.URLEncoder;
		import javax.crypto.Mac;
		import javax.crypto.spec.SecretKeySpec;
		import android.util.Base64;
		import android.view.View;
		import android.widget.EditText;

6. `MainActivity.java` ファイルで、`MainActivity` クラスの先頭に次のメンバーを追加します。

	    private String HubEndpoint = null;
	    private String HubSasKeyName = null;
	    private String HubSasKeyValue = null;

6. 通知ハブにメッセージを送信するには、POST 要求を認証するためのソフトウェア アクセス署名 (SaS) トークンを作成する必要があります。これを実行するには、接続文字列からキーのデータを解析し、「[共通概念](http://msdn.microsoft.com/library/azure/dn495627.aspx)」の REST API リファレンスで説明したように SaS トークンを作成します。次のコードでは、実装例を示します。

	`MainActivity.java` で、接続文字列を解析する次のメソッドを `MainActivity` クラスに追加します。

	    /**
	     * Example code from http://msdn.microsoft.com/library/azure/dn495627.aspx
	     * to parse the connection string so a SaS authentication token can be
	     * constructed.
	     *
	     * @param connectionString This must be the DefaultFullSharedAccess connection
	     *                         string for this example.
	     */
	    private void ParseConnectionString(String connectionString)
	    {
	        String[] parts = connectionString.split(";");
	        if (parts.length != 3)
	            throw new RuntimeException("Error parsing connection string: "
	                    + connectionString);
	
	        for (int i = 0; i < parts.length; i++) {
	            if (parts[i].startsWith("Endpoint")) {
	                this.HubEndpoint = "https" + parts[i].substring(11);
	            } else if (parts[i].startsWith("SharedAccessKeyName")) {
	                this.HubSasKeyName = parts[i].substring(20);
	            } else if (parts[i].startsWith("SharedAccessKey")) {
	                this.HubSasKeyValue = parts[i].substring(16);
	            }
	        }
	    }


7. `MainActivity.java` で、SAS 認証トークンを作成する次のメソッドを `MainActivity` クラスに追加します。

	    /**
	     * Example code from http://msdn.microsoft.com/library/azure/dn495627.aspx to
	     * construct a SaS token from the access key to authenticate a request.
	     *
	     * @param uri The unencoded resource URI string for this operation. The resource
	     *            URI is the full URI of the Service Bus resource to which access is
	     *            claimed. For example,
	     *            "http://<namespace>.servicebus.windows.net/<hubName>"
	     */
	    private String generateSasToken(String uri) {
	
	        String targetUri;
	        String token = null;
	        try {
	            targetUri = URLEncoder
	                    .encode(uri.toString().toLowerCase(), "UTF-8")
	                    .toLowerCase();
	
	            long expiresOnDate = System.currentTimeMillis();
	            int expiresInMins = 60; // 1 hour
	            expiresOnDate += expiresInMins * 60 * 1000;
	            long expires = expiresOnDate / 1000;
	            String toSign = targetUri + "\n" + expires;
	
	            // Get an hmac_sha1 key from the raw key bytes
	            byte[] keyBytes = HubSasKeyValue.getBytes("UTF-8");
	            SecretKeySpec signingKey = new SecretKeySpec(keyBytes, "HmacSHA256");
	
	            // Get an hmac_sha1 Mac instance and initialize with the signing key
	            Mac mac = Mac.getInstance("HmacSHA256");
	            mac.init(signingKey);
	
	            // Compute the hmac on input data bytes
	            byte[] rawHmac = mac.doFinal(toSign.getBytes("UTF-8"));
	
	            // Using android.util.Base64 for Android Studio instead of
	            // Apache commons codec
	            String signature = URLEncoder.encode(
	                    Base64.encodeToString(rawHmac, Base64.NO_WRAP).toString(), "UTF-8");
	
	            // Construct authorization string
	            token = "SharedAccessSignature sr=" + targetUri + "&sig="
	                    + signature + "&se=" + expires + "&skn=" + HubSasKeyName;
	        } catch (Exception e) {
	            if (isVisible) {
	                ToastNotify("Exception Generating SaS : " + e.getMessage().toString());
	            }
	        }
	
	        return token;
	    }



8. `MainActivity.java` で、次のメソッドを `MainActivity` クラスに追加します。このメソッドは **[Send Notification]** ボタンのクリックを処理し、組み込みの REST API を使用してハブにプッシュ通知メッセージを送信します。

	    /**
	     * Send Notification button click handler. This method parses the
	     * DefaultFullSharedAccess connection string and generates a SaS token. The
	     * token is added to the Authorization header on the POST request to the
	     * notification hub. The text in the editTextNotificationMessage control
	     * is added as the JSON body for the request to add a GCM message to the hub.
	     *
	     * @param v
	     */
	    public void sendNotificationButtonOnClick(View v) {
	        EditText notificationText = (EditText) findViewById(R.id.editTextNotificationMessage);
	        final String json = "{"data":{"message":"" + notificationText.getText().toString() + ""}}";
	
	        new Thread()
	        {
	            public void run()
	            {
	                try
	                {
	                    // Based on reference documentation...
	                    // http://msdn.microsoft.com/library/azure/dn223273.aspx
	                    ParseConnectionString(NotificationSettings.HubFullAccess);
	                    URL url = new URL(HubEndpoint + NotificationSettings.HubName +
	                            "/messages/?api-version=2015-01");
	
	                    HttpURLConnection urlConnection = (HttpURLConnection)url.openConnection();
	
	                    try {
	                        // POST request
	                        urlConnection.setDoOutput(true);
	
	                        // Authenticate the POST request with the SaS token
	                        urlConnection.setRequestProperty("Authorization", 
								generateSasToken(url.toString()));
	
	                        // Notification format should be GCM
	                        urlConnection.setRequestProperty("ServiceBusNotification-Format", "gcm");
	
	                        // Include any tags
	                        // Example below targets 3 specific tags
	                        // Refer to : https://azure.microsoft.com/ja-JP/documentation/articles/notification-hubs-routing-tag-expressions/
	                        // urlConnection.setRequestProperty("ServiceBusNotification-Tags", 
							//		"tag1 || tag2 || tag3");
	
	                        // Send notification message
	                        urlConnection.setFixedLengthStreamingMode(json.length());
	                        OutputStream bodyStream = new BufferedOutputStream(urlConnection.getOutputStream());
	                        bodyStream.write(json.getBytes());
	                        bodyStream.close();
	
	                        // Get reponse
	                        urlConnection.connect();
	                        int responseCode = urlConnection.getResponseCode();
	                        if ((responseCode != 200) && (responseCode != 201)) {
                                BufferedReader br = new BufferedReader(new InputStreamReader((urlConnection.getErrorStream())));
                                String line;
                                StringBuilder builder = new StringBuilder("Send Notification returned " +
                                        responseCode + " : ")  ;
                                while ((line = br.readLine()) != null) {
                                    builder.append(line);
                                }

                                ToastNotify(builder.toString());
	                        }
	                    } finally {
	                        urlConnection.disconnect();
	                    }
	                }
	                catch(Exception e)
	                {
	                    if (isVisible) {
	                        ToastNotify("Exception Sending Notification : " + e.getMessage().toString());
	                    }
	                }
	            }
	        }.start();
	    }



##アプリケーションのテスト

####エミュレーターでのプッシュ通知

エミュレーターの内部でプッシュ通知をテストする場合は、エミュレーター イメージがアプリケーション用に選択した Google API レベルをサポートしていることを確認してください。イメージがネイティブの Google API をサポートしていない場合は、最終的に **SERVICE\_NOT\_AVAILABLE** 例外を受け取ることになります。

上記に加えて、**[設定]**、**[アカウント]** で、実行しているエミュレーターに Google アカウントを追加していることを確認してください。それ以外の場合、GCM での登録の試行が **AUTHENTICATION\_FAILED** 例外になる可能性があります。

####アプリケーションの実行

1. アプリケーションを実行し、登録 ID の登録完了が報告されていることを確認します。

   	![Testing on Android - Channel registration][18]

2. ハブに登録されているすべての Android デバイスに送信される通知メッセージを入力します。

   	![Testing on Android - sending a message][19]

3. **[Send Notification (通知の送信)]** を押します。アプリケーションが実行されているすべてのデバイスで、プッシュ通知メッセージを示す `AlertDialog` インスタンスが表示されます。実行中のアプリケーションがなくても事前にプッシュ通知に登録されているデバイスでは、Android 通知マネージャーで通知が受信されます。左上隅から下へスワイプすると、通知を表示できます。

   	![Testing on Android - notifications][21]

##次のステップ

次のステップとして「[Notification Hubs を使用したユーザーへのプッシュ通知]」チュートリアルをお勧めします。このチュートリアルでは、特定のユーザーに対してタグを使用して ASP.NET バックエンドから通知を送信する方法について説明しています。

対象グループごとにユーザーを区分する場合は、「[Notification Hubs を使用したニュース速報の送信]」のチュートリアルを参照してください。

Notification Hubs の全般的な情報については、「[Notification Hubs の概要]」を参照してください。

<!-- Images. -->
[6]: ./media/notification-hubs-android-get-started/notification-hub-android-new-class.png

[12]: ./media/notification-hubs-android-get-started/notification-hub-connection-strings.png

[13]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-new-project.png
[14]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-choose-form-factor.png
[15]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app4.png
[16]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app5.png
[17]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app6.png

[18]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-registered.png
[19]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-set-message.png

[20]: ./media/notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-received-message.png
[22]: ./media/notification-hubs-android-get-started/notification-hub-scheduler1.png
[23]: ./media/notification-hubs-android-get-started/notification-hub-scheduler2.png
[29]: ./media/mobile-services-android-get-started-push/mobile-eclipse-import-Play-library.png

[30]: ./media/notification-hubs-android-get-started/notification-hubs-debug-hub-gcm.png

[31]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-add-ui.png


<!-- URLs. -->
[Get started with push notifications in Mobile Services]: ../mobile-services-javascript-backend-android-get-started-push.md
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[Referencing a library project]: http://go.microsoft.com/fwlink/?LinkId=389800
[Azure Classic Portal]: https://manage.windowsazure.com/
[Notification Hubs の概要]: http://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs を使用したユーザーへのプッシュ通知]: notification-hubs-aspnet-backend-android-notify-users.md
[Notification Hubs を使用したニュース速報の送信]: notification-hubs-aspnet-backend-android-breaking-news.md
[Azure ポータル]: https://portal.azure.com

<!---HONumber=AcomDC_0810_2016-->