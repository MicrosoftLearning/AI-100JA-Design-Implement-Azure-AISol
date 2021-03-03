# ラボ 7: LUIS をボット ダイアログに統合する

## 紹介

このボットは、ユーザーの入力を受け取り、ユーザーの入力に基づいて応答できるようになりました。残念ながら、このボットのコミュニケーション スキルは不安定です。1 つのタイプミス、または単語の言い換えをボットは理解しません。これにより、ユーザーの不満を招くことがあります。[ラボ 6](../Lab6-Implement_LUIS/02-Implement_LUIS.md) で構築した LUIS モデルでボットが自然言語を理解できるようにすることで、ボットの会話能力を大幅に向上させることができます。

LUIS を使用するには、ボットを更新する必要があります。  「Startup.cs」 と 「PictureBot.cs」 を変更することで、これを行うことができます。

> 前提条件: このラボは、[ラボ 3](../Lab3-Basic_Filter_Bot/02-Basic_Filter_Bot.md) に基づいて作成されます。このラボで説明するようにログを実装するには、このラボを実行することをお勧めします。そうでない場合、ニーズによっては、すべての演習を注意深く読み、コードの一部を確認するか、独自のアプリケーションで使用するだけで十分です。

> 注意: Finished フォルダーのコードを使用する場合は、アプリ固有の情報を独自のアプリ ID とエンドポイントに置き換える必要があります。

## ラボ 7.1: 自然言語の理解の追加

### LUIS の Startup.cs への追加

1. まだ開いていない場合は、Visual Studio で **PictureBot** ソリューションを開きます

    > **注** Lab 1 から開始しなかった場合は、 **{GitHubPath}/Lab7-Integrate_LUIS/code/Starter/PictureBot/PictureBot.sln** ソリューションから開始することもできます。
    > すべてのアプリ設定の値を置き換えてください

1. 「**Startup.cs**」 を開き、`ConfigureServices`メソッドを見つけます。ここで、状態アクセサーを作成して登録した後、LUIS 用の追加サービスを追加して、LUIS を追加します。

    下記のように追加します。

    ```csharp
    // Create and register a LUIS recognizer.
    services.AddSingleton(sp =>
    {
        var luisApplication = new LuisApplication(
            Configuration.GetSection("luisAppId")?.Value,
            Configuration.GetSection("luisAppKey")?.Value,
            Configuration.GetSection("luisEndPoint")?.Value);
            // Set the recognizer options depending on which endpoint version you want to use.
            // More details can be found in https://docs.microsoft.com/en-gb/azure/cognitive-services/luis/luis-migration-api-v3
            var recognizerOptions = new LuisRecognizerOptionsV3(luisApplication)
            {
                PredictionOptions = new Microsoft.Bot.Builder.AI.LuisV3.LuisPredictionOptions
                {
                    IncludeAllIntents = true,
                }
            };
        return new LuisRecognizer(recognizerOptions);
    });
    ```

1. **appsettings.json** を変更して、次のプロパティを含めます。 

    ```json
    "luisAppId": "",
    "luisAppKey": "",
    "luisEndPoint": ""
    ```

1. LUISのサイトからアプリ ID、 予測リソースのキー、 エンドポイントURLを取得し、appsettings.jsonに入力します。

    > **注**: .NET SDK の Luis エンドポイント URL は、 **https://{region}.api.cognitive.microsoft.com**のようなもので、その後ろに API やバージョンは必要ありません。

## ラボ 7.2: LUIS の PictureBot の MainDialog への追加

1. **PictureBot.cs** を開きます。最初に行う必要があるのは、`PictureBotAccessors`で行ったのと同様に、LUIS 認識エンジンを初期化することです。`private readonly PictureBotAccessors _accessors;`の下に、次のコードを追加します。

    ```csharp
    private LuisRecognizer _recognizer { get; } = null;
    ```

1. PictureBotファイルに、次の名前空間を追加します。

    ```csharp
    using Microsoft.Bot.Builder.AI.Luis;
    ```

1. **PictureBot** コンストラクターに移動します。

    ```csharp
    public PictureBot(PictureBotAccessors accessors)
    ```

1. 次のように、入力要件（パラメーター `LuisRecognizer recognizer`）を追加します。

    ```csharp
    public PictureBot(PictureBotAccessors accessors, LuisRecognizer recognizer)
    ```

1. コンストラクタの先頭に以下のコードを追加します。

    ```csharp
    _recognizer = recognizer ?? throw new ArgumentNullException(nameof(recognizer));
    ```

1. 以下のusing句を追加します。

    ```csharp
    using System;
    ```

    繰り返しますが、これは`_accessors`のインスタンスを初期化した方法と非常によく似ています。

    `MainDialog`を更新する限り、ユーザーの入力に関係なく、会話の開始時にユーザーが挨拶されるので、最初の`GreetingAsync`の手順で何かを追加する必要はありません。

