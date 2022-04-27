---
lab:
  title: Speech および Language Understanding サービスの使用
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: 31b88612bce38780f828859f8e2b166dbe8df34d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819351"
---
# <a name="use-the-speech-and-language-understanding-services"></a>Speech および Language Understanding サービスの使用

音声サービスを Language Understanding サービスと統合して、音声入力からユーザーの意図をインテリジェントに判断できるアプリケーションを作成できます。

> **注**: この演習は、マイクがあると適切に行うことができます。 一部のホストされる仮想環境では、ローカル マイクから音声をキャプチャできる場合があります。しかし、これが機能しない場合 (または、マイクがない場合)、音声入力用に付属の音声ファイルを使用できます。 マイクまたは音声ファイルを使用するかどうかに応じて、ことなるオプションを選択する必要があるろきは、慎重に手順に従ってください。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボの作業をしている環境に既にクローンしている場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐクローンしてください。

1. Visual Studio Code を起動します。
2. パレットを開き (Shift + Ctrl + P)、**Git: Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-language-understanding-resources"></a>Language Understanding リソースの作成

Azure サブスクリプションに Language Understanding オーサリングおよび予測リソースが既にある場合は、この演習でそれらを使用できます。 それ以外の場合は、次の手順に従って作成してください。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択して、「*language understanding*」を検索し、次の設定を使用して **Language Understanding** リソースを作成します。
    - **[Create option]\(作成オプション\)**: 両方
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[名前]**: *一意の名前を入力します*
    - **[作成場所]**: *希望の場所を選択します*
    - **[価格レベルを作成しています]**: F0
    - **[予測の場所]**: *作成場所と <u>同じ場所</u>を選択します*
    - **予測価格レベル**: F0 (*F0 が使用できない場合は、S0 を選択*)

3. リソースが作成されるまで待ちます。2 つの Language Understanding リソースがプロビジョニングされていることに注意してください。1 つは作成用、もう 1 つは予測用です。 作成先のリソース グループに移動すると、この両方を表示できます。

## <a name="prepare-a-language-understanding-app"></a>Language Understanding アプリを準備する

前の演習の **時計** アプリを既にお持ちの場合は、`https://www.luis.ai` の Language Understanding ポータルで開きます。 それ以外の場合は、次の手順に従って作成してください。

1. ブラウザーの新しいタブで、`https://www.luis.ai` の Language Understanding ポータルを開きます。
2. Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。 初めて Language Understanding ポータルにサインインする場合は、アカウントの詳細にアクセスするために、アプリケーションにいくつかのアクセス許可を付与する必要が生じることがあります。 次に、Azure サブスクリプションと作成したオーサリング リソースを選択して、*ようこそ* の手順を完了します。
3. **[会話アプリ]** ページを開き、 **[新しいアプリ]** の横にあるドロップダウン リストを表示して、 **[LU としてインポート]** を選択します。
この演習のラボ ファイルを含むプロジェクト フォルダー内の **11-luis-speech** サブフォルダーを参照し、**Clock.lu** を選択します。 次に、時計アプリの一意の名前を指定します。
4. 効果的な Language Understanding アプリを作成するためのヒントが表示されたパネルが表示された場合、そのパネルを閉じます。

## <a name="train-and-publish-the-app-with-speech-priming"></a>*音声プライミング* でアプリをトレーニングして公開する

1. アプリがまだトレーニングされていない場合は、Language Understanding ポータルの上部にある **[トレーニング]** を選択して、アプリをトレーニングします。
2. Language Understanding ポータルの右上にある **[公開]** を選択します。 次に、 **[本番スロット]** を選択し、設定を変更して **音声プライミング** を有効にします (これにより、音声認識のパフォーマンスが向上します)。
3. 公開が完了したら、Language Understanding ポータルの上部にある **[管理]** を選択します。
4. **[設定]** ページで、**アプリ ID** をメモします。 クライアント アプリケーションがアプリを使用するには、これが必要です。
5. **[Azure リソース]** ページの **[予測リソース]** で、予測リソースがリストされていない場合は、Azure サブスクリプションに予測リソースを追加します。
6. 予測リソースの **プライマリ キー**、**セカンダリ キー**、および **場所** (エンドポイントでは<u>ありません</u>) に注意してください。 Speech SDK クライアント アプリケーションは、予測リソースに接続して認証されるために、場所とキーの 1 つを必要とします。

