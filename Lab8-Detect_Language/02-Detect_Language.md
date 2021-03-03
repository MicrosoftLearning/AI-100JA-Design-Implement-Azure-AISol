# ラボ 8: 言語を検出する

このラボでは、Cognitive Services の言語検出機能をボットに統合します。

## ラボ 8.1: Cognitive Services の URL とキーを取得する

1. [Azure Portal](https://portal.azure.com) を開く

1. リソース グループに移動し、ラボ1で作成した汎用的な Cognitive Services リソースを選択します。

1. 「**リソース管理**」 の 「**キーとエンドポイント**」 メニューを選択し、Cognitive Services リソースの URL とキーを記録します。

## ラボ 8.2: ボットに言語サポートを追加する

1. まだ開いていない場合は、**PictureBot** ソリューションを開きます。

1. プロジェクトを右クリックし、**「Nuget パッケージの管理」** を選択します。

1. 「**参照**」を選択します

1. **Microsoft.Azure.CognitiveServices.Language.TextAnalytics** を検索し、「**インストール**」を選択し、「**同意する**」をクリックします。

1. **Startup.cs** ファイルを開き、次の using ステートメントを追加します。

    ```csharp
    using Azure.AI.TextAnalytics;
    using Azure;
    ```

1. 次のコードを **ConfigureServices** メソッドに追加します。

    ```csharp
    services.AddSingleton<TextAnalyticsClient>(sp =>
    {
        Uri cogsBaseUrl = new Uri(Configuration.GetSection("cogsBaseUrl")?.Value);
        string cogsKey = Configuration.GetSection("cogsKey")?.Value;

        var credentials = new AzureKeyCredential(cogsKey);
        return new TextAnalyticsClient(cogsBaseUrl, credentials);
    });
    ```

1. **PictureBot.cs** ファイルを開き、次の using ステートメントを追加します。

    ```csharp
    using Azure.AI.TextAnalytics;
    ```

1. 次のクラス変数を追加します。

    ```csharp
    private TextAnalyticsClient _textAnalyticsClient;
    ```

1. コンストラクタを変更して、新しい TextAnalyticsClient を含めます。

    ```csharp
    public PictureBot(PictureBotAccessors accessors, LuisRecognizer recognizer, TextAnalyticsClient analyticsClient)
    ```

1. コンストラクター内で、クラス変数を初期化します。

    ```csharp
    _textAnalyticsClient = analyticsClient;
    ```

1. **OnTurnAsync** メソッドに移動し、次のコード行を見つけます。

    ```csharp
    var utterance = turnContext.Activity.Text;
    var state = await _accessors.PictureState.GetAsync(turnContext, () => new PictureState());
    state.UtteranceList.Add(utterance);
    await _accessors.ConversationState.SaveChangesAsync(turnContext);
    ```

1. その後に、次のコード行を追加します

    ```csharp
    //Check the language
    DetectedLanguage detectedLanguage = _textAnalyticsClient.DetectLanguage(turnContext.Activity.Text);
    switch (detectedLanguage.Name)
    {
        case "English":
            break;
        default:
            //throw error
            await turnContext.SendActivityAsync($"I'm sorry, I can only understand English. [{detectedLanguage.Name}]");
            break;
    }
    ```

1. `switch`文の後のコードを、`case "English"`に移動します。

1. 最終的に`OnTurnAsync`は次のようになります。

    ```csharp
    public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type is "message")
        {
            var utterance = turnContext.Activity.Text;
            var state = await _accessors.PictureState.GetAsync(turnContext,() => new PictureState());
            state.UtteranceList.Add(utterance);
            await _accessors.ConversationState.SaveChangesAsync(turnContext);

            //Check the language
            DetectedLanguage detectedLanguage = _textAnalyticsClient.DetectLanguage(turnContext.Activity.Text);
            switch (detectedLanguage.Name)
            {
                    case "English":
                        // Establish dialog context from the conversation state.
                        var dc = await _dialogs.CreateContextAsync(turnContext);
                        // Continue any current dialog.
                        var results = await dc.ContinueDialogAsync(cancellationToken);

                        // Every turn sends a response, so if no response was sent,
                        // then there no dialog is currently active.
                        if (!turnContext.Responded)
                        {
                            // Start the main dialog
                            await dc.BeginDialogAsync("mainDialog", null, cancellationToken);
                        }
                        break;
                    default:
                        //throw error
                        await turnContext.SendActivityAsync($"I'm sorry, I can only understand English. [{detectedLanguage.Name}]");
                        break;
            }
        }
    }
    ```

1. **appsettings.json** ファイルを開き、Cognitive Services の設定を入力します。

    ```csharp
    "cogsBaseUrl": "",
    "cogsKey" :  ""
    ```

1. **F5** を押してボットを開始します

1. ボット エミュレーターを使用して、複数のフレーズを送信し、何が起こるかを確認します。

- こんにちは
- Como Estes?
- Bon Jour!
- Привет
- Hello

## さらに進む

以前のラボで既に LUIS を紹介しているため、LUIS を使用して複数の言語をサポートするために必要な変更を検討してください。  役立つ記事:

- [LUIS の言語と地域のサポート](https://docs.microsoft.com/ja-jp/azure/cognitive-services/luis/luis-language-support)

## リソース

- [例: Text Analytics で言語を検出する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-language-detection)
- [クイック スタート: .NET 用 Text Analytics クライアント ライブラリ](https://docs.microsoft.com/ja-jp/azure/cognitive-services/text-analytics/quickstarts/csharp)

## 次のステップ

- [ラボ 09-01: DirectLine でボットをテストする](../Lab9-Test_Bots_DirectLine/01-Introduction.md)
