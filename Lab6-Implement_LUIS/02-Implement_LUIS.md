# ラボ 6: LUIS モデルの実装

このハンズオン ラボでは、マイクロソフトの Language Understanding Intelligent Service (LUIS) を使用して、アプリケーションの自然言語処理機能を強化したモデルを作成する方法について説明します。

## 紹介

このラボでは、LUIS モデルの構築、トレーニング、および公開を行い、(将来のラボで作成される) ボットが人間のユーザーと効果的に通信できるようにします。

> **注記:** このラボでは、今後のラボで使用する LUIS モデルのみを作成して、よりインテリジェントなボットを構築します。

LUIS の機能については既にワークショップで説明しました。LUIS について復習するには、[詳細をお読みください](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/Home)。

LUIS の概要を把握したところで、LUIS アプリを計画します。既に特定のテキストが含まれるメッセージに応答する基本的なボット ("PictureBot") を作成しています。ボットが実行できるさまざまなアクションをトリガーする意図を作成し、さまざまなアクションに必要なエンティティを作成する必要があります。たとえば、PictureBot の意図は "OrderPic" であり、適切な応答を提供するためにボットをトリガーするとします。

たとえば、「Search」の場合 (ここでは実装されません)、PictureBot の意図が "SearchPics" であり、写真を検索するために Azure Cognitive Search サービスがトリガーされます。これには、検索対象を知るために "facet" エンティティが必要です。  アプリを計画するためのその他の例については、[ここ](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/plan-your-app)を参照してください。

アプリについて考え抜いたら、[それを構築してトレーニングする](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/luis-get-started-create-app)準備が整いました。

復習のために、LUIS アプリケーションの作成時に一般的に実行する手順は次のとおりです。

  1. [意図を追加する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-intents)
  2. [発話を追加する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-example-utterances)
  3. [エンティティを追加する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-entities)
  4. [フレーズ リスト](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/add-features)と[パターンを使用してパフォーマンスを向上させる](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/luis-how-to-model-intent-pattern)
  5. [トレーニングとテストを行う](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/train-test)
  6. [エンドポイントの発話を確認する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/label-suggested-utterances)
  7. [公開する](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/publishapp)

## ラボ 6.0: Portal での LUIS サービスの作成 (オプション)

LUIS ではラボで使用できる "スターター キー" が提供されるため、Portal での LUIS サービスの作成はオプションです。ただし、Portal で無料または有料のサービスを作成する方法を確認する場合は、以下の手順に従って実行できます。  

> **注**;  pre-req ARMテンプレートを実行した場合、Language Understanding API を含む Cognitive Services リソースがすでにあります。

