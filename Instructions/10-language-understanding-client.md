---
lab:
  title: Language Understanding クライアント アプリケーションの作成
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: 36f75e47910707c959495140be43c4196649d471
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819368"
---
# <a name="create-a-language-understanding-client-application"></a>Language Understanding クライアント アプリケーションの作成

Language Understanding サービスを使用すると、アプリケーションがユーザーからの自然言語入力を解釈し、ユーザーの "*意図*" (達成したいこと) を予測し、意図を適用する必要がある "*エンティティ*" を特定するために使用できる言語モデルをカプセル化するアプリを定義できます。 Language Understanding アプリを使用するクライアント アプリケーションは、REST インターフェイスを介して直接作成するか、言語固有のソフトウェア開発キット (SDK) を使用して作成できます。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボの作業をしている環境に既にクローンしている場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐクローンしてください。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-language-understanding-resources"></a>Language Understanding リソースの作成

Azure サブスクリプションに Language Understanding オーサリングおよび予測リソースが既にある場合は、この演習でそれらを使用できます。 それ以外の場合は、次の手順に従って作成してください。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択して、「*language understanding*」を検索し、次の設定を使用して **Language Understanding** リソースを作成します。
    - **[Create option]\(作成オプション\)**: 両方
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: "*リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)* "
    - **[名前]**: *一意の名前を入力します*
    - **[作成場所]**: *希望の場所を選択します*
    - **[価格レベルを作成しています]**: F0
    - **[予測の場所]**: *作成場所と <u>同じ場所</u>を選択します*
    - **予測価格レベル**: F0 (*F0 が使用できない場合は、S0 を選択*)

3. リソースが作成されるまで待ちます。2 つの Language Understanding リソースがプロビジョニングされていることに注意してください。1 つは作成用、もう 1 つは予測用です。 作成先のリソース グループに移動すると、この両方を表示できます。

## <a name="import-train-and-publish-a-language-understanding-app"></a>Language Understanding アプリをインポート、トレーニング、公開する

前の演習の **Clock** アプリを既にお持ちの場合は、この演習で使用できます。 それ以外の場合は、次の手順に従って作成してください。

1. ブラウザーの新しいタブで、`https://www.luis.ai` の Language Understanding ポータルを開きます。
2. Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。 初めて Language Understanding ポータルにサインインする場合は、アカウントの詳細にアクセスするために、アプリケーションにいくつかのアクセス許可を付与する必要が生じることがあります。 次に、Azure サブスクリプションと作成したオーサリング リソースを選択して、*ようこそ* の手順を完了します。
3. **[会話アプリ]** ページを開き、 **[新しいアプリ]** の横にあるドロップダウン リストを表示して、 **[LU としてインポート]** を選択します。
この演習のラボ ファイルを含むプロジェクト フォルダー内の **10-luis-client** サブフォルダーを参照し、**Clock.lu** を選択します。 次に、時計アプリの一意の名前を指定します。
4. 効果的な Language Understanding アプリを作成するためのヒントが表示されたパネルが表示された場合、そのパネルを閉じます。
5. Language Understanding ポータルの上部にある **[トレーニング]** を選択して、アプリをトレーニングします。
6. Language Understanding ポータルの右上にある **[公開]** を選択し、アプリを **運用スロット** に公開します。
7. 公開が完了したら、Language Understanding ポータルの上部にある **[管理]** を選択します。
8. **[設定]** ページで、**アプリ ID** をメモします。 クライアント アプリケーションがアプリを使用するには、これが必要です。
9. **[Azure リソース]** ページの **[予測リソース]** で、予測リソースが一覧表示されていない場合は、Azure サブスクリプションに予測リソースを追加します。
10. 予測リソースの **主キー**、**2 次キー**、**エンドポイント URL** に注目します。 クライアント アプリケーションは、予測リソースに接続して認証されるために、エンドポイントとキーの 1 つを必要とします。

## <a name="prepare-to-use-the-language-understanding-sdk"></a>Language Understanding SDK を使用する準備をする

