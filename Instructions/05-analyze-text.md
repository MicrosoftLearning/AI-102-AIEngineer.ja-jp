---
lab:
  title: テキストの分析
  module: Module 3 - Getting Started with Natural Language Processing
ms.openlocfilehash: 27cd22e81d4fe8fda27e6fc960d3b3aeb8debded
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "137819534"
---
# <a name="analyze-text"></a>テキストの分析

**言語** サービスは、言語検出、感情分析、キー フレーズ抽出、エンティティ認識など、テキストの分析をサポートする Cognitive Services です。

たとえば、旅行代理店が会社の Web サイトに送信されたホテルのレビューを処理したいとします。 言語サービスを使用すると、各レビューが書かれている言語、レビューの感情 (ポジティブ、ニュートラル、ネガティブ)、レビューで議論されている主なトピックを示す可能性のあるキー フレーズ、場所、ランドマーク、またはレビューで言及された人などの名前付きエンティティなどを特定できます。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="provision-a-cognitive-services-resource"></a>Cognitive Services リソースをプロビジョニングする

サブスクリプションに **Cognitive Services** リソースがまだない場合は、プロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*Cognitive Services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **[リソース グループ]** : "*リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がない可能性があります - 提供されているものを使用してください)* "
    - **[リージョン]**: 使用できるリージョンを選択します
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースがデプロイされたら、そこに移動して、その **[キーとエンドポイント]** ページを表示します。 次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## <a name="prepare-to-use-the-language-sdk-for-text-analytics"></a>テキスト分析に Language SDK を使用する準備をする

この演習では、言語サービス テキスト分析 SDK を使用してホテルのレビューを分析する、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**:**C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**05-analyze-text** フォルダーを参照し、言語の設定に応じて、**C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. **text-analysis** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適合するコマンドを実行して、Text Analytics SDK パッケージをインストールします。
    
    **C#**
    
    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    ```
    
    **Python**
    
    ```
    pip install azure-ai-textanalytics==5.1.0
    ```
    
3. **text-analysis** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの **エンドポイント** と認証 **キー** を反映します。 変更を保存します。

4. **text-analysis** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: text-analysis.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Text Analytics SDK を使用するために必要な名前空間インポートします。

    **C#**
    
    ```C#
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**

    ```Python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

5. **Main** 関数で、構成ファイルから Cognitive Services のエンドポイントとキーを読み込むためのコードが既に提供されていることにご注意ください。 次に、**エンドポイントとキーを使用してクライアントを作成する** というコメントを見つけ、次のコードを追加して、Text Analysis API のクライアントを作成します。

    **C#**

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(cogSvcKey);
    Uri endpoint = new Uri(cogSvcEndpoint);
    TextAnalyticsClient CogClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(cog_key)
    cog_client = TextAnalyticsClient(endpoint=cog_endpoint, credential=credential)
    ```

6. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

6. コードがエラーなしで実行され、**reviews** フォルダー内の各レビュー テキストファイルの内容が表示されるので、出力を確認します。 アプリケーションは Text Analytics API のクライアントを正常に作成しますが、それを利用しません。 次の手順で修正します。

## <a name="detect-language"></a>言語を検出する

Text Analytics API のクライアントを作成したので、それを使用して、各レビューが書かれている言語を検出しましょう。

1. プログラムの **Main** 関数で、コメント **Get language** を見つけます。 次に、このコメントの下に、各レビュー ドキュメントで言語を検出するために必要なコードを追加します。

    **C#**
    
    ```C
    // Get language
    DetectedLanguage detectedLanguage = CogClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**
    
    ```Python
    # Get language
    detectedLanguage = cog_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

    > **注**: *この例では、各レビューが個別に分析され、ファイルごとにサービスが個別に呼び出されます。別の方法として、ドキュメントのコレクションを作成し、1 回の呼び出しでサービスに渡す方法があります。どちらの方法でも、サービスからの応答はドキュメントのコレクションで構成されます。これは、上記の Python コードでは、応答 ([0]) 内の最初の (そして唯一の) ドキュメントのインデックスが指定されている理由です。*

6. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

7. 出力を観察します。今回は、各レビューの言語が識別されていることに注意してください。

## <a name="evaluate-sentiment"></a>感情の評価

"*感情分析*" は、テキストを "*ポジティブ*" または "*ネガティブ*" ("*ニュートラル*" または "*混合*" の場合もある) として分類するために一般的に使用される手法です。 ソーシャル メディアの投稿、製品レビュー、およびテキストの感情が有用な洞察を提供する可能性があるその他のアイテムを分析するために一般的に使用されます。

1. プログラムの **Main** 関数で、コメント **Get sentiment** を見つけます。 次に、このコメントの下に、各レビュードキュメントの感情を検出するために必要なコードを追加します。

    **C#**
    
    ```C
    // Get sentiment
    DocumentSentiment sentimentAnalysis = CogClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**
    
    ```Python
    # Get sentiment
    sentimentAnalysis = cog_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

2. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

3. 出力を観察し、レビューの感情が検出されたことに注意します。

## <a name="identify-key-phrases"></a>キー フレーズの特定

テキストの本文でキーフレーズを特定すると、説明する主なトピックを特定するのに役立ちます。

1. プログラムの **Main** 関数で、コメント **Get key phrases** を見つけます。 次に、このコメントの下に、各レビュードキュメントのキー フレーズを検出するために必要なコードを追加します。

    **C#**

    ```C
    // Get key phrases
    KeyPhraseCollection phrases = CogClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```
    
    **Python**
    
    ```Python
    # Get key phrases
    phrases = cog_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

2. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. 各ドキュメントにキー フレーズが含まれていることに注意して、出力を確認します。それはレビューが何であるかについてのいくつかの洞察を与えます。

## <a name="extract-entities"></a>エンティティの抽出

多くの場合、ドキュメントまたはその他のテキスト本体は、人、場所、期間、またはその他のエンティティについて言及しています。 Text Analytics API は、テキスト内のエンティティの複数のカテゴリ (およびサブカテゴリ) を検出できます。

1. プログラムの **Main** 関数で、コメント **Get entities** を見つけます。 次に、このコメントの下に、各レビューで言及されているエンティティを識別するために必要なコードを追加します。

    **C#**
    
    ```C
    // Get entities
    CategorizedEntityCollection entities = CogClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**
    
    ```Python
    # Get entities
    entities = cog_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

2. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. 出力を確認します。テキストで検出されたエンティティに注意してください。

## <a name="extract-linked-entities"></a>リンクされたエンティティを抽出する

分類されたエンティティに加えて、Text Analytics API は、Wikipedia などのデータソースへの既知のリンクがあるエンティティを検出できます。

1. プログラムの **Main** 関数で、コメント **Get linked entities** を見つけます。 次に、このコメントの下に、各レビューで言及されているリンクされたエンティティを識別するために必要なコードを追加します。

    **C#**
    
    ```C
    // Get linked entities
    LinkedEntityCollection linkedEntities = CogClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**
    
    ```Python
    # Get linked entities
    entities = cog_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

2. 変更を保存して、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. 出力を確認します。識別されたリンクされたエンティティに注意してください。

## <a name="more-information"></a>詳細情報

**言語** サービスの使用の詳細については、[Text Analytics のドキュメント](https://docs.microsoft.com/azure/cognitive-services/language-service/)を参照してください。
