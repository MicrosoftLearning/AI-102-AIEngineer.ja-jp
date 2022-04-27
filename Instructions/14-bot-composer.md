---
lab:
  title: Bot Framework Composer を使用したボットの作成
  module: Module 7 - Conversational AI and the Azure Bot Service
ms.openlocfilehash: f25609df8d9abc29e691bd83d0470561c9e3e4b0
ms.sourcegitcommit: b934aa694b86756d8b297a384cc6b707f0536e57
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2021
ms.locfileid: "137819342"
---
# <a name="create-a-bot-with-bot-framework-composer"></a>Bot Framework Composer を使用したボットの作成

Bot Framework Composer は、コードを記述せずに高度な会話型ボットをすばやく簡単に構築できるグラフィカル デザイナーです。 Composer は、ボットを構築するためのビジュアル キャンバスを提供するオープンソース ツールです。

## <a name="prepare-to-develop-a-bot"></a>ボットを開発する準備をする

初めに、ボットの開発に必要なサービスとツールを準備します。

### <a name="get-an-openweather-api-key"></a>OpenWeather API キーを取得する

この演習では、Open Weather サービスを使用してユーザーが入力した地域の気象条件を取得するボットを作成します。 サービスを機能させるには、API キーが必要です。

1. Web ブラウザーで、OpenWeather サイト (`https://openweathermap.org/price`) にアクセスします。
2. 無料の API キーをリクエストし、OpenWeather アカウントを作成します (まだ持っていない場合)。
3. サインアップ後、 **[API keys]\(API キー\)** ページを表示して API キーを確認します。

### <a name="update-bot-framework-composer"></a>Bot Framework Composer を更新する

これから、Bot Framework Composer を使用してボットを作成します。 このツールは定期的に更新されるため、最新バージョンがインストールされていることを確認しましょう。

> **注**: 更新には、この演習の手順に影響を与えるユーザー インターフェイスの変更が含まれる場合があります。

1. **Bot Framework Composer** を起動し、更新をインストールするプロンプトが自動的表示されない場合は、 **[ヘルプ]** メニューの **[更新の確認]** オプションを使用して更新を確認します。
2. アップデートがある場合は、アプリケーションを閉じたときにインストールを行うオプションを選択します。 次に、Bot Framework Composer を閉じて、現在ログインしているユーザーのアップデートをインストールし、インストールの完了後に Bot Framework Composer を再起動します。 インストールには数分かかることがあります。
3. Bot Framework Composer のバージョンが **2.0.0** 以降であることを確認します。

## <a name="create-a-bot"></a>ボットの作成

これで、Bot Framework Composer を使用してボットを作成する準備ができました。

### <a name="create-a-bot-and-customize-the-welcome-dialog-flow"></a>ボットを作成し、"welcome" ダイアログ フローをカスタマイズする

1. Bot Framework Composer がまだ開いていない場合は、起動します。
2. **[Home]\(ホーム\)** 画面の **[New]\(新規作成\)** を選択します。 次に、新しい空のボットを作成します。**WeatherBot** という名前を付けて、ローカル フォルダーに保存します。
3. **[作業の開始]** ペインが開いたら閉じ、左側のナビゲーション ウィンドウで、 **[案内]** を選択してオーサリング キャンバスを開き、ユーザーが最初にボットとの会話に参加したときに呼び出される *ConversationUpdate* アクティビティを表示します。 このアクティビティは、アクションのフローで構成されます。
4. 右側のプロパティ ペインで、上部にある **Greeting** という単語を選択し、それを「**WelcomeUsers**」に変更して、**Greeting** というタイトルを編集します。
5. 作成キャンバスで、**[Send a response]\(応答の送信\)** アクションを選択します。 次に、プロパティ ペインで、既定のテキストを *Hi to your bot* から「`Hi! I'm WeatherBot.`」に変更します。
6. オーサリング キャンバスで、最後の **[+]** 記号 (ダイアログ フローの <u>終わり</u>を示す円のすぐ上) を選択し、 **[テキスト]** 応答に新しい **[質問する]** アクションを追加します。

    この新しいアクションにより、ダイアログ フローに 2 つのノードが作成されます。 最初のノードではボットがユーザーに質問する際のプロンプトを定義し、2 番目のノードはユーザーから受け取る応答を表します。 プロパティ ペインでは、これらのノードに対応する **[ボット応答]** タブと **[ユーザー入力]** タブがあります。