1. `MainMenuAsync`では、正規表現を試してみることから始めるため、そのほとんどは残しておきます。ただし、正規表現で意図を見つけられない場合は、`default`アクションを変更します。次に、LUIS を呼び出します。

    `MainMenuAsync`スイッチ ブロック内で、以下を置き換えます。

    ```csharp
    default:
        {
            await MainResponses.ReplyWithConfused(stepContext.Context);
            return await stepContext.EndDialogAsync();
        }
    ```

    以下に置き換えます。

    ```csharp
    default:
    {
        // Call LUIS recognizer
        var result = await _recognizer.RecognizeAsync(stepContext.Context, cancellationToken);
        // Get the top intent from the results
        var topIntent = result?.GetTopScoringIntent();
        // Based on the intent, switch the conversation, similar concept as with Regex above
        switch ((topIntent != null) ? topIntent.Value.intent : null)
        {
            case null:
                // Add app logic when there is no result.
                await MainResponses.ReplyWithConfused(stepContext.Context);
                break;
            case "None":
                await MainResponses.ReplyWithConfused(stepContext.Context);
                // with each statement, we're adding the LuisScore, purely to test, so we know whether LUIS was called or not
                await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
                break;
            case "Greeting":
                await MainResponses.ReplyWithGreeting(stepContext.Context);
                await MainResponses.ReplyWithHelp(stepContext.Context);
                await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
                break;
            case "OrderPic":
                await MainResponses.ReplyWithOrderConfirmation(stepContext.Context);
                await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
                break;
            case "SharePic":
                await MainResponses.ReplyWithShareConfirmation(stepContext.Context);
                await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
                break;
            case "SearchPic":
                await MainResponses.ReplyWithSearchConfirmation(stepContext.Context);
                await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
                break;
            default:
                await MainResponses.ReplyWithConfused(stepContext.Context);
                break;
        }
        return await stepContext.EndDialogAsync();
    }
    ```

1. `MainMenuAsync` メソッド内で、混乱を避けるために空のダイアログのブロックを削除しましょう。次のコードを見つけて削除します。
    ```csharp
    case "search":
        // switch to the search dialog
        return await stepContext.BeginDialogAsync("searchDialog", null, cancellationToken);
    ```

    新しいコードの追加で行っていることを簡単に説明します。まず、we don't understand、と答える代わりに、LUIS を呼び出します。そのため、LUIS 認識エンジンを使用して LUIS を呼び出し、一番上の意図を変数に格納します。次に、`switch`を使用して、ピックアップされた意図に応じて、さまざまな方法で応答します。これは、正規表現で行ったこととほぼ同じです。

    > **注意**: LUIS で、意図に[ラボ 6](../Lab6-Implement_LUIS/02-Implement_LUIS.md) で指示したのとは異なる名前を付けた場合は、それに応じて`case`ステートメントを変更する必要があります。

    もう 1 つの注意点は、LUIS を呼び出したすべての応答の後に、LUIS の意図の値とスコアを追加することです。その理由は、正規表現とは対照的に、LUIS がいつ呼び出されたかだけを示すためです (最終製品からはこれらの応答を削除しますが、ボットをテストする際には良い指標です)。

## ラボ 7.3: 自然な音声フレーズのテスト

1. **F5** を押してアプリを実行します。

1. Bot Emulator に切り替えます。さまざまな方法でイメージを検索してボットに送信してみてください。「send me pictures of water」 (水の写真を送って) や 「show me dog pics」 (犬の写真を見せて) と言うと、どうなりますか? 他の方法で写真を要求したり、共有したり、並べ替えたりしてみてください。

    時間が余った場合は、LUIS で予期しないものがピックアップされているかどうかを確認します。おそらく、この時点が luis.ai に移動し、[エンドポイントの発話を確認](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/label-suggested-utterances)し、モデルを再トレーニング/再公開する良いタイミングです。

    > **楽しい余談**: エンドポイントの発話を確認することは非常に有益です。  LUIS は、どの発話を表面化するかについてスマートに決定します。  人間参加型 (human-in-the-loop) で手動でラベル付けすると、改善するために最大限に役立つものを選びます。  たとえば、LUIS モデルで、特定の発話が 47% の信頼度で Intent1 にマッピングされ、48% の信頼度で Intent2 にマッピングされると予測された場合、このモデルが 2 つの意図の間に非常に近いため、手動でマッピングする人間に対して表面化する有力な候補になります。

## さらに進む

LUIS の実装のカスタマイズで問題が発生した場合は、[ここ](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&tabs=cs)でボットの LUIS への追加に関するドキュメント ガイダンスを参照してください。

>行き詰まってしまったときは? このラボのこの時点までのソリューションは、[code/Finished](./code/Finished) にあります。Azure Bot Service のキーを`appsettings.json`ファイルに挿入する必要があります。このコードは、ソリューションとして実行するのではなく、参照として使用することをお勧めしますが、実行する場合は、必要なキーを必ず追加してください (このセクションでは、必ずしも必要はありません)。

## **追加クレジット**

Azure Cognitive Search を含めて、以前の補足的な LUIS model-with-search [training](https://github.com/Azure/LearnAI-Bootcamp/tree/master/lab01.5-luis) で構築した LUIS ボットの統合を試みる場合は、以下のトレーニングを行ってください。[Azure Cognitive Search](https://github.com/Azure/LearnAI-Bootcamp/tree/master/lab02.1-azure_search), と [Azure Cognitive Search ボット](https://github.com/Azure/LearnAI-Bootcamp/blob/master/lab02.2-building_bots/2_Azure_Search.md).

## 次のステップ

- [ラボ 08-01: 言語を検出する](../Lab8-Detect_Language/01-Introduction.md)