1. [Azure Portal](https://portal.azure.com) を開く

1. 「**+ リソースの作成**」を選択します。

1. 検索ボックスに**language**と入力して、 **Language Understanding**を選びます

1. 作成オプションで、**両方**を選択します。
    
    > AuthoringリソースとPredictionリソースの両方が作成されます。

1. サブスクリプションとリソースグループを選択します

1. 名前に **{YOURINIT}luisbot** と入力します

1. 作成リソースの作成場所は、**米国西部**を選択します

1. 価格帯については、 **Free F0**を選択します

1. 予測リソースの場所については、リソース グループと同じ場所を選択します。

1. 予測価格レベルは、**Free F0** を選択します。

1. 「確認および作成」をクリックし、「**作成**」を選択します

## ラボ 6.1: LUIS を使用してアプリケーションにインテリジェンスを追加する

LUIS を使用して、自然言語機能を追加する方法を見てみましょう。LUIS を使用すると、自然言語の発話 (ボットと話すときにユーザーが話す単語/フレーズ/文) を意図 (ユーザーが実行するタスクやアクション) にマッピングできます。ここでは、写真の検索、写真の共有、写真のプリントの順序付けなど、いくつかの意図があります。これらの事柄を尋ねる方法として、発話のいくつかの例をご紹介します。LUIS では、学習した内容に基づいて、それぞれの意図に追加の新しい発話がマッピングされます。

1. [https://www.luis.ai](https://www.luis.ai)に移動します （**ヨーロッパまたはオーストラリアにいない場合**）。ボットをサポートする新しい LUIS アプリを作成します。

    > **注**:**ヨーロッパ** リージョンでキーを作成した場合は、[https://eu.luis.ai/](https://eu.luis.ai/) でアプリケーションを作成する必要があります。**オーストラリア** リージョンでキーを作成した場合は、[https://au.luis.ai/](https://au.luis.ai/) でアプリケーションを作成する必要があります。LUIS を公開しているリージョンの詳細については、[ここ](https://docs.microsoft.com/ja-jp/azure/cognitive-services/luis/luis-reference-regions)を参照してください。

1. **Login** をクリックし、組織のアカウントまたは Microsoft アカウントを使用してサインインします。(これは、前のセクションで LUIS を作成したのと同じアカウントにする必要があります)。

1. **Choose an authoring resource** 画面で、先ほど作成した LUIS の Authoring リソースを選択し、**Done** をクリックします。

1. **Conversation apps** ページで、「**+ New App**」ボタンをクリックします

    > **注意**: [現在のページ](https://www.luis.ai/applications)の「New App」 (新しいアプリ) ボタンの横に「Import as JSON」 (アプリのインポート) もあります。  LUIS アプリケーションを作成した後、アプリ全体を JSON としてエクスポートし、ソース管理にチェックインできます。  これは推奨されるベスト プラクティスであり、コードのバージョン管理に合わせて LUIS モデルのバージョンを管理できます。  エクスポートされた LUIS アプリは、「Import App」 (アプリのインポート) ボタンを使用して再インポートできます。  ラボの進行が遅れてしまい、ショートカットする場合は、「Import App」 (アプリのインポート) ボタンをクリックして [LUIS モデル](./code/LUIS/PictureBotLuisModel.json)をインポートできます。

1. Nameを**PictureBotLuisModel**と入力します

1. Cultureを **Japanese** にします。
 
1. **Prediction resource** を選択します

1. 「**Done**」をクリックします。

1. **How to create an effective LUIS app** が表示されたら閉じてください。

1. 上部のナビゲーションで、「**BUILD**」 リンクを選択します。  「None」 (なし) というIntent（意図）が 1 つあることに注意してください。  いずれの意図にもマッピングされないランダムな発話は、"None" (なし) にマッピングされる場合があります。

    ![LUIS ダッシュボード](../images//LuisCreateIntent.png)

    ボットが次のことを実行できるようにしましょう。

    - 写真を検索する
    - ソーシャル メディアで写真を共有する
    - 写真のプリントを注文する
    - ユーザーに挨拶する (ただし、これは後で説明するように、他の方法でも可能)

    これらのそれぞれを要求するユーザーの意図を作成してみましょう。  

1. Intents 画面で、「**Create**」ボタンをクリックします。

1. Intent name に「**Greeting**」という名前を付け、「**Done**」をクリックします。  

1. ボットに挨拶するときにユーザーが話す可能性のある言葉の例を日本語でいくつ入力し、それぞれの後に Enter キーを押します。

    ![LUIS Greeting の意図](../images//LuisGreetingIntent.png)

    エンティティを作成する方法を見てみましょう。  ユーザーが写真の検索を要求するとき、何を探すかを指定できます。 エンティティ内でキャプチャしてみましょう。

1. 左側の列内の「**Entities**」を選択してから「**+ Create**」を選択します。  

1. エンティティ名 **facet** を付ける

1. エンティティ タイプとして「**Machine learned**」を選択します。  

1. 「**Create**」を選択します。

    ![種類が「Simple」の facet というエンティティ名の追加](../images/select-facet.png)

1. 左側のサイドバーの「**Intents**」をクリックし、「**+ Create**」ボタンをクリックします。  

1. 「**SearchPic**」という意図名を付け、「**Done**」をクリックします。

    「greetings」 で行ったのと同じように、いくつかのサンプル発話 (ボットと話すときにユーザーが話す可能性のある単語/フレーズ/文) を追加してみましょう。  写真を検索する方法は数多くあります。  以下の発話の一部を自由に使用して、ボットに写真を検索するように依頼する方法について独自の文章を追加してください。

    - 屋外の写真を見つける
    - 電車の写真はあるか
    - 食べ物の画像を探して
    - 遊んでいる子供の写真を検索
    - ビーチの写真を見せて
    - 犬の写真
    - 眼鏡をかけている男性の画像

    いくつかの発話を用意したら、**検索トピック**を 「facet」 エンティティとして選択する方法を LUIS に教える必要があります。「facet」 エンティティが選択するものは何でも検索されます。

1. 単語にカーソルを合わせてクリックするか、連続する単語をクリックして単語のグループを選択し、次に 「facet」 エンティティを選択します。

    ![エンティティのラベル付け](../images//LuisFacet.png)

    ファセットにラベルが付けられると、発話がこのようになる可能性があります。

    ![ファセット エンティティを追加する](../images//SearchPicsIntentAfter.png)

    >**注**: このワークショップには Azure Cognitive Search は含まれていませんが、この機能はデモンストレーションのために残されています。

1. 左側のサイドバーの「**Intents**」をクリックして、さらに 2 つの意図を追加します。

    - 1つの意図に「**SharePic**」という名前を付けます。  これは、次のような発話によって識別される場合があります。

        - この写真を共有
        - ツイートできますか?
        - ツイッターに投稿

    - 「**OrderPic**」という名前の別の意図を作成します。  これは、次のような発話で伝達できます。

        - この写真を印刷
        - プリントを注文したい
        - 8x10 が入手可能であるか
        - 注文

    発話を選択するときには、質問、命令、「したい...」 形式の組み合わせを使用すると便利です。

1. 最後に、「None」 という意図にサンプル発話を追加する必要があります。これにより、アプリケーションのスコープ外である場合に LUIS にラベルを付けることができます。「ピザが食べたい、「動画を検索」 などを追加します。「None」 (なし) という意図には、アプリの発話の約 10-15% が必要です。

## ラボ 6.2: LUIS モデルのトレーニング

これで、モデルをトレーニングする準備が整いました。この練習では、モデルをテストするために簡単なトレーニング操作を実行します。  テストは、LUIS ポータルの組み込みテスト パネルを使用して行われます。

1. 一番上のメニューで、「**Train**」を選択します。トレーニング中、LUIS はモデルを構築し、提供されたトレーニング データに基づいて発話をマッピングします。

    > [!TIP]
    > トレーニングはすぐに行われるとは限りません。場合によっては、キューに入れられ、数分かかることがあります。

## LUIS サービスのパブリック エンドポイントを作成する

1. トレーニングが終了したら、上部のツールバーにある「**MANAGE**」を選択します。左側のツールバーに次のオプションが表示されます。

    > [!NOTE]
    > ポータルが更新されると、左側のウィンドウのカテゴリが変更される場合があります。  その結果、キーとエンドポイントは、ここに示されているものとは異なるカテゴリに分類される可能性があります。

    - **Settings**
    - **Publish Settings**
    - **Azure Resources**
    - **Versions**

1. 「**Azure Resources**」を選択します。この画面は、LUIS サービスへのアクセスに使用される URL エンドポイントを管理するために使用されます。

1. 「**Prediction Resources**」タブに「**Primary Key**」が既に作成されていることがわかります。  

  **Prediction Resources**が表示されている場合は、次の「**アプリを公開する**」セクションに進みます。
  
    1. 既存の**Prediction Resources**が表示されない場合は、「**Add Prediction resource**」を選択します。「**Azure directory**」は既に選択されています。
    1. サブスクリプションと、先に Azure Portal で作成したリソースを選択し、「**Done**」を選択してから Language Understanding リソースを LUIS サービスに接続します。

## アプリを公開する

1. トップバーで「**Publish**」をクリックします。

   > [!NOTE]
   > **Production** エンドポイントまたは **Staging** エンドポイントに発行できます。

1. 「**Production Slot**」を選択してから「**Done**」を選択します。

    公開すると、LUIS モデルを呼び出すエンドポイントが作成されます。エンドポイント URL が表示されます。エンドポイント URL をコピーし、今後使用するためにキーのリストに追加します。

1. トップバーで「**Test**」をクリックします。以下のような発話を入力して、意図が返されることを確認してください。これらのタスクの例を次に示します。

   | 発話 | 結果 | スコアの意味 |
   |---------|---------|---------|
   | **地元のビーチの写真を表示して** | スコア付きの **SearchPic** 意図が返されます。 | 高い一致 |
   | **こんにちは** | スコア付きの **Greeting** 意図が返されます。 | ほぼ正しい一致 |
   | **トムに送信する** | 低いスコアが返されます | 再トレーニングが必要か、意図に一致しません |

    低スコアの発話のモデルを再トレーニングするには、次の手順を実行します。

    1. 低スコアの発話 (この場合は「**トムに送信する**」) の横にある「**Inspect**」を選択します。
    1. 「**Top-scoring intent**」の横にある「Assign to a new intent」をクリックし、ドロップダウンリストから「**SharePic**」を選択します。
    1. 「**Test**」パネルを閉じます。
    1. 「**Train**」ボタンを選択して、モデルを再トレーニングします。
    1. **トムに送信する**の発話を再度テストします。高いスコアで **SharePic**意図が返されるようになります。

    これで、クライアント アプリケーションで LUIS アプリを使用したり、一覧に記載されたエンドポイントを介してブラウザーでテストしたり、ボットに統合したりする準備が整いました。


    [公開されたエンドポイントをブラウザーでテストする](https://docs.microsoft.com/ja-jp/azure/cognitive-services/LUIS/PublishApp#test-your-published-endpoint-in-a-browser)こともできます。エンドポイントの URL をコピーします。ブラウザーでこの URL を開くには、URL パラメーター`&q`をテスト クエリに設定します。たとえば、URL に`Find pictures of dogs`を追加し、Enter キーを押します。ブラウザーには、HTTP エンドポイントの JSON 応答が表示されます。

## さらに進む

まだ時間がある場合は、www.luis.ai サイトを探索してください。「Prebuilt Domains」 (事前構築済みドメイン) を選択し、[既に利用できるものを確認してください](https://docs.microsoft.com/ja-jp/azure/cognitive-services/luis/luis-reference-prebuilt-domains)。また、[他の機能](https://docs.microsoft.com/ja-jp/azure/cognitive-services/luis/luis-concept-feature)や[パターン](https://docs.microsoft.com/ja-jp/azure/cognitive-services/luis/luis-concept-patterns)の一部を確認し、
LUIS モデルの作成、LUIS モデルの管理、会話のシミュレーションなどを行うための [BotBuilder](https://github.com/Microsoft/botbuilder-tools) ツールを確認することもできます。後で、[LUIS スキーマの設計方法が含まれる別のコース](https://aka.ms/daaia)に興味を抱くこともあるでしょう。

## 追加クレジット

Azure Cognitive Search が含まれる LUIS モデルの作成を試みる場合は、[検索を含めた LUIS モデル](https://github.com/Azure/LearnAI-Bootcamp/tree/master/lab01.5-luis)のトレーニングに従います。

## 次のステップ

- [ラボ 07-01: 統合 LUIS](../Lab7-Integrate_LUIS/01-Introduction.md)
