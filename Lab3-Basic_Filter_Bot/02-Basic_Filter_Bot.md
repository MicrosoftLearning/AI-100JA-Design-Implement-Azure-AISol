# ラボ 3: 基本的なフィルタリング ボットを作成する

## 紹介

新たに登場する技術はどれも、それがもたらす機会と同じぐらい多くの疑問点が伴います。そして、AI を活用する技術には、固有の考慮事項があります。
AI ツールを設計して実装するときは、次の AI 倫理の原則を常に念頭に置いてください。

1. *公平性*: 効率を最大化するために尊厳を傷つけてはならない
1. *説明責任*: AI はそのアルゴリズムに責任を持つ必要がある
1. *透明性*: 偏見や、人間の尊厳の破壊を許してはならない
1. *倫理的な応用*: AI は人類を支援するものであり、インテリジェントにプライバシーを守る設計でなければならない

インテリジェントなアプリを構築するときの倫理面に関する考慮事項について、[詳しい説明](https://ai-ethics.azurewebsites.net/)を読むことをお勧めします。

## 前提条件

1. [Lab1-Technical_Requirements.md](../Lab1-Technical_Requirements/02-Technical_Requirements.md) に記載されている手順に従って v4 Bot Framework Emulator をダウンロードします。これは、ボットをローカルでテストできるようにするためです。

## ラボ 3.0 Azure Webアプリボットの作成

Microsoft Bot Framework を使用して作成されたボットは、パブリック アクセス可能な任意の URL でホストできます。  このラボの目的のために、[Azure Bot Service](https://docs.microsoft.com/ja-jp/bot-framework/bot-service-overview-introduction) を使用してボットを登録します。

1. [Azure portal](https://portal.azure.com) に移動します

1. ポータルでリソースグループに移動し、「**+追加**」をクリックして**bot**を検索します。

1. 「**Web App Bot**」を選択して「**作成**」をクリックします。

1. ボットハンドル用に、一意の識別子を作成する必要があります。PictureBot[i][n] のような規則にすることをお勧めします。[i] は自分のイニシャルで、[n] は数字です (例: PictureBotamt6)。

1. リージョンを選択します。

1. 価格レベルの場合は、**F0 (10K Premium メッセージ)** を選択します。

1. ボット テンプレート領域の選択

    1. **C＃**の**Echo Botを**選択します。後でPictureBotに更新します。

    ![ボット テンプレート領域が強調表示され、言語とボットの種類が選択されます。](../images/lab02-createbot.png 'Select the bot type')

1. 「**OK**」をクリックして、「**Echo Bot**」が表示されていることを確認します。

1. 新しい App Service プランを構成します (ボットと同じ場所に配置する)。

1. Application Insights をオンにし、ボットと同じ場所を指定します。

1. 「**アプリ ID とパスワードの自動作成**」は変更もクリックも**しないでください**。これは後で設定します。

1. 「**作成**」を選択します

1. デプロイされたら、新しい Azure Web アプリボットリソースに移動します。

1. 左側のメニューの「**設定**」配下の、「**Configuration**」をクリックします

1. 「**Microsoft App ID**」の「**管理**」リンクをクリックします

![「管理」リンクを選択します](../images/ManageBot.png)

1. 「**新しいクライアント シークレット**」をクリックします

1. 名前に「**PictureBot**」と入力します

1. 有効期限については、「**なし**」を選択します

1. 「**追加**」を選択します

1. シークレットをメモ帳などに記録し、後でラボで使用できるようにします。

1. 「**概要**」をクリックし、**アプリケーション ID** をメモ帳などに記録します。

1. 「**Webアプリボット**」リソースに戻り、「**設定**」の下の「**Webチャットでテスト**」タブを選択します

1. 起動したら、何ができるかを調べます。  ご覧のとおり、メッセージをエコー バックするだけです。

![基本的なエコー ボット応答](../images/EchoBot.png)

## ラボ 3.1: 単純なボットを作成して実行する

1. **Visual Studio 2019**を開きます

1. 「**新しいプロジェクトの作成**」をクリックし、**ボット**を検索します。

1. **Echo Bot (Bot Framework v4)** が表示されるまで下にスクロールします。

    >注意:
    インストールされている Visual Studio のバージョンによっては、以下のスクリーンショットは独自のものと異なる場合があります。  Echo Bot テンプレートに複数のバージョンが表示されている場合は、バージョン 2.1 ではなくバージョン 3.1 を選択します。

    ![エコー ボット プロジェクト テンプレートを選択します。](../images/NewBotProject.png)

1. 「**次へ**」を選択します

    > **注意:** エコー ボット テンプレートが表示されない場合は、req 前の手順から Visual Studio アドインをインストールする必要があります。

1. 名前に「**PictureBot**」と入力し、「**作成**」をクリックします

1. Echo Bot テンプレートから生成されるさまざまなものをすべて見てみましょう。個々のファイルについて説明することはしませんが、**後で**少し時間を取ってこのサンプル (ともう 1 つの Web App Bot サンプル - Basic Bot) を実際に試して精査することを**強くお勧めします** (まだの場合)。この中には、ボット開発に必要となる、重要で有用なシェルがあります。これとその他の有用なシェルやサンプルは、[ここ](https://github.com/Microsoft/BotBuilder-Samples)にあります。

1. まず、ソリューション名を右クリックして「**ビルド**」を選択します。これにより、Nuget パッケージが復元されます。

1. **appsettings.json** ファイルを開き、上記で記録したボット サービス情報を追加して更新します。

    ```json
    {
        "MicrosoftAppId": "YOURAPPID",
        "MicrosoftAppPassword": "YOURAPPSECRET"
    }
    ```

1. **Bots / EchoBot.cs** ファイルを右クリックし、[**名前の変更**]を選択して、クラス ファイルの名前を **PictureBot.cs** に変更します。

    Visual Studio のソリューション/プロジェクトの名前を変更することは、きわめて注意を要する作業です。**慎重に**タスクを行って、すべての名前に EchoBot の代わりに PictureBot が反映される状態にしてください。

    プロンプトが表示されない場合は、クラスの名前を手動で変更してから、クラスのすべての参照を **PictureBot** に変更する必要があります。  プロジェクトをビルドしようとすると、見逃したかどうかがわかります。

1. Startup.csの36行目付近の行を以下のように書き換えます。
    ```
    services.AddTransient<IBot, PictureBot.Bots.PictureBot>();
    ```

1. プロジェクトを右クリックして、「 **Nuget Packagaesの管理(Manage Nuget Packages)** 」を選択します

1. 「**参照(Browse)**」タブをクリックし、次のパッケージをインストールします。

    * Microsoft.Bot.Builder.Azure.Blobs
    * Microsoft.Bot.Builder.Dialogs
    * Microsoft.Bot.Builder.AI.Luis
    * Microsoft.Bot.Builder.Integration.AspNet.Core
    * Azure.AI.TextAnalytics

=======

1. ソリューションをビルドします。

    >**ヒント**:  モニターが 1 台だけの場合に、手順の説明と Visual Studio を簡単に切り替えられるように、手順ファイルを Visual Studio ソリューションに追加できます。それには、ソリューション エクスプローラーでプロジェクトを右クリックし、**「追加」 > 「既存の項目」** を選択します。「Lab3」に移動し、種類が 「MD ファイル」 のすべてのファイルを追加します。

### Hello World ボットを作成する

ラボの今後の部分で使用する命名と NuGet パッケージをサポートするように基本シェルの更新が完了したので、カスタム コードの追加を開始する準備ができました。最初に、V4 SDK を使用するボット開発のウォームアップとしてシンプルな 「Hello world」 ボットを作成します。

重要な概念の 1 つが `"ターン"` です。これは、ユーザーへのメッセージとボットからの応答を表すのに使用されます。
たとえば、ユーザーが「ハロー、ボット」と言ってボットが「こんにちは、お元気ですか?」と答えると、これが **1 つの**ターンとなります。次の画像で、1 つの**ターン**がボット アプリケーションの複数のレイヤーをどのように通過するかを確認してください。

![ボットの概念](https://github.com/MicrosoftLearning/AI-100-Design-Implement-Azure-AISol/raw/master/images/bots-concepts-middleware.png)

1. **PictureBot.cs** ファイルを開きます。  

1. `OnMessageActivityAsync` メソッドを確認します。このメソッドは、会話のたびに呼び出されます。このことが重要である理由は後で分かりますが、ここでは OnMessageActivityAsync がすべてのターンで呼び出されることを覚えておいてください。

1. ツールバーで **IIS Express** が選択されている場合には、**EchoBot** を選択してください。

1. **F5**を押してデバッグを開始します。

    次のことに**注意してください**。

    * default.htm (wwwroot の下) ページがブラウザーに表示されます。

    * Web ページのローカルホスト ポート番号に注目してください。これは、Emulator のエンドポイントと一致しているはずです (一致する必要があります)。

    >行き詰まってしまったときは? このラボのこの時点までのソリューションは、[{GitHubPath}/Lab3-Basic_Filter_Bot/code/Finished/PictureBot/](./code/Finished/PictureBot) にあります。このソリューション内の readme ファイルを開くと、ソリューションを実行するためにどのキーの追加が必要かがわかります。

#### Bot Framework Emulator を使用する

ボットと対話するには:

* Bot Framework Emulator を起動します (このラボでは v4 Emulator を使用します)。  
    * 「**スタート**」をクリックし、「**Bot Emulator**」を検索します。

* ようこそページで、「**Create a new bot configuration**」を選択します

* 名前に「**PictureBot**」と入力します

* Endpoint url に、先ほどのWeb ページに表示されている URL をコピーして、**https://<your_bots_hostname>/api/messages** という形式で入力します。

* appsettings.json に入力した AppId と App Secret を入力します

    > **注**: ボット設定にidとsecretの値を入力しない場合は、ボットエミュレーターに値を入力する必要もありません。

* 「**Save and connect**」をクリックし、.bot ファイルをローカルに保存します。

* この時点で、ボットと会話できる状態になっています。

* **Hello** と入力します。ボットは、先ほど作成した Azure ボットに似たメッセージをエコーして応答します。

    > **注** 「Restart Conversation」を選択して、会話履歴をクリアできます。

    ![Bot Emulator](../images/botemulator3.png)

    ログの中に、次のような行があるかもしれません。

    ![ngrok](../images/ngrok.png)

    これは、ngrok がローカル アドレスに対してバイパスされることを示しています。このワークショップでは ngrok を使用しませんが、仮に一般公開版のボットに接続するとしたら、"production" エンドポイントを介して接続します。この "production" エンドポイントを開き、各環境でのボットの違いを観察してください。これは、開発版ボットをテストして本番ボットと比較するときに役立つ機能です。

    Emulator の使い方の詳細については、[ここ](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0)を参照してください。

1. サンプルのボット コードのファイルの内容を調べます。特に、

    * **Startup.cs** は、サービス/ミドルウェアを追加し、HTTP 要求パイプラインを構成するのに使用するファイルです。この中に多数存在するコメントは、何が行われるのかを理解するのに役立ちます。少し時間を取って読んでみてください。

    * **PictureBot.cs**: `OnMessageActivityAsync` メソッドは、ユーザーからのメッセージを待つエントリ ポイントです。ここで、受信したメッセージに反応して、さらにメッセージを待つことができます。  `turnContext.SendActivityAsync` を使用して、ボットからユーザーにメッセージを返すことができます。

## ラボ 3.2:  状態とサービスの管理

1. デバッグを止め、**Startup.cs**ファイルに移動します

1. `using` ステートメントに、以下を**追加**します。

    ```csharp
    using System;
    using System.Linq;
    using System.Text.RegularExpressions;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Options;
    using Microsoft.Extensions.Logging;

    using Microsoft.Bot.Builder.AI.Luis;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Azure.Blobs;
    ```

    上記の全ての名前空間はまだ使用しませんが、いつ使うかを考えてみてください。

1. **Startup.cs** クラスで、`ConfigureServices` メソッドに注目してください。これは、ボットにサービスを追加するのに使用されます。内容を精査して、何が自動的にビルドされるかを見つけてください。

    > より深く理解するための追加の注意事項:
    >
    > * 依存関係挿入については、[こちらの詳しい説明](https://docs.microsoft.com/ja-jp/aspnet/web-api/overview/advanced/dependency-injection)を参考にしてください。
    > * このラボとテストの目的では、ローカル メモリを使用できます。本番運用には、[状態データを管理する](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-storage-concept?view=azure-bot-service-4.0)方法を実装する必要があります。`ConfigureServices` の中のまとまったコメント行に、このヒントがあります。
    > * このメソッドの一番下で、状態アクセサーを作成して登録していることに気付いたでしょうか。状態の管理は、インテリジェントなボットを作成するための鍵です。[詳しくは、こちらを参照してください](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-dialog-state?view=azure-bot-service-4.0)。

    幸い、このシェルは非常に包括的なので、2つの項目を追加するだけです。

    * ミドルウェア
    * カスタム状態アクセサー

### ミドルウェア

ミドルウェアは、単一のクラスまたはクラスの集合で、アダプターとボット ロジックの間に存在し、アダプターのミドルウェアのコレクションに初期化時に追加されます。

SDK を使用して、開発者は独自のミドルウェアをプログラミングすることや、他者が作成したミドルウェアの再利用可能コンポーネントを追加することができます。ボットに出入りするすべてのアクティビティは、ミドルウェアを通って流れます。これについては、このラボで後ほど詳しく取り上げますが、現時点で理解しておいてほしいのは、どのアクティビティもミドルウェアを通って流れるということです。その理由は、ミドルウェアが `ConfigureServices` メソッドの中に存在し、このメソッドが実行時に呼び出されるからです (ユーザーと `OnMessageActivityAsync` によって送信されるすべてのメッセージの間で実行されます)。

1. プロジェクトに**Middleware**という名前の新しいフォルダを追加します

1. 「**Middleware**」フォルダーを右クリックして、**「追加」>「既存の項目」** を選択します。

1. **{GitHubDir}\Lab3-Basic_Filter_Bot\code\Middleware** に移動し、3つのファイルをすべて選択して、「**追加**」 を選択します

1. **Startup** クラスに次の変数を追加します。

    ```csharp
    private ILoggerFactory _loggerFactory;
    ```

1. **ConfigureServices** メソッドの次のコードを置き換えます。

    ```csharp
    services.AddTransient<IBot, PictureBot>();
    ```

    以下のコードで置き換えます。

    ```csharp
    services.AddBot<PictureBot.Bots.PictureBot>(options =>
    {
        var appId = Configuration.GetSection("MicrosoftAppId")?.Value;
                    var appSecret = Configuration.GetSection("MicrosoftAppPassword")?.Value;

                    options.CredentialProvider = new SimpleCredentialProvider(appId, appSecret);

        // Creates a logger for the application to use.
        ILogger logger = _loggerFactory.CreateLogger<PictureBot.Bots.PictureBot>();

        // Catches any errors that occur during a conversation turn and logs them.
        options.OnTurnError = async (context, exception) =>
        {
            logger.LogError($"Exception caught : {exception}");
            await context.SendActivityAsync("Sorry, it looks like something went wrong.");
        };


        var middleware = options.Middleware;
        // Add middleware below with "middleware.Add(...."
        // Add Regex below
    });
    ```

1. **Configure** メソッドを次のコードに置き換えます。

    ```csharp
    public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
    {
        _loggerFactory = loggerFactory;

        app.UseDefaultFiles()
                    .UseBotFramework()
                    .UseStaticFiles()
                    .UseWebSockets()
                    .UseRouting()
                    .UseAuthorization()
                    .UseEndpoints(endpoints =>
                    {
                        endpoints.MapControllers();
                    });
    }
    ```

#### カスタム状態アクセサー

必要なカスタム状態アクセサーについて説明する前に、背景情報が重要です。ダイアログ (次のセクションで説明します) は、マルチターン会話ロジックを実装するためのアプローチの 1 つです。この場合は、ユーザーが現在会話のどこにいるかを知るために、永続化された状態に依存する必要があります。このラボで作成するダイアログ ベースのボットでは、DialogSet を使用してさまざまなダイアログを保持します。この DialogSet を作成するには、"アクセサー" と呼ばれるオブジェクトへのハンドルを使用します。

アクセサーは、SDK の中にある `IStatePropertyAccessor` インターフェイスを実装します。これによって、状態に関する情報の取得、設定、削除ができるようになります。したがって、開発者はユーザーが会話の中のどのステップにいるかを追跡することができます。

1. 作成するアクセサーのそれぞれについて、最初にプロパティ名を指定する必要があります。このラボでは、次のものを追跡します。

    * `PictureState`
        * 既にユーザーに挨拶したか?
            * 挨拶を 2 回以上する必要はありませんが、会話の最初に確実に挨拶する必要があります。
        * ユーザーは現在特定の単語を検索しているか? もしそうなら、それは何か?
            * ユーザーが何を探しているかをこちらに伝えてくれたかどうか、伝えてくれた場合はそれが何なのかを、追跡する必要があります。
    * `DialogState`
        * ユーザーは現在ダイアログの途中にいるか?
            * これは、ユーザーがあるダイアログまたは会話フローの中のどこにいるかを特定するのに使用します。ダイアログの知識がなくても、心配しないでください。この後すぐに説明します。

        これらのコンストラクトを使用して、`PictureState`と呼ぶものを追跡できます。

1. **Startup.cs** ファイルの **ConfigureServices** メソッドで、カスタム状態アクセサーのリスト内に `PictureState` を追加し、ダイアログを追跡するために、組み込みの `DialogState` を使用します。

    ```csharp
    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<PictureBotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        if (options == null)
        {
            throw new InvalidOperationException("BotFrameworkOptions must be configured prior to setting up the state accessors");
        }

        var conversationState = sp.GetRequiredService<ConversationState>();
        //var conversationState = services.BuildServiceProvider().GetService<ConversationState>();

        if (conversationState == null)
        {
            throw new InvalidOperationException("ConversationState must be defined and added before adding conversation-scoped state accessors.");
        }

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        return new PictureBotAccessors(conversationState)
        {
            PictureState = conversationState.CreateProperty<PictureState>(PictureBotAccessors.PictureStateName),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>("DialogState"),
        };

    });
    ```

1. いくつかのアイテムの下にエラー (赤い波線) があります。しかし、これらを修正する前に疑問に思うかもしれません。なぜアクセサーを 2 つ作成する必要があったのか? なぜ 1 つでは不十分だったのか?

    * `DialogState` は `Microsoft.Bot.Builder.Dialogs` ライブラリからのアクセサーです。メッセージが送信されると、Dialog サブシステムが `CreateContext` を `DialogSet` に対して呼び出します。このコンテキストを追跡するには、`DialogState` アクセサーが必要です。これは、適切なダイアログ状態 JSON を取得するためのアクセサーです。
    * 一方、`PictureState` は、指定された特定の会話プロパティを会話全体で追跡するために使用されます (たとえば、既にユーザーに挨拶したかどうか)。

    > ダイアログに関する専門用語を知らなくても、このプロセスは理解できるはずです。よく理解できなかった場合は、[状態のしくみの詳しい説明](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-dialog-state?view=azure-bot-service-4.0)を参照してください。

1. では、先ほど見ていたエラーに戻りましょう。この情報を保存する必要がありますが、どこに、どのように保存するかはまだ指定していません。情報を保存してアクセスするには、"PictureState.cs" と "PictureBotAccessor.cs" を更新する必要があります。

1. プロジェクトを右クリックして **「追加」-> 「クラス」** を選択し、クラスファイルを選択して**PictureState**という名前を付けます。

1. 次のコードを **PictureState.cs** にコピーします。

    ```csharp
    using System.Collections.Generic;

    namespace Microsoft.PictureBot
    {
        /// <summary>
        /// Stores counter state for the conversation.
        /// Stored in <see cref="Microsoft.Bot.Builder.ConversationState"/> and
        /// backed by <see cref="Microsoft.Bot.Builder.MemoryStorage"/>.
        /// </summary>
        public class PictureState
        {
            /// <summary>
            /// Gets or sets the number of turns in the conversation.
            /// </summary>
            /// <value>The number of turns in the conversation.</value>
            public string Greeted { get; set; } = "not greeted";
            public string Search { get; set; } = "";
            public string Searching { get; set; } = "no";
        }
    }
    ```

1. コードを確認します。  ここに、アクティブな会話に関する情報を格納します。  文字列の目的を説明するコメントを自由に追加してください。これで、PictureState が適切に初期化されたので、"**Startup.cs**" で発生していたエラーを解消するように PictureBotAccessor を作成できます。

1. プロジェクトを右クリックして **「追加」-> 「クラス」** を選択し、クラスファイルを選択して**PictureBotAccessors**という名前を付けます。

1. 次の項目をコピーします。

    ```csharp
    using System;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;

    namespace Microsoft.PictureBot
    {
        /// <summary>
        /// This class is created as a Singleton and passed into the IBot-derived constructor.
        ///  - See <see cref="PictureBot"/> constructor for how that is injected.
        ///  - See the Startup.cs file for more details on creating the Singleton that gets
        ///    injected into the constructor.
        /// </summary>
        public class PictureBotAccessors
        {
            /// <summary>
            /// Initializes a new instance of the <see cref="PictureBotAccessors"/> class.
            /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
            /// </summary>
            /// <param name="conversationState">The state object that stores the counter.</param>
            public PictureBotAccessors(ConversationState conversationState)
            {
                ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
            }

            /// <summary>
            /// Gets the <see cref="IStatePropertyAccessor{T}"/> name used for the <see cref="CounterState"/> accessor.
            /// </summary>
            /// <remarks>Accessors require a unique name.</remarks>
            /// <value>The accessor name for the counter accessor.</value>
            public static string PictureStateName { get; } = $"{nameof(PictureBotAccessors)}.PictureState";

            /// <summary>
            /// Gets or sets the <see cref="IStatePropertyAccessor{T}"/> for CounterState.
            /// </summary>
            /// <value>
            /// The accessor stores the turn count for the conversation.
            /// </value>
            public IStatePropertyAccessor<PictureState> PictureState { get; set; }

            /// <summary>
            /// Gets the <see cref="ConversationState"/> object for the conversation.
            /// </summary>
            /// <value>The <see cref="ConversationState"/> object.</value>
            public ConversationState ConversationState { get; }

            /// <summary> Gets the IStatePropertyAccessor{T} name used for the DialogState accessor. </summary>
            public static string DialogStateName { get; } = $"{nameof(PictureBotAccessors)}.DialogState";

            /// <summary> Gets or sets the IStatePropertyAccessor{T} for DialogState. </summary>
            public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
        }
    }
    ```

1. コードを確認し、`PictureStateName`と`PictureState`の実装に注目してください。

1. Staratup.csに以下のusing句を追加します。

```
using Microsoft.PictureBot;
```

1. これを正しく構成したかどうかが心配ですか? 「**Startup.cs**」 に戻り、カスタム状態アクセサーの作成に関するエラーが解決されていることを確認してください。

    > すべてのエラーが消えたわけではありません。次の手順で解決していきます。

## ラボ 3.3: ボットのコードを整理する

ボットを開発する方法はさまざまなものがあり、好みもそれぞれ異なります。この SDK では、開発者の好みの方法でコードを整理することができます。このラボでは、会話を整理してさまざまなダイアログにまとめます。また、会話関連のコードを [MVVM スタイル](https://msdn.microsoft.com/ja-jp/library/hh848246.aspx)で整理する方法も考察します。

この PictureBot は、次のように整理します。

* **Dialogs** - モデルを編集するためのビジネス ロジック
* **Responses** - ユーザーへの出力を定義するクラス
* **Models** - 変更対象のオブジェクト

1. プロジェクト内に 2 つの新しいフォルダー 「**Responses**」 と 「**Models**」 を作成するには、プロジェクトを右クリックし、**「追加」 > 「新しいフォルダー」** を選択します。

### ダイアログ

ダイアログとそのしくみについては、既にご存じかもしれません。そうでない場合は、続行する前に[こちらのダイアログに関するページ](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-dialog-manage-conversation-flow?view=azure-bot-service-4.0&tabs=csharp)をお読みください。

1 つのボットで複数のタスクを実行できる場合は、複数のダイアログを持てるようにすると、ユーザーがさまざまな会話フローをたどることができるようになります。このラボの PictureBot では、ユーザーが最初のメニュー フロー (メイン ダイアログと呼ばれることもよくあります) を通過したら、複数のダイアログに分岐します。この分岐は、ユーザーがしようとしていることが写真の検索、写真の共有、写真の注文、ヘルプ参照のどれであるかに基づきます。これを簡単に行うには、ダイアログ コンテナーを使用します。このラボでは `DialogSet` を使用します。続行する前に、[モジュール型のボット ロジックと複雑なダイアログの作成に関する記事](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-compositcontrol?view=azure-bot-service-4.0&tabs=csharp)をお読みください。

このラボの目的では、処理をごく単純なものにしますが、修了すると多数のダイアログから成るダイアログ セットを作成できるようになります。このラボの PictureBot には、次の 2 つの主要ダイアログがあります。

* **MainDialog** - ボット開始時の既定のダイアログ。このダイアログから、ユーザーが要求した他のダイアログが起動します。このダイアログはダイアログ セットのメイン ダイアログであるため、このダイアログでダイアログ コンテナーを作成し、必要に応じてユーザーを他のダイアログにリダイレクトします。

* **SearchDialog** - 検索要求を処理して結果をユーザーに返すことを管理するダイアログ。  ***注**: このワークショップでは、この機能を呼び出しますが、検索は実装しません。*

ダイアログは 2 つしかないので、単純に両方とも PictureBot クラスに入れておくことができます。ただし、複雑なシナリオでは、これらを同じフォルダー内の別々のダイアログに分けることが必要になる可能性があります ("応答" と "モデル" を分けたのと同じように)。

1. **PictureBot.cs** に戻り、一連の `using` ステートメントを次のコードで置き換えます。

    ```csharp
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Schema;
    using Microsoft.PictureBot;
    using PictureBot.Responses;
    ```

これで、Models/Responsesへのアクセスと、サービス LUIS と Azure Cognitive Search へのアクセスが追加されました。最後の Newtonsoft への参照は、LUIS からの応答を解析するのに役立ちます。これについては、後続のラボで学習します。

次に、`OnTurnAsync` のメソッドを置き換える必要があります。具体的には、受け取ったメッセージを処理してからさまざまなダイアログを通してそのメッセージをルーティングするメソッドで置き換えます。

1. **PictureBot** クラスを次のコードで置き換えます。

    ```csharp
    /// <summary>
    /// Represents a bot that processes incoming activities.
    /// For each user interaction, an instance of this class is created and the OnTurnAsync method is called.
    /// This is a Transient lifetime service.  Transient lifetime services are created
    /// each time they're requested. For each Activity received, a new instance of this
    /// class is created. Objects that are expensive to construct, or have a lifetime
    /// beyond the single turn, should be carefully managed.
    /// For example, the <see cref="MemoryStorage"/> object and associated
    /// <see cref="IStatePropertyAccessor{T}"/> object are created with a singleton lifetime.
    /// </summary>
    /// <seealso cref="https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-2.1"/>
    /// <summary>Contains the set of dialogs and prompts for the picture bot.</summary>
    public class PictureBot : ActivityHandler
    {
        private readonly PictureBotAccessors _accessors;
        // Initialize LUIS Recognizer

        private DialogSet _dialogs;

        /// <summary>
        /// Every conversation turn for our PictureBot will call this method.
        /// There are no dialogs used, since it's "single turn" processing, meaning a single
        /// request and response. Later, when we add Dialogs, we'll have to navigate through this method.
        /// </summary>
        /// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
        /// for processing this conversation turn. </param>
        /// <param name="cancellationToken">(Optional) A <see cref="CancellationToken"/> that can be used by other objects
        /// or threads to receive notice of cancellation.</param>
        /// <returns>A <see cref="Task"/> that represents the work queued to execute.</returns>
        /// <seealso cref="BotStateSet"/>
        /// <seealso cref="ConversationState"/>
        /// <seealso cref="IMiddleware"/>
        public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
        {
            if (turnContext.Activity.Type is "message")
            {
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
            }
        }

        public PictureBot(PictureBotAccessors accessors)
        {
            _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

            // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
            _dialogs = new DialogSet(_accessors.DialogStateAccessor);

            // This array defines how the Waterfall will execute.
            // We can define the different dialogs and their steps here
            // allowing for overlap as needed. In this case, it's fairly simple
            // but in more complex scenarios, you may want to separate out the different
            // dialogs into different files.
            var main_waterfallsteps = new WaterfallStep[]
            {
                GreetingAsync,
                MainMenuAsync,
            };
            var search_waterfallsteps = new WaterfallStep[]
            {
                // Add SearchDialog water fall steps

            };

            // Add named dialogs to the DialogSet. These names are saved in the dialog state.
            _dialogs.Add(new WaterfallDialog("mainDialog", main_waterfallsteps));
            _dialogs.Add(new WaterfallDialog("searchDialog", search_waterfallsteps));
            // The following line allows us to use a prompt within the dialogs
            _dialogs.Add(new TextPrompt("searchPrompt"));
        }

        // Add MainDialog-related tasks

        // Add SearchDialog-related tasks

        // Add search related tasks

    }
    ```

少し時間を取ってこのシェルを精査し、ワークショップの他の参加者とディスカッションしてください。続行する前に各行の目的を理解する必要があります。

これには、後ほどさらにコードを追加します。エラーは無視してかまいません (今のところは)。

#### 応答

ダイアログの内容を作成する前に、応答を準備しておく必要があります。既に説明したように、ダイアログと応答を分けます。このようにするとコードがすっきりするのと、ダイアログのロジックを追うのが簡単になるのが理由です。今は同意できなくても、すぐにできるようになります。

1. **Responses** フォルダーで、 **MainResponses.cs** および **SearchResponses.cs** という2つのクラスを作成します。既にお分かりかもしれませんが、Responses のファイルの内容はユーザーに送信するさまざまな出力だけであり、ロジックは含まれていません。

1. **MainResponses.cs** で、コードを次のように置き換えます。

    ```csharp
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;

    namespace PictureBot.Responses
    {
        public class MainResponses
        {
            public static async Task ReplyWithGreeting(ITurnContext context)
            {
                await context.SendActivityAsync("Hello, Im a Picture Bot");
            }
            public static async Task ReplyWithHelp(ITurnContext context)
            {
                await context.SendActivityAsync($"I can search for pictures, share pictures and order prints of pictures.");
            }
            public static async Task ReplyWithResumeTopic(ITurnContext context)
            {
                await context.SendActivityAsync($"What can I do for you?");
            }
            public static async Task ReplyWithConfused(ITurnContext context)
            {
                // Add a response for the user if Regex or LUIS doesn't know
                // What the user is trying to communicate
                await context.SendActivityAsync($"I'm sorry, I don't understand.");
            }
            public static async Task ReplyWithLuisScore(ITurnContext context, string key, double score)
            {
                await context.SendActivityAsync($"Intent: {key} ({score}).");
            }
            public static async Task ReplyWithShareConfirmation(ITurnContext context)
            {
                await context.SendActivityAsync($"Posting your picture(s) on twitter...");
            }
            public static async Task ReplyWithOrderConfirmation(ITurnContext context)
            {
                await context.SendActivityAsync($"Ordering standard prints of your picture(s)...");
            }
            public static async Task ReplyWithSearchConfirmation(ITurnContext context)
            {
                await context.SendActivityAsync($"Searching picture(s)...");
            }
        }
    }
    ```

値のない応答が 2 つあることに注目してください (ReplyWithGreeting と ReplyWithConfused)。適切だと思う値を入力してしてください。

1. 「SearchResponses.cs」内で、コードを次のように置き換えます。

    ```csharp
    using Microsoft.Bot.Builder;
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading.Tasks;
    using Microsoft.Bot.Schema;

    namespace PictureBot.Responses
    {
        public class SearchResponses
        {
            // add a task called "ReplyWithSearchRequest"
            // it should take in the context and ask the
            // user what they want to search for
            public static async Task ReplyWithSearchConfirmation(ITurnContext context, string utterance)
            {
                await context.SendActivityAsync($"Ok, searching for pictures of {utterance}");
            }
            public static async Task ReplyWithNoResults(ITurnContext context, string utterance)
            {
                await context.SendActivityAsync("There were no results found for \"" + utterance + "\".");
            }
        }
    }
    ```

1. タスク全体が欠落していることに注意してください。適切だと思う内容を自分で入力してください。ただし、新しいタスクの名前は 「ReplyWithSearchRequest」 としてください。このとおりでない場合は、後で問題が発生する可能性があります。

#### モデル

時間に制限があるため、すべてのモデルの作成の説明はしません。これらのモデルは単純ですが、追加後に少し時間を取ってコードを精査することをお勧めします。 

1. 「**Models**」 フォルダーを右クリックして、**「追加」>「既存の項目」** を選択します。

1. **{GitHubDir}\Lab3-Basic_Filter_Bot\code\Models** に移動し、3つのファイルをすべて選択して、「**追加**」 を選択します

## ラボ 3.4: 正規表現とミドルウェア

ボットをさらに良いものにするためにできることは多数あります。何よりも、LUIS を単純な "写真を検索" メッセージ (ボットがかなり頻繁にユーザーから受け取る要求です) に使用したくはありません。  単純な正規表現でこれを行うことができ、時間の節約になり (ネットワークの待機時間)、費用も節約できます (LUIS サービスを呼び出すコスト)。

また、ボットの複雑さが増し、ユーザーの入力を受け取って複数のサービスを使用して解釈するようになると、そのフローを管理するプロセスが必要になります。  たとえば、最初に正規表現を試してみて、それで見つけられない場合は LUIS を呼び出し、その後は他のサービス、たとえば [QnA Maker](http://qnamaker.ai) や Azure Cognitive Search を試します。これを管理するのに良い方法の 1 つは[ミドルウェア](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0)を使用することであり、SDK はこれをサポートします。

ラボを続行する前に、ミドルウェアと Bot Framework SDK についてさらに学習してください。

    * [概要とアーキテクチャ](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-basics?view=azure-bot-service-4.0)

    * [ミドルウェア](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0)

    * [ミドルウェアの作成](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-create-middleware?view=azure-bot-service-4.0&tabs=csaddmiddleware%2Ccsetagoverwrite%2Ccsmiddlewareshortcircuit%2Ccsfallback%2Ccsactivityhandler)

最終的には、ミドルウェアを使用して、ユーザーが言っていることの理解を試みます。最初に正規表現 (Regex) を使用し、理解できない場合は LUIS を呼び出します。それでも理解できない場合は、「おっしゃっていることの意味がわかりません」という応答、またはその他の開発者が 「ReplyWithConfused」 に対して指定したものを返します。

1. 「**Startup.cs**」 の、`ConfigureServices` の中にある「// Add Regex below」というコメントの下に、次の行を追加します。

```csharp
middleware.Add(new RegExpRecognizerMiddleware()
.AddIntent("search", new Regex("search picture(?:s)*(.*)|search pic(?:s)*(.*)", RegexOptions.IgnoreCase))
.AddIntent("share", new Regex("share picture(?:s)*(.*)|share pic(?:s)*(.*)", RegexOptions.IgnoreCase))
.AddIntent("order", new Regex("order picture(?:s)*(.*)|order print(?:s)*(.*)|order pic(?:s)*(.*)", RegexOptions.IgnoreCase))
.AddIntent("help", new Regex("help(.*)", RegexOptions.IgnoreCase)));
```

    > ここでは、正規表現の使い方のごく一部を示しています。興味がある場合は、[こちらの詳しい情報を参照してください](https://docs.microsoft.com/ja-jp/dotnet/standard/base-types/regular-expression-language-quick-reference)。

1. 次のコードを`ConfigureServices`に追加します。

    ```csharp
    // Create the User state.
    services.AddSingleton<UserState>(sp => {
        var dataStore = sp.GetRequiredService<IStorage>();
            return new UserState(dataStore);
    });

    // Create the Conversation state.
    services.AddSingleton<ConversationState>(sp =>
    {
        var dataStore = sp.GetRequiredService<IStorage>();
        return new ConversationState(dataStore);
    });

    // Create the IStorage.
    services.AddSingleton<IStorage, MemoryStorage>(sp =>
    {
        return new MemoryStorage();
    });
    ```

1. LUIS を追加していないので、このボットはいくつかのバリエーションを理解するだけですが、ユーザーがこのボットを使って写真を検索し、共有し、プリントを注文するときに、かなりのメッセージを理解するはずです。

    > 余談: ボットができることについてのオプションを並べたメニューを受け取るためにユーザーが「help」と入力する必要はないと主張する人もいるかもしれませんが、これはボットと最初に接触したときの既定の動作です。**見つけやすさ**はボットにとって最大の課題の 1 つです。このボットに何ができるかをユーザーに知ってもらう必要があります。 優れた[ボット設計の原則](https://docs.microsoft.com/ja-jp/bot-framework/bot-design-principles)が役立ちます。

## ラボ 3.5: ボットを実行する

### MainDialog 再び

本題に戻りましょう。ボットがユーザーの言いたいことに反応できるように、PictureBot.cs　内で MainDialog に記入する必要があります。 正規表現からの結果に基づいて、正しい方向に会話を指示する必要があります。コードを注意深く読んで、コードが何をしているのかを理解することを確認してください。

1. **PictureBot.cs** で、次のコードを貼り付けて追加します。

    ```csharp
    // If we haven't greeted a user yet, we want to do that first, but for the rest of the
    // conversation we want to remember that we've already greeted them.
    private async Task<DialogTurnResult> GreetingAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // Get the state for the current step in the conversation
        var state = await _accessors.PictureState.GetAsync(stepContext.Context, () => new PictureState());

        // If we haven't greeted the user
        if (state.Greeted == "not greeted")
        {
            // Greet the user
            await MainResponses.ReplyWithGreeting(stepContext.Context);
            // Update the GreetedState to greeted
            state.Greeted = "greeted";
            // Save the new greeted state into the conversation state
            // This is to ensure in future turns we do not greet the user again
            await _accessors.ConversationState.SaveChangesAsync(stepContext.Context);
            // Ask the user what they want to do next
            await MainResponses.ReplyWithHelp(stepContext.Context);
            // Since we aren't explicitly prompting the user in this step, we'll end the dialog
            // When the user replies, since state is maintained, the else clause will move them
            // to the next waterfall step
            return await stepContext.EndDialogAsync();
        }
        else // We've already greeted the user
        {
            // Move to the next waterfall step, which is MainMenuAsync
            return await stepContext.NextAsync();
        }

    }

    // This step routes the user to different dialogs
    // In this case, there's only one other dialog, so it is more simple,
    // but in more complex scenarios you can go off to other dialogs in a similar
    public async Task<DialogTurnResult> MainMenuAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // Check if we are currently processing a user's search
        var state = await _accessors.PictureState.GetAsync(stepContext.Context);

        // If Regex picks up on anything, store it
        var recognizedIntents = stepContext.Context.TurnState.Get<IRecognizedIntents>();
        // Based on the recognized intent, direct the conversation
        switch (recognizedIntents.TopIntent?.Name)
        {
            case "search":
                // switch to the search dialog
                return await stepContext.BeginDialogAsync("searchDialog", null, cancellationToken);
            case "share":
                // respond that you're sharing the photo
                await MainResponses.ReplyWithShareConfirmation(stepContext.Context);
                return await stepContext.EndDialogAsync();
            case "order":
                // respond that you're ordering
                await MainResponses.ReplyWithOrderConfirmation(stepContext.Context);
                return await stepContext.EndDialogAsync();
            case "help":
                // show help
                await MainResponses.ReplyWithHelp(stepContext.Context);
                return await stepContext.EndDialogAsync();
            default:
                {
                    await MainResponses.ReplyWithConfused(stepContext.Context);
                    return await stepContext.EndDialogAsync();
                }
        }
    }
    ```

1. **F5** を押してボットを実行します。

1. ボット エミュレーターを使用して、いくつかのコマンドを送信してボットをテストします。

    * help
    * share pics
    * order pics
    * search pics
    
    > **注** ボットで500エラーが発生した場合、**OnTurnError** デリゲート メソッド内の **Startup.cs** ファイルにブレークポイントを配置できます。  最も一般的なエラーは、AppId と AppSecret の不一致です。

1. 期待どおりの結果を得られなかったのが 「search pics」 だけの場合は、すべては自分で構成したとおりに動作しています。「search pics」 の失敗は、ラボのこの時点での予期される動作ですが、理由は分かりますか? 次に進む前に答えを考えてください。

    >ヒント: ブレーク ポイントを使用して、case 「search」 への一致を、**PictureBot.cs** からトレースしてください。
    >行き詰まってしまったときは? このラボのこの時点までのソリューションは、[resources/code/Finished](./code/Finished) にあります。このソリューション内の readme ファイルを開くと、ソリューションを実行するためにどのキーの追加が必要かがわかります。このコードは、ソリューションとして実行するのではなく、参照として使用することをお勧めしますが、実行する場合は、環境に必要なキーを必ず追加してください。

## リソース

* [Bot Builder Basics](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-basics?view=azure-bot-service-4.0&tabs=cs)
* [.NET ボット ビルダー SDK チュートリアル](https://docs.microsoft.com/ja-jp/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0)
* [ボット サービスのドキュメント](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0)
* [ボットを展開する](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=newrg)

## 次のステップ

* [ラボ 04-01: チャットをログに記録する](../Lab4-Log_Chat/01-Introduction.md)