7. プロパティ ペインの **[ボット応答]** タブで、「`What's your name?`」というテキストの応答を追加します。 次に、 **[ユーザー入力]** タブで、 **[プロパティ]** の値を「`user.name`」に設定して、ボットの会話内で後でアクセスできる変数を定義します。
8. 作成キャンバスに戻り、先ほど追加した **[User input (Text)]\(ユーザー入力 (テキスト)\)** アクションの下の **+** 記号を選択し、 **[Send a response]\(応答の送信\)** アクションを追加します。
9. 新しく追加した **[Send a response]\(応答の送信\)** アクションを選択し、プロパティ ペインで、テキスト値を「`Hello ${user.name}, nice to meet you!`」に設定します。

    完成したアクティビティ フローはこのようになります。

    ![ユーザーを歓迎し、名前をたずねるダイアログ フロー](./images/welcomeUsers.png)

### <a name="test-the-bot"></a>ボットのテスト

基本的なボットが完成したのでテストしてみましょう。

1. Composer の右上隅にある **[Start Bot]\(ボットの開始\)** を選択し、ボットがコンパイルされて開始されるまで待ちます。 この処理には数分かかることがあります。

    - Windows ファイアウォールのメッセージが表示された場合は、すべてのネットワークのアクセスを有効にします。

