# ラボ 9: DirectLine でボットをテストする

## 紹介

状況によっては、ボットと直接通信する必要があります。たとえば、ホストされたボットを使用して機能テストを実行する場合です。ボットと独自のクライアント アプリケーション間の通信は、[Direct Line API](https://docs.microsoft.com/ja-jp/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts) を使用して実行できます。

さらに、他のチャネルを使用したり、ボットをさらにカスタマイズしたい場合があります。その場合は、DirectLine を使用してカスタム ボットと通信できます。

Microsoft Bot Framework DirectLine ボットは、独自の設計のカスタム クライアントで機能できるボットです。DirectLine ボットは通常のボットに似ています。提供されたチャネルを使用する必要はありません。必要に応じて、DirectLine クライアントに書き込むことができます。Android クライアント、iOS クライアント、またはコンソール ベースのクライアント アプリケーションを作成できます。

このハンズオン ラボでは、Direct Line API に関連する主要な概念を紹介します。

## ラボ 9.0: 前提条件

このラボは、[ラボ 3](../Lab3-Basic_Filter_Bot/02-Basic_Filter_Bot.md)でボットをビルドして公開したという前提で始まります。次のラボで成功するためには、このラボを実行することをお勧めします。そうでない場合、ニーズによっては、すべての演習を注意深く読み、コードの一部を調るか、独自のアプリケーションで使用するだけで十分です。

また、[ラボ 4](../Lab4-Log_Chat/02-Logging_Chat.md) を完了したことを前提としますが、ログに関するラボを完了しなくても、このラボを完了できるはずです。

### キーの収集

最後のいくつかのラボでは、さまざまなキーを収集しました。スターター プロジェクトを開始点として使用している場合、それらのほとんどが必要になります。

>_Keys_
>
>- Cognitive Services API URL:
>- Cognitive Services API キー:
>- LUIS API エンドポイント:
>- LUIS API キー:
>- LUIS API アプリ ID:
>- ボット アプリ名:
>- ボット アプリ ID:
>- ボット アプリのパスワード:
>- Azure Storage 接続文字列:
>- Cosmos DB URI:
>- Cosmos DB キー:
>- DirectLineキー:

**appsettings.json** ファイルを必要なすべての値で更新してください。

## ラボ 9.1: ボットを公開する

1. **PictureBot** ソリューションを開きます

1. プロジェクトを右クリックして、「**公開**」 を選択します

1. 公開ダイアログで、「**既存を選択**」を選択し 、「**公開**」をクリックします

1. プロンプトが表示されたら、ラボで使用したアカウントを使用してログインします。

1. 使用しているサブスクリプションを選択します

1. リソース グループを展開し、ラボ 3 で作成したPictureBot アプリ サービスを選択します。

1. 「**Publish**」を選択します。

> **注**: このラボへのアクセスにかかったパスによっては、2回目の公開が必要になる場合があります。  ボットを再公開します。今回は、公開設定を変更して既存のファイルを削除します。  

## ラボ 9.2: DirectLine チャネルの設定

1. ポータルで、公開されている PictureBot **Web アプリ ボット**を見つけて、[**チャンネル**] タブに移動します。

1. 「Direct Line」アイコンを選択します (地球のように見えます)。**デフォルトのサイト**が表示されます。

1. **シークレット キー**については、「**表示**」をクリックして、いずれかのシークレット キーをメモ帳に保存するか、キーを追跡する場所に保存します。

[Direct Line チャネル](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-service-channel-connect-directline?view=azure-bot-service-4.0) と[秘密とトークン](https://docs.microsoft.com/ja-jp/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-3.0#secrets-and-tokens)を有効にする詳細な手順を読むことができます。

## ラボ 9.3: コンソール アプリケーションを作成する

コンソール アプリケーションを作成して、Direct Line を使用してボットに直接接続する方法を理解できるようにします。作成するコンソール クライアント アプリケーションは、2つのスレッドで動作します。プライマリ スレッドではユーザー入力を受け入れ、ボットにメッセージを送信します。セカンダリ スレッドでは、ボットを 1 秒に 1 回ポーリングしてボットからメッセージを取得し、受信したメッセージを表示します。

> **注**: ここでの手順とコードは、ドキュメントの[ベスト プラクティス](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-howto-direct-line?view=azure-bot-service-4.0&tabs=cscreatebot%2Ccsclientapp%2Ccsrunclient)から変更されています。

1. まだ開いていない場合は、Visual Studio で **PictureBot** ソリューションを開きます

1. ソリューション エクスプローラーでソリューションを右クリックし、**「追加」 > 「新しいプロジェクト」** を選択します。

1. **Console App (.NET Core)** を検索して選択し、「**次へ**」をクリックします

1. 名前として「 **PictureBotDL**」と入力します。

1. **「作成」** を選択します。

### NuGet パッケージを PictureBotDL に追加する

1. PictureBotDL プロジェクトを右クリックし、**「NuGet パッケージの管理」** を選択します。

1. 「**参照**」 タブで、次を検索およびインストール/更新します。

- Microsoft.Bot.Connector.DirectLine
- Microsoft.Rest.ClientRuntime

1. **Program.cs**を開く

1. (PictureBotDL 内の) **Program.cs** の内容を次のように置き換えます。

    ```csharp
    using System;
    using System.Diagnostics;
    using System.Linq;
    using System.Threading.Tasks;
    using Microsoft.Bot.Connector.DirectLine;
    using Newtonsoft.Json;
    using Activity = Microsoft.Bot.Connector.DirectLine.Activity;

    namespace PictureBotDL
    {
        class Program
        {
            // ************
            // Replace the following values with your Direct Line secret and the name of your bot name.
            //*************
            private static string directLineSecret = "YourDLSecret";
            private static string botName = "YourBotServiceName";

            // This gives a name to the bot user.
            private static string fromUser = "PictureBotSampleUser";

            static void Main(string[] args)
            {
                StartBotConversation().Wait();
            }


            /// <summary>
            /// Drives the user's conversation with the bot.
            /// </summary>
            /// <returns></returns>
            private static async Task StartBotConversation()
            {
                // Create a new Direct Line client.
                DirectLineClient client = new DirectLineClient(directLineSecret);

                // Start the conversation.
                var conversation = await client.Conversations.StartConversationAsync();

                // Start the bot message reader in a separate thread.
                new System.Threading.Thread(async () => await ReadBotMessagesAsync(client, conversation.ConversationId)).Start();

                // Prompt the user to start talking to the bot.
                Console.Write("Conversation ID: " + conversation.ConversationId + Environment.NewLine);
                Console.Write("Type your message (or \"exit\" to end): ");

                // Loop until the user chooses to exit this loop.
                while (true)
                {
                    // Accept the input from the user.
                    string input = Console.ReadLine().Trim();

                    // Check to see if the user wants to exit.
                    if (input.ToLower() == "exit")
                    {
                        // Exit the app if the user requests it.
                        break;
                    }
                    else
                    {
                        if (input.Length > 0)
                        {
                            // Create a message activity with the text the user entered.
                            Activity userMessage = new Activity
                            {
                                From = new ChannelAccount(fromUser),
                                Text = input,
                                Type = ActivityTypes.Message
                            };

                            // Send the message activity to the bot.
                            await client.Conversations.PostActivityAsync(conversation.ConversationId, userMessage);
                        }
                    }
                }
            }


            /// <summary>
            /// Polls the bot continuously and retrieves messages sent by the bot to the client.
            /// </summary>
            /// <param name="client">The Direct Line client.</param>
            /// <param name="conversationId">The conversation ID.</param>
            /// <returns></returns>
            private static async Task ReadBotMessagesAsync(DirectLineClient client, string conversationId)
            {
                string watermark = null;

                // Poll the bot for replies once per second.
                while (true)
                {
                    // Retrieve the activity set from the bot.
                    var activitySet = await client.Conversations.GetActivitiesAsync(conversationId, watermark);
                    watermark = activitySet?.Watermark;

                    // Extract the activies sent from our bot.
                    var activities = from x in activitySet.Activities
                                    where x.From.Id == botName
                                    select x;

                    // Analyze each activity in the activity set.
                    foreach (Activity activity in activities)
                    {
                        // Display the text of the activity.
                        Console.WriteLine(activity.Text);

                        // Are there any attachments?
                        if (activity.Attachments != null)
                        {
                            // Extract each attachment from the activity.
                            foreach (Attachment attachment in activity.Attachments)
                            {
                                switch (attachment.ContentType)
                                {
                                    // Display a hero card.
                                    case "application/vnd.microsoft.card.hero":
                                        RenderHeroCard(attachment);
                                        break;

                                    // Display the image in a browser.
                                    case "image/png":
                                        Console.WriteLine($"Opening the requested image '{attachment.ContentUrl}'");
                                        Process.Start(attachment.ContentUrl);
                                        break;
                                }
                            }
                        }

                    }

                    // Wait for one second before polling the bot again.
                    await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
                }
            }


            /// <summary>
            /// Displays the hero card on the console.
            /// </summary>
            /// <param name="attachment">The attachment that contains the hero card.</param>
            private static void RenderHeroCard(Attachment attachment)
            {
                const int Width = 70;
                // Function to center a string between asterisks.
                Func<string, string> contentLine = (content) => string.Format($"{{0, -{Width}}}", string.Format("{0," + ((Width + content.Length) / 2).ToString() + "}", content));

                // Extract the hero card data.
                var heroCard = JsonConvert.DeserializeObject<HeroCard>(attachment.Content.ToString());

                // Display the hero card.
                if (heroCard != null)
                {
                    Console.WriteLine("/{0}", new string('*', Width + 1));
                    Console.WriteLine("*{0}*", contentLine(heroCard.Title));
                    Console.WriteLine("*{0}*", new string(' ', Width));
                    Console.WriteLine("*{0}*", contentLine(heroCard.Text));
                    Console.WriteLine("{0}/", new string('*', Width + 1));
                }
            }
        }
    }
    ```

    > **注**: このコードは、次のラボのセクションで使用するいくつかの項目が含まれるように、[ドキュメント](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-howto-direct-line?view=azure-bot-service-4.0&tabs=cscreatebot%2Ccsclientapp%2Ccsrunclient#create-the-console-client-app)から少し変更されました。

1. **Program.cs** で、DirectLineシークレットとボット ID を特定の値に更新します。

    少し時間を取って、このサンプル コードを確認してください。これは、PictureBot に接続して応答を得る方法を理解していることを確認するための良い練習です。

### クライアント アプリを実行します。

1. **PictureBotDL** プロジェクトを右クリックし、「**スタートアップ プロジェクトに設定**」を選択します。

1. 次に、**F5** を押してアプリを実行します。

1. コマンドライン アプリケーションを使用してボットと会話する

    ![コンソール アプリ](../images//consoleapp.png)

    > **注**: 応答がない場合、ボットにエラーがある可能性があります。ボット エミュレーターを使用してボットをローカルでテストし、問題を修正してから再公開します。

    クイック クイズ - 会話 ID を表示する方法は? 次のセクションでは、これが必要な理由について説明します。

## ラボ 9.4: HTTP Get を使用してメッセージを取得する

会話 ID を取得すると、HTTP Get を使用してユーザーとボットのメッセージを取得できるようになります。Rest Client に精通していて経験豊富な方は、お好きなツールをご自由にお使いください。

このラボでは、Postman (Web ベースのクライアント) を使用してメッセージを取得します。

### Postman をダウンロードする

1. [プラットフォームのネイティブ アプリをダウンロードします](https://www.getpostman.com/apps)。単純なアカウントを作成することが必要な場合があります。

### アプリをテストする

Direct Line API を使用すると、クライアントが HTTP Post 要求を発行してボットにメッセージを送信できます。クライアントは WebSocket ストリームを介して、または HTTP GET 要求を発行することによって、ボットからメッセージを受信できます。このラボでは、メッセージを受信するための HTTP Get オプションについて説明します。

GET 要求を行います。また、ヘッダーにヘッダー名 (**Authorization**) とヘッダー値 (**Bearer YourDirectLineSecret**) が含まれていることを確認する必要があります。さらに、{conversationId} を、要求の現在の会話 ID に置き換えることで、コンソール アプリで既存の会話を呼び出します。`https://directline.botframework.com/api/conversations/{conversationId}/messages`.

Postman により、これが非常に簡単に行えます。

- ドロップダウンを使用して「GET」と入力するように要求を変更します。
- 会話 ID で要求 URL を入力します。
- 「Bearer Token」 (ベアラー トークン) と入力するように「Authorization」を変更し、「トークン」ボックスに DirectLine 秘密鍵を入力します。

    ![ベアラー トークン](../images//bearer.png)

1. **Postman** を開く

1. タイプについては、**GET**が選択されていることを確認してください

1. URL に **https://directline.botframework.com/api/conversations/{conversationId}/messages** と入力します 。  converstationId を特定の会話 ID に必ず置き換えてください

1. 「**Auth**」をクリックして、TYPEに「**Bearer Token**」を選択します。

1. 値を **{Your Direct Line Secret}** に設定します

1. 最後に、「**Send**」を選択します。

1. 結果を調べます。

1. コンソール アプリで新しい会話を作成し、イメージを検索します。

1. 新しい会話 ID を使用して、Postman を使って新しい要求を作成します。

1. 返された応答を検査します。  応答の画像配列内に表示されている画像の URL を見つけてください。

    ![イメージ配列の例](../images//imagesarray.png)

## さらに進む

時間が余りましたか? (Postman のときと同様に) 会話を取得するために、端末から cURL (ダウンロード リンク: https://curl.haxx.se/download.html) を利用できますか?

> ヒント: コマンドは`curl -H "Authorization:Bearer {SecretKey}" https://directline.botframework.com/api/conversations/{conversationId}/messages -XGET`のようになります。

## リソース

- [Direct Line API](https://docs.microsoft.com/ja-jp/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts)