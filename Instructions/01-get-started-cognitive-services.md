---
lab:
  title: Cognitive Services の概要
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: 4baba38b03c6d7bb5fe04fa5e73bb606e970550b
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "137819563"
---
# <a name="get-started-with-cognitive-services"></a>Cognitive Services の概要

この演習では、Azure サブスクリプションで **Cognitive Services** リソースを作成し、それをクライアント アプリケーションから使用して、Cognitive Services の使用を開始します。 この演習の目標は、特定のサービスに関する専門知識を習得することではなく、開発者として Cognitive Services をプロビジョニングおよび操作するための一般的なパターンに精通することです。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

1. Visual Studio Code を起動します。
2. パレットを開き (Shift + Ctrl + P)、**Git: Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="provision-a-cognitive-services-resource"></a>Cognitive Services リソースをプロビジョニングする

Azure Cognitive Services は、アプリケーションに組み込むことができる人工知能機能をカプセル化するクラウドベースのサービスです。 特定の API (**言語** または **Computer Vision** など) に個別の **Cognitive Services** リソースをプロビジョニングすることも、単一のエンドポイントとキーを介して複数の Cognitive Services API へのアクセスを提供する一般的な Cognitive Services リソースをプロビジョニングすることもできます。 この場合、単一の **Cognitive Services** リソースを使用します。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*Cognitive Services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: "*リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)* "
    - **[リージョン]**: 使用できるリージョンを選択します
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースに移動し、その **[キーとエンドポイントページ]** ページを表示します。 このページには、リソースに接続して、開発したアプリケーションからリソースを使用するために必要な情報が含まれています。 具体的な内容は次のとおりです。
    - クライアント アプリケーションが要求を送信できる HTTP *エンドポイント*。
    - 認証に使用できる 2 つの "*キー*" (クライアント アプリケーションはどちらのキーも認証に使用できます)。
    - リソースがホストされている "*場所*"。 これは、一部の (すべてではない) API へのリクエストに必要です。

## <a name="use-a-rest-interface"></a>REST インターフェイスの使用

Cognitive Services API は REST ベースであるため、HTTP 経由で JSON リクエストを送信することで API を利用できます。 この例では、**言語** REST API を使用して言語検出を実行するコンソール アプリケーションについて説明します。ただし、基本的な原則は、Cognitive Services リソースでサポートされているすべての API で同じです。

> **注**: この演習では、**C#** または **Python** のいずれかから REST API を使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **[エクスプローラー]** ウィンドウで、**01-getting-started** フォルダーを参照し、言語の設定に応じて **C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. **rest-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの **エンドポイント** と認証 **キー** を反映します。 変更を保存します。
4. **rest-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: rest-client.py

    コード ファイルを開き、含まれているコードを確認して、次の詳細に注意してください。
    - HTTP 通信を可能にするために、さまざまな名前空間がインポートされます
    - **Main** 関数のコードは、Cognitive Services リソースのエンドポイントとキーを取得します。これらは、REST リクエストを Text Analytics サービスに送信するために使用されます。
    - プログラムはユーザー入力を受け入れ、**GetLanguage** 関数を使用して、Cognitive Services エンドポイントの Text Analytics 言語検出 REST API を呼び出して、入力されたテキストの言語を検出します。
    - API に送信される要求は、入力データを含む JSON オブジェクトで構成されます。この場合、**ドキュメント** オブジェクトのコレクションであり、それぞれに **ID** と **テキスト** があります。
    - サービスのキーは、クライアント アプリケーションを認証するためのリクエスト ヘッダーに含まれています。
    - サービスからの応答は JSON オブジェクトであり、クライアント アプリケーションはこれを解析できます。
5. **rest-client** フォルダーを右クリックして、統合ターミナルを開きます。 次に、次の言語固有のコマンドを入力して、プログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python rest-client.py
    ```

6. プロンプトが表示されたら、テキストを入力し、サービスによって検出された言語を確認します。これは、JSON 応答で返されます。 たとえば、「Hello」、「Bonjour」、「Hola」と入力してみてください。
7. アプリケーションのテストが終了したら、「quit」と入力してプログラムを終了します。

## <a name="use-an-sdk"></a>SDK を使用する

Cognitive Services のREST API を直接使用するコードを記述できますが、Microsoft C#、Python、Node.js など、多くの一般的なプログラミング言語用のソフトウェア開発キット (SDK) があります。 SDK を使用すると、Cognitive Services を使用するアプリケーションの開発を大幅に簡素化できます。

1. Visual Studio Code の **[エクスプローラー]** ウィンドウの **01-getting-started** フォルダーで、言語の設定に応じて **C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. **sdk-client** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適合するコマンドを実行して、Text Analytics SDK パッケージをインストールします。

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    ```

3. **sdk-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの **エンドポイント** と認証 **キー** を反映します。 変更を保存します。
    
4. **sdk-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: sdk-client.py

    コード ファイルを開き、含まれているコードを確認して、次の詳細に注意してください。
    - インストールした SDK の名前空間がインポートされます
    - **Main** 関数のコードは、Cognitive Services リソースのエンドポイントとキーを取得します。これらは SDK で使用され、Text Analytics サービスのクライアントを作成します。
    - **GetLanguage** 関数は、SDK を使用してサービスのクライアントを作成し、クライアントを使用して入力されたテキストの言語を検出します。
5. **sdk-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python sdk-client.py
    ```

6. プロンプトが表示されたら、テキストを入力し、サービスによって検出された言語を確認します。 たとえば、「Goodbye」、「Au revoir」、「Hasta la vista」と入力してみてください。
7. アプリケーションのテストが終了したら、「quit」と入力してプログラムを終了します。

> **注**: Unicode 文字セットを必要とする一部の言語は、この単純なコンソール アプリケーションでは認識されない場合があります。

## <a name="more-information"></a>詳細情報

Azure Cognitive Services の詳細については、[Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/what-are-cognitive-services) のドキュメントを参照してください。