2. **[ローカル ボット ランタイム マネージャー]** ペインで、 **[Web チャットを開きます]** を選択します。
3. しばらくすると、**WeatherBot** Web チャット ペインにウェルカム メッセージと名の入力を求めるメッセージが表示されます。  名を入力し、**Enter** キーを押します。
4. ボットは、「**Hello *your_name*, nice to meet you!** 」と応答するはずです。
5. Web チャットパネルを閉じます。
6. Composer の右上の **[&#8635; Restart bot]\(ボットの再起動\)** の隣にある **<u>=</u>** をクリックして **[ローカル ボット ランタイム マネージャー]** ペインを開き、⏹ アイコンを使用してボットを停止します。

## <a name="add-a-dialog-to-get-the-weather"></a>天気を取得するためのダイアログを追加する

作業用のボットが用意できたので、特定の対話のダイアログを追加することで機能を拡張できます。 この例では、ユーザーが "天気" について入力したときにトリガーされるダイアログを追加します。

### <a name="add-a-dialog"></a>ダイアログを追加する

まず、天気に関する質問を処理するために使用されるダイアログ フローを定義する必要があります。

1. 次に示すように、Composer のナビゲーション ウィンドウで最上位ノード (**WeatherBot**) をポイントし、 **[...]** メニューの **[+ Add a dialog]\(ダイアログの追加\)** を選択します。

    ![ダイアログの追加メニュー](./images/add-dialog.png)

    次に、**GetWeather** という名前の新しいダイアログを作成し、「**Get the current weather condition for the provided zip code**」という説明を付けます。
2. ナビゲーション ウィンドウで、新しい **GetWeather** ダイアログの **BeginDialog** ノードを選択します。 次に、オーサリング キャンバスで、 **[+]** 記号を使用して、 **[テキスト]** 応答に新しい **[質問する]** アクションを追加します。
3. プロパティ ペインの **[ボット応答]** タブで、「`Enter your city.`」という応答を追加します。
4. **[ユーザー入力]** タブで、 **[プロパティ]** フィールドを「`dialog.city`」に設定し、 **[出力形式]** フィールドを式「`=trim(this.value)`」に設定して、ユーザーが指定した値の周囲の余分なスペースを削除します。

    これまでのアクティビティ フローはこのようになります。

    ![「応答の送信」アクションが 1 つあるダイアログ フロー](./images/getWeather-dialog-1.png)

    ここまでで、ダイアログはユーザーに地域を入力するように求めました。 次に、入力された地域の天気情報を取得するロジックを実装する必要があります。

6. オーサリング キャンバスで、ユーザーの地域エントリの **[ユーザー入力]** アクションのすぐ下で、 **[+]** 記号を選択して新しいアクションを追加します。
7. アクションの一覧で、**[Access external resources]\(外部リソースへのアクセス\)**、**[Send an HTTP request]\(HTTP 要求の送信\)** の順に選択します。
8. **HTTP 要求** のプロパティを次のように設定し、**YOUR_API_KEY** を [OpenWeather](https://openweathermap.org/price) API キーに置き換えます
    - **HTTP メソッド**: GET
    - **Url**: `http://api.openweathermap.org/data/2.5/weather?units=metric&q=${dialog.city}&appid=YOUR_API_KEY`
    - **結果プロパティ**: `dialog.api_response`

    結果には、HTTP 応答の次の 4 つのプロパティのいずれかを含めることができます。

    - **statusCode**: **dialog.api_response.statusCode** を介してアクセスします。
    - **reasonPhrase**: **dialog.api_response.reasonPhrase** を介してアクセスします。
    - **content**: **dialog.api_response.content** を介してアクセスします。
    - **headers**: **dialog.api_response.headers** を介してアクセスします。

    さらに、応答の種類が JSON の場合は、**dialog.api_response.content** プロパティを介して使用できる逆シリアル化されたオブジェクトになります。 OpenWeather API とそれが返す応答の詳細については、[OpenWeather API のドキュメント](https://openweathermap.org/current)を参照してください。

    次に、応答を処理するロジックをダイアログ フローに追加する必要があります。これにより、HTTP 要求が成功か失敗かを示します。

9. オーサリング キャンバスで、作成した **[Send HTTP Request]\(HTTP 要求の送信アクション\)** の下に、 **[Create a condition]\(条件の作成\)**  >  **[Branch: if/else]\(分岐: if/else\)** アクションを追加します。 このアクションでは、**True** パスと **False** パスを使用してダイアログ フローの分岐を定義します。
10. 分岐アクションの **[Properties]\(プロパティ\)** で、**[Condition]\(条件\)** フィールドを次の式に設定します。

    ```
    =dialog.api_response.statusCode == 200
    ```

11. 呼び出しが成功した場合、応答を変数に格納する必要があります。 オーサリング キャンバスの **True** 分岐で、 **[プロパティの管理]**  >  **[プロパティの設定]** アクションを追加します。 プロパティ ウィンドウで、次のプロパティの割り当てを追加します。

    | プロパティ | 値 |
    | -- | -- |
    | `dialog.weather` | `=dialog.api_response.content.weather[0].description` |
    | `dialog.temp` | `=round(dialog.api_response.content.main.temp)` |
    | `dialog.icon` | `=dialog.api_response.content.weather[0].icon` |

12. まだ **True** 分岐で、 **[プロパティの設定]** アクションの下に **[Send a response]\(応答の送信\)** アクションを追加し、そのテキストを次のように設定します。

    ```
    The weather in ${dialog.city} is ${dialog.weather} and the temperature is ${dialog.temp}&deg;.
    ```

    ***注**: このメッセージは、前のアクションで設定した **dialog.city**、**dialog.weather**、**dialog.temp** プロパティを使用します。 後で、**dialog.icon** プロパティも使用します。*

13. 200 以外の気象サービスからの応答も考慮する必要があるため、**False** 分岐で、 **[Send a response]\(応答の送信\)** を追加し、そのテキストを「`I got an error: ${dialog.api_response.content.message}.`」に設定します

    ダイアログ フローは次のようなります。

    ![HTTP 応答結果のブランチを含むダイアログ フロー](./images/getWeather-dialog-2.png)

### <a name="add-a-trigger-for-the-dialog"></a>ダイアログのトリガーを追加する

次に、既存の welcome ダイアログから新しいダイアログを開始するためのなんらかの方法が必要です。

1. ナビゲーション ウィンドウで、**WelcomeUsers** を含む **WeatherBot** ダイアログを選択します (これは同じ名前のトップレベルのボット ノードの下にあります)。

    ![選択された WeatherBot ワークフロー](./images/select-workflow.png)

2. 選択した **WeatherBot** ダイアログのプロパティ ペインの **[言語理解]** セクションで、 **[認識エンジンの種類]** を **[Regular expression recognizer]\(正規表現認識エンジン\)** に設定します。

    > 既定の認識タイプは、Language Understanding サービスを使用して、自然言語理解モデルを使用してユーザーの意図を生成します。 この演習を簡略化するために、正規表現認識機能を使用しています。 実際のアプリケーションでは、Language Understanding を使用して、より高度な意図認識を可能にすることを検討する必要があります。

3. **WeatherBot** の **[...]** メニューで、 **[新しいトリガーの追加]** を選択します。

    ![トリガーの追加メニュー](./images/add-trigger.png)

    次の設定を使用してトリガーを作成します。

    - **[What is the type of this trigger?]\(このトリガーの種類を選択してください\)**: Intent recognized\(意図認識\)
    - **[What is the name of this trigger (RegEx)]\(このトリガー (RegEx) の名前を指定してください\)** : `WeatherRequested`
    - **[Please input regex pattern]\(regex パターンを入力してください\)** : `weather`

    > 入力されたテキストを正規表現パターンのテキストボックスは、ボットが着信メッセージで *weather* という単語を検索するようにする単純な正規表現パターンです。  "weather" が存在する場合、メッセージは **認識された意図** になり、トリガーが開始されます。

4. トリガーが作成されたら、そのアクションを構成する必要があります。 トリガーのオーサリング キャンバスで、新しい **WeatherRequested** トリガー ノードの下にある **[+]** 記号を選択します。 次に、アクションの一覧で、**[Dialog Management]\(ダイアログ管理\)** を選択し、**[Begin a new dialog]\(新しいダイアログの開始\)** を選択します。
5. **[Begin a new dialog]\(新しいダイアログの開始\)** アクションを選択した状態で、プロパティ ペインで、 **[Dialog name]\(ダイアログ名\)** ドロップダウン リストから **GetWeather** ダイアログを選択し、**WeatherRequested** トリガーが認識されたときに前に定義した **GetWeather** ダイアログを開始します。

    **WeatherRequested** アクティビティ フローは次のようになります。

    ![regex トリガーによる getWeather ダイアログの開始](./images/weather-regex.png)

6. ボットを再起動して Web チャット ペインを開きます。次に、会話を再開し、名前を入力した後、「`What is the weather like?`」と入力します。 プロンプトが表示されたら、「`Seattle`」などの地域を入力します。 ボットがサービスに問い合わせ、天気予報を示す簡単な文で応答します。
7. テストが完了したら、Web チャット ウィンドウを閉じ、ボットを停止します。

## <a name="handle-interruptions"></a>割り込みの処理

適切に設計されたボットは、ユーザーが、たとえば要求をキャンセルすることによって、会話の流れを変更できるようにする必要があります。

1. Bot Composer のナビゲーション ウィンドウで、**WeatherBot** ダイアログの **[...]** メニューを使用します (既存の **WelcomeUsers** および **WeatherRequested** トリガーに加えて) 新しいトリガーを追加します。 新しいトリガーには、次の設定が必要です。

    - **[What is the type of this trigger?]\(このトリガーの種類を選択してください\)**: Intent recognized\(意図認識\)
    - **[What is the name of this trigger (RegEx)]\(このトリガー (RegEx) の名前を指定してください\)** : `CancelRequest`
    - **[Please input regex pattern]\(regex パターンを入力してください\)** : `cancel`

    > テキストに入力されたテキスト ボックスは単純な正規表現パターンであり、ボットは着信メッセージで *cancel* という単語を検索します。

2. トリガーのオーサリング キャンバスで、 **[Send a response]\(応答の送信\)** アクションを追加し、そのテキスト応答を「`OK. Whenever you're ready, you can ask me about the weather.`」に設定します。
3. **[Send a response]\(応答の送信\)** アクションで、 **[Dialog management]\(ダイアログの管理\)** と **[End this dialog]\(このダイアログの終了\)** を選択して、ダイアログに新しいアクションを追加します。

    **CancelRequest** ダイアログ フローは次のようになります。

    ![応答の送信とこのダイアログアクションの終了を伴う CancelRequest トリガー](./images/cancel-dialog.png)

    ユーザーのキャンセル要求に応答するトリガーができたので、天気情報を要求した後に郵便番号の入力を求められた場合など、ユーザーがそのような要求を行う可能性のあるダイアログ フローの中断を許可する必要があります。

4. ナビゲーション ウィンドウで、**GetWeather** ダイアログの下の **BeginDialog** を選択します。
5. ユーザーに地域の入力を求める **[Prompt for text]\(テキストの入力を求める\)** アクションを選択します。
6. アクションのプロパティの **[その他]** タブで、 **[Prompt Configurations]\(プロンプト構成\)** を展開し、 **[Allow Interruptions]\(中断を許可\)** プロパティを **true** に設定します。
7. ボットを再起動し、Web チャット ペインを開きます。 会話を再開し、名前を入力した後、「`What is the weather like?`」と入力します。 次に、プロンプトが表示されたら、「`cancel`」と入力し、リクエストがキャンセルされたことを確認します。
8. リクエストをキャンセルした後、「`What's the weather like?`」と入力し、適切なトリガーによって **GetWeather** ダイアログの新しいインスタンスが開始され、郵便番号の入力を再度求められることに注意してください。
9. テストが完了したら、Web チャット ウィンドウを閉じ、ボットを停止します。

## <a name="enhance-the-user-experience"></a>ユーザー エクスペリエンスを強化する

これまでの気象ボットとのやり取りは、テキストを介して行われました。  ユーザーが意図を表すテキストを入力すると、ボットがテキストで応答します。 多くの場合、テキストはコミュニケーションに適した方法ですが、他の形式のユーザー インターフェイス要素を使用してエクスペリエンスを向上させることができます。  たとえば、ボタンを使用して推奨されるアクションを開始したり、*カード* を表示して情報を視覚的に表示したりできます。

### <a name="add-a-button"></a>ボタンを追加する

1. Bot Framework Composer のナビゲーション ウィンドウの **GetWeather** アクションで、**BeginDialog** を選択します。
2. 作成キャンバスで、地域のプロンプトを含む **[Prompt for text]\(テキストの入力を求める\)** アクションを選択します。
3. プロパティペインで、 **[Show code]\(コードを表示する\)** を選択し、既存のコードを次のコードに置き換えます。

```
[Activity    
    Text = Enter your city.
    SuggestedActions = Cancel
]
```

このアクティビティでは、以前と同じようにユーザーに地域の入力を求めますが、 **[キャンセル]** ボタンも表示します。

### <a name="add-a-card"></a>カードを追加する

1. **GetWeather** ダイアログの HTTP 天気サービスからの応答を確認した後の **True** パスで、天気レポートを表示する **[Send a response]\(応答の送信\)** アクションを選択します。
2. プロパティ ペインで、 **[Show code]\(コードを表示する\)** を選択し、既存のコードを次のコードに置き換えます。

```
[ThumbnailCard
    title = Weather for ${dialog.city}
    text = ${dialog.weather} (${dialog.temp}&deg;)
    image = http://openweathermap.org/img/w/${dialog.icon}.png
]
```

このテンプレートでは、天気について以前と同じ変数を使用しますが、表示されるカードにタイトルを追加し、天気の画像も追加します。

### <a name="test-the-new-user-interface"></a>新しいユーザー インターフェイスをテストする

1. ボットを再起動し、Web チャット ペインを開きます。 会話を再開し、名前を入力した後、「`What is the weather like?`」と入力します。 次に、プロンプトが表示されたら、 **[キャンセル]** ボタンをクリックしてリクエストをキャンセルします。
2. キャンセル後、「`Tell me about the weather`」と入力し、プロンプトが表示されたら、「`London`」などの有効な地域名を入力します。 ボットはサービスに連絡し、気象条件を示すカードで応答する必要があります。
3. テストが完了したら、エミュレーターを閉じ、ボットを停止します。

## <a name="more-information"></a>詳細情報

Bot Framework Composer の詳細については、[Bot Framework Composer のドキュメント](https://docs.microsoft.com/composer/introduction)を参照してください。