## <a name="configure-a-client-application-for-language-understanding"></a>Language Understanding 用クライアント アプリケーションを構成する

この演習では、音声入力を受け入れ、Language Understanding アプリを使用して、ユーザーに意図を予測するクライアント アプリケーションを作成します。

> **注**: この演習では、**C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **エクスプローラー** ペインで、**11-luis-speech** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **speaking-clock-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Language Understanding アプリの **アプリ ID** を含めます。**場所** (完全なエンドポイントでは <u>ありません</u> - たとえば、*eastus*) とその予測リソースの **キー** の 1 つ (Language Understanding ポータルのアプリの **[管理]** ページから)。

## <a name="install-sdk-packages"></a>SDK パッケージをインストールする

Speech SDK を Language Understanding と共に使用するには、使用するプログラム言語用の Speech SDK を印刷インストールする必要があります。

1. Visual Studio で、**speaking-clock-client** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Language Understanding SDK パッケージをインストールします

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

2. また、システムに動作するマイクが<u>ない</u>場合は、アプリケーションに音声入力を提供するために、音声ファイルを使用する必要があります。 この場合は、次のコマンドを使用して、プログラムが音声ファイルを再生できるように、次のパッケージをインストールします (マイクを使用する場合は、これをスキップできます)。

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

3. **speaking-clock-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#** : Program.cs
    - **Python**: speaking-clock-client.py

4. コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Speech SDK を使用するために必要な名前空間インポートします。

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Intent;
    ```

    **Python**

    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. また、システムに動作するマイクが<u>ない</u>場合、既存の名前空間の下に、次のコードをインポートして追加して、音声ファイルを再生するために使用するライブラリをインポートします。

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

## <a name="create-an-intentrecognizer"></a>*IntentRecognizer* を作成する

**IntentRecognizer** クラスは、音声入力から Language Understanding 予測を取得するために使用できるクライアント オブジェクトを提供します。

1. **Main** 関数では、構成ファイルからアプリ ID、予測リージョン、およびキーを読み込むためのコードが既に提供されていることに注意してください。 次にコメント「**Configure speech service and get intent recognizer**」を見つけ、音声入力にマイクまたは音声ファイルを使用するかに応じて、次のコードを追加します。

    ### <a name="if-you-have-a-working-microphone"></a>**機能するマイクがある場合:**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

    ### <a name="if-you-need-to-use-an-audio-file"></a>**音声ファイルを使用する場合:**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    string audioFile = "time-in-london.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    System.Threading.Thread.Sleep(2000);
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    audioFile = 'time-in-london.wav'
    playsound(audioFile)
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

## <a name="get-a-predicted-intent-from-spoken-input"></a>音声入力から予測意図を取得する

音声入力から予測意図を取得するコードを実装する準備が整いました。

1. **Main** 関数の追加したコードのすぐ下に、コメント **「Get the model from the AppID and add the intents we want to use」** を見つけ、使用する意図を追加し、次のコードを追加して、(アプリ ID に基づいて) Language Understanding モデルを取得し、認識機能に識別させたい意図を指定します。

    **C#**

    ```C#
    // Get the model from the AppID and add the intents we want to use
    var model = LanguageUnderstandingModel.FromAppId(luAppId);
    recognizer.AddIntent(model, "GetTime", "time");
    recognizer.AddIntent(model, "GetDate", "date");
    recognizer.AddIntent(model, "GetDay", "day");
    recognizer.AddIntent(model, "None", "none");
    ```

    *各意図に文字列ベースの ID を指定できることに注意してください*

    **Python**

    ```Python
    # Get the model from the AppID and add the intents we want to use
    model = speech_sdk.intent.LanguageUnderstandingModel(app_id=lu_app_id)
    intents = [
        (model, "GetTime"),
        (model, "GetDate"),
        (model, "GetDay"),
        (model, "None")
    ]
    recognizer.add_intents(intents)
    ```

2. このループ内で、コメント **「Process speech input」** を見つけ、次のコードを追加します。このコードは、レコグナイザーを使用して、音声入力を使用して Language Understanding サービスを非同期的に呼び出し、応答を取得します。 応答に予測された意図が含まれている場合、音声クエリ、予測された意図、および完全な JSON 応答が表示されます。 それ以外の場合、コードは返された理由に基づいて応答を処理します。

**C#**

```C
// Process speech input
string intent = "";
var result = await recognizer.RecognizeOnceAsync().ConfigureAwait(false);
if (result.Reason == ResultReason.RecognizedIntent)
{
    // Intent was identified
    intent = result.IntentId;
    Console.WriteLine($"Query: {result.Text}");
    Console.WriteLine($"Intent Id: {intent}.");
    string jsonResponse = result.Properties.GetProperty(PropertyId.LanguageUnderstandingServiceResponse_JsonResult);
    Console.WriteLine($"JSON Response:\n{jsonResponse}\n");
    
    // Get the first entity (if any)

    // Apply the appropriate action
    
}
else if (result.Reason == ResultReason.RecognizedSpeech)
{
    // Speech was recognized, but no intent was identified.
    intent = result.Text;
    Console.Write($"I don't know what {intent} means.");
}
else if (result.Reason == ResultReason.NoMatch)
{
    // Speech wasn't recognized
    Console.WriteLine($"Sorry. I didn't understand that.");
}
else if (result.Reason == ResultReason.Canceled)
{
    // Something went wrong
    var cancellation = CancellationDetails.FromResult(result);
    Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");
    if (cancellation.Reason == CancellationReason.Error)
    {
        Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
        Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
    }
}
```

**Python**

```Python
# Process speech input
intent = ''
result = recognizer.recognize_once_async().get()
if result.reason == speech_sdk.ResultReason.RecognizedIntent:
    intent = result.intent_id
    print("Query: {}".format(result.text))
    print("Intent: {}".format(intent))
    json_response = json.loads(result.intent_json)
    print("JSON Response:\n{}\n".format(json.dumps(json_response, indent=2)))
    
    # Get the first entity (if any)
    
    # Apply the appropriate action
    
elif result.reason == speech_sdk.ResultReason.RecognizedSpeech:
    # Speech was recognized, but no intent was identified.
    intent = result.text
    print("I don't know what {} means.".format(intent))
elif result.reason == speech_sdk.ResultReason.NoMatch:
    # Speech wasn't recognized
    print("Sorry. I didn't understand that.")
elif result.reason == speech_sdk.ResultReason.Canceled:
    # Something went wrong
    print("Intent recognition canceled: {}".format(result.cancellation_details.reason))
    if result.cancellation_details.reason == speech_sdk.CancellationReason.Error:
        print("Error details: {}".format(result.cancellation_details.error_details))
```

これまでに追加したコードは *意図* を識別しますが、一部の意図は *エンティティ* を参照できるため、サービスから返される JSON からエンティティ情報を抽出するコードを追加する必要があります。

3. 追加したコードで、コメント **「Get the first entity (if any)」** を見つけ、その下に次のコードを追加します。

**C#**

```C
// Get the first entity (if any)
JObject jsonResults = JObject.Parse(jsonResponse);
string entityType = "";
string entityValue = "";
if (jsonResults["entities"].HasValues)
{
    JArray entities = new JArray(jsonResults["entities"][0]);
    entityType = entities[0]["type"].ToString();
    entityValue = entities[0]["entity"].ToString();
    Console.WriteLine(entityType + ": " + entityValue);
}
```

**Python**

```Python
# Get the first entity (if any)
entity_type = ''
entity_value = ''
if len(json_response["entities"]) > 0:
    entity_type = json_response["entities"][0]["type"]
    entity_value = json_response["entities"][0]["entity"]
    print(entity_type + ': ' + entity_value)
```
    
コードは、言語理解アプリを使用して、入力発話で検出されたエンティティだけでなく、インテントも予測するようになりました。 クライアント アプリケーションは、その予測を使用して適切なアクションを決定および実行する必要があります。

4. 追加したコードの下に、 **「Apply the appropriate action」** というコメントを見つけ、次のコードを追加します。このコードは、アプリケーションでサポートされている意図 (**GetTime**、**GetDate**、および **GetDay**) をチェックし、適切な応答を生成するために既存の関数を呼び出す前に、関連するエンティティが検出されたかどうかを判断します。

**C#**

```C#
// Apply the appropriate action
switch (intent)
{
    case "time":
        var location = "local";
        // Check for entities
        if (entityType == "Location")
        {
            location = entityValue;
        }
        // Get the time for the specified location
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;
    case "day":
        var date = DateTime.Today.ToShortDateString();
        // Check for entities
        if (entityType == "Date")
        {
            date = entityValue;
        }
        // Get the day for the specified date
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;
    case "date":
        var day = DateTime.Today.DayOfWeek.ToString();
        // Check for entities
        if (entityType == "Weekday")
        {
            day = entityValue;
        }

        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;
    default:
        // Some other intent (for example, "None") was predicted
        Console.WriteLine("You said " + result.Text.ToLower());
        if (result.Text.ToLower().Replace(".", "") == "stop")
        {
            intent = result.Text;
        }
        else
        {
            Console.WriteLine("Try asking me for the time, the day, or the date.");
        }
        break;
}
```

**Python**

```Python
# Apply the appropriate action
if intent == 'GetTime':
    location = 'local'
    # Check for entities
    if entity_type == 'Location':
        location = entity_value
    # Get the time for the specified location
    print(GetTime(location))

elif intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if entity_type == 'Date':
        date_string = entity_value
    # Get the day for the specified date
    print(GetDay(date_string))

elif intent == 'GetDate':
    day = 'today'
    # Check for entities
    if entity_type == 'Weekday':
        # List entities are lists
        day = entity_value
    # Get the date for the specified day
    print(GetDate(day))

else:
    # Some other intent (for example, "None") was predicted
    print('You said {}'.format(result.text))
    if result.text.lower().replace('.', '') == 'stop':
        intent = result.text
    else:
        print('Try asking me for the time, the day, or the date.')
```

## <a name="run-the-client-application"></a>クライアント アプリケーションを実行する

1. 変更を保存し、**speaking-clock-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock-client.py
    ```

2. マイクを使用している場合、アプリケーションをテストするために声を出して発言を話します。 たとえば、次のことを試してください (毎回、プログラムを再実行します)。

    *What's the time?*
    
    *今何時?*

    *What day is it?*

    *What is the time in London?*

    *What's the date?*

    *What date is Sunday?*

> **注**: アプリケーションのロジックは意図的に単純であり、いくつかの制限がありますが、Speech SDK を使用して音声入力から意図を予測する Language Understanding モデルの機能をテストする目的に役立つはずです。 *MM/DD/YYYY* 形式で日付を言語化するのが難しいため、特定の日付エンティティで **GetDay** 意図を認識できない場合があります。

## <a name="more-information"></a>詳細情報

音声と Language Understanding の統合の詳細については、[音声のドキュメント](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/intent-recognition)を参照してください。