この演習では、クロック Language Understanding アプリを使用してユーザー入力から意図を予測し、適切に応答する、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**:**C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**10-luis-client** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **clock-client** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Language Understanding SDK パッケージをインストールします

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
```

"***ランタイム** (予測) パッケージに加えて、Language Understanding モデルを作成および管理するためのコードを記述するために使用できる **オーサリング** パッケージがあります。* "

**Python**

```
pip install azure-cognitiveservices-language-luis==0.7.0
```
    
"*Python SDK パッケージには、**予測** と **オーサリング** の両方のクラスが含まれています。* "

3. **clock-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、Language Understanding アプリの **アプリ ID**、**エンドポイント URL**、その予測リソースの **キー** のうち 1 つを含む構成値を更新します (Language Understanding ポータルのアプリの **[管理]** ページから)。

4. **clock-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: clock-client.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Language Understanding 予測 SDK を使用するために必要な名前空間をインポートします。

**C#**

```C#
// Import namespaces
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
```

**Python**

```Python
# Import namespaces
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
```

## <a name="get-a-prediction-from-the-language-understanding-app"></a>Language Understanding アプリから予測を取得する

これで、SDK を使用して Language Understanding アプリから予測を取得するコードを実装する準備が整いました。

1. **Main** 関数では、構成ファイルからアプリ ID、予測エンドポイント、キーを読み込むためのコードが既に提供されていることにご注意ください。 次に、コメント **Create a client for the LU app** を見つけ、次のコードを追加して、Language Understanding アプリの予測クライアントを作成します。

**C#**

```C#
// Create a client for the LU app
var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.ApiKeyServiceClientCredentials(predictionKey);
var luClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
```

**Python**

```Python
# Create a client for the LU app
credentials = CognitiveServicesCredentials(lu_prediction_key)
lu_client = LUISRuntimeClient(lu_prediction_endpoint, credentials)
```

2. ユーザーが「quit」と入力するまで、**Main** 関数のコードはユーザー入力を求めるプロンプトを表示することにご注意ください。 このループ内で、コメント **Call the LU app to get intent and entities** を見つけて、次のコードを追加します。

**C#**

```C#
// Call the LU app to get intent and entities
var slot = "Production";
var request = new PredictionRequest { Query = userText };
PredictionResponse predictionResponse = await luClient.Prediction.GetSlotPredictionAsync(luAppId, slot, request);
Console.WriteLine(JsonConvert.SerializeObject(predictionResponse, Formatting.Indented));
Console.WriteLine("--------------------\n");
Console.WriteLine(predictionResponse.Query);
var topIntent = predictionResponse.Prediction.TopIntent;
var entities = predictionResponse.Prediction.Entities;
```

**Python**

```Python
# Call the LU app to get intent and entities
request = { "query" : userText }
slot = 'Production'
prediction_response = lu_client.prediction.get_slot_prediction(lu_app_id, slot, request)
top_intent = prediction_response.prediction.top_intent
entities = prediction_response.prediction.entities
print('Top Intent: {}'.format(top_intent))
print('Entities: {}'.format (entities))
print('-----------------\n{}'.format(prediction_response.query))
```

Language Understanding アプリを呼び出すと、予測が返されます。これには、入力発話で検出されたエンティティだけでなく、最上位の (最も可能性の高い) 意図も含まれます。 クライアント アプリケーションは、その予測を使用して適切なアクションを決定および実行する必要があります。

3. コメント **Apply the appropriate action** を見つけ、次のコードを追加します。このコードは、アプリケーションでサポートされている意図 (**GetTime**、**GetDate**、および **GetDay**) をチェックします。また、適切な応答を生成するために既存の関数を呼び出す前に、関連するエンティティが検出されたかどうかを判断します。

**C#**

```C#
// Apply the appropriate action
switch (topIntent)
{
    case "GetTime":
        var location = "local";
        // Check for entities
        if (entities.Count > 0)
        {
            // Check for a location entity
            if (entities.ContainsKey("Location"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Location"].ToString());
                // ML entities are strings, get the first one
                location = entityJson[0].ToString();
            }
        }

        // Get the time for the specified location
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;

    case "GetDay":
        var date = DateTime.Today.ToShortDateString();
        // Check for entities
        if (entities.Count > 0)
        {
            // Check for a Date entity
            if (entities.ContainsKey("Date"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Date"].ToString());
                // Regex entities are strings, get the first one
                date = entityJson[0].ToString();
            }
        }
        // Get the day for the specified date
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;

    case "GetDate":
        var day = DateTime.Today.DayOfWeek.ToString();
        // Check for entities
        if (entities.Count > 0)
        {
            // Check for a Weekday entity
            if (entities.ContainsKey("Weekday"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Weekday"].ToString());
                // List entities are lists
                day = entityJson[0][0].ToString();
            }
        }
        // Get the date for the specified day
        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;

    default:
        // Some other intent (for example, "None") was predicted
        Console.WriteLine("Try asking me for the time, the day, or the date.");
        break;
}
```

**Python**

```Python
# Apply the appropriate action
if top_intent == 'GetTime':
    location = 'local'
    # Check for entities
    if len(entities) > 0:
        # Check for a location entity
        if 'Location' in entities:
            # ML entities are strings, get the first one
            location = entities['Location'][0]
    # Get the time for the specified location
    print(GetTime(location))

elif top_intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if len(entities) > 0:
        # Check for a Date entity
        if 'Date' in entities:
            # Regex entities are strings, get the first one
            date_string = entities['Date'][0]
    # Get the day for the specified date
    print(GetDay(date_string))

elif top_intent == 'GetDate':
    day = 'today'
    # Check for entities
    if len(entities) > 0:
        # Check for a Weekday entity
        if 'Weekday' in entities:
            # List entities are lists
            day = entities['Weekday'][0][0]
    # Get the date for the specified day
    print(GetDate(day))

else:
    # Some other intent (for example, "None") was predicted
    print('Try asking me for the time, the day, or the date.')
```
    
4. 変更を保存し、**clock-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python clock-client.py
```

5. プロンプトが表示されたら、発話を入力してアプリケーションをテストします。 たとえば、次の操作を試してください。

    *Hello*
    
    *今何時?*

    *What's the time in London?* \(ロンドンの時刻は何時ですか?\)

    *What's the date?* \(何日ですか?\)

    *What date is Sunday?* \(日曜日は何日ですか?\)

    *What day is it?* \(何曜日ですか?\)

    *What day is 01/01/2025?* \(2025 年 1 月 1 日は何曜日ですか?\)

> **注**: アプリケーションのロジックは意図的に単純であり、いくつかの制限があります。 たとえば、時間を取得する場合、制限された都市のセットのみがサポートされ、夏時間は無視されます。 目標は、アプリケーションが次のことを行う必要がある言語理解を使用するための一般的なパターンの例を確認することです。
>
>   1. 予測エンドポイントに接続します。
>   2. 予測を得るために発話を送信します。
>   3. 予測された意図とエンティティに適切に応答するロジックを実装します。

6. テストが終了したら、「*quit*」と入力します。

## <a name="more-information"></a>詳細情報

Language Understanding クライアントの作成の詳細については、[開発者向けドキュメント](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)を参照してください。
