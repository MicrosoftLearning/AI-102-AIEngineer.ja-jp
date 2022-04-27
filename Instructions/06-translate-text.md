---
lab:
  title: テキストの翻訳
  module: Module 3 - Getting Started with Natural Language Processing
ms.openlocfilehash: 4ca4394ce5d9456abeabbdded1ee219f271f284e
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819425"
---
# <a name="translate-text"></a>テキストの翻訳

**Translator** サービスは、言語間でテキストを翻訳できるようにする Cognitive Services です。

たとえば、旅行代理店が、分析に使用される言語として英語を標準化して、会社の Web サイトに送信されたホテルのレビューを調べたいとします。 Translator サービスを使用することで、各レビューが書かれている言語を判別できます。まだ英語でない場合は、書かれているソース言語から英語に翻訳します。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボの作業をしている環境に既にクローンしている場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐクローンしてください。

1. Visual Studio Code を起動します。
2. パレットを開き (Shift + Ctrl + P)、**Git: Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。

## <a name="provision-a-cognitive-services-resource"></a>Cognitive Services リソースをプロビジョニングする

サブスクリプションに **Cognitive Services** リソースがまだない場合は、プロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*Cognitive Services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: "*リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)* "
    - **[リージョン]**: 使用できるリージョンを選択します
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースがデプロイされたら、そこに移動して、その **キーとエンドポイント** のページを表示します。 次の手順では、このページからサービスがプロビジョニングされるキーと場所の 1 つが必要になります。

## <a name="prepare-to-use-the-translator-service"></a>Translator サービスを使用する準備をする

この演習では、Translator REST API を使用してホテルのレビューを翻訳する部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** のいずれかから API を使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **エクスプローラー** ペインで、**06-translate-text** フォルダーを参照し、言語の設定に応じて、**C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. **text-translation** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの認証 **キー** と、それがデプロイされている **場所** (エンドポイントでは <u>ない</u>) を含めます。Cognitive Services リソース用に、**キーとエンドポイント** ページからこれらの両方をコピーする必要があります。 変更を保存します。
3. **text-translation** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: text-translation.py

    コード ファイルを開き、含まれているコードを調べます。

4. **Main** 関数では、構成ファイルから Cognitive Services のキーとリージョンをロードするコードがすでに提供されていることにご注意ください。 Translator サービスのエンドポイントもコードで指定されています。
5. **text-translation** フォルダーを右クリックし、統合ターミナルを開き、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

6. コードがエラーなしで実行され、**reviews** フォルダー内の各レビュー テキストファイルの内容が表示されるので、出力を確認します。 現在、アプリケーションは Translator サービスを利用していません。 次の手順で修正します。

## <a name="detect-language"></a>言語を検出する

Translator サービスは、翻訳されるテキストのソース言語を自動的に検出できますが、テキストが書かれている言語を明示的に検出することもできます。

1. コード ファイルで、**GetLanguage** 関数を見つけます。これにより、現在、すべてのテキスト値に対して "en" が返されます。
2. **GetLanguage** 関数のコメント **Use the Translator detect function** の下に、次のコードを追加し、Translator の REST API を使用して、指定されたテキストの言語を検出します。言語を返す関数の最後にあるコードを置き換えないようにご注意ください。

**C#**

```C#
// Use the Translator detect function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/detect?api-version=3.0";
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get language
        JArray jsonResponse = JArray.Parse(responseContent);
        language = (string)jsonResponse[0]["language"]; 
    }
}
```

**Python**

```Python
# Use the Translator detect function
path = '/detect'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0'
}

headers = {
'Ocp-Apim-Subscription-Key': cog_key,
'Ocp-Apim-Subscription-Region': cog_region,
'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get language
language = response[0]["language"]
```

3. 変更を保存して、**text-translation** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. 出力を観察します。今回は、各レビューの言語が識別されていることに注意してください。

## <a name="translate-text"></a>テキストを翻訳する

アプリケーションがレビューの作成言語を判別できるようになったので、Translator サービスを使用して、英語以外のレビューを英語に翻訳できます。

1. コード ファイルで、**Translate** 関数を見つけます。これにより、現在、すべてのテキスト値に対して空の文字列が返されます。
2. **Translate** 関数のコメント **Use the Translator translate function** の下に、次のコードを追加して、Translator の REST API を使用し、指定されたテキストをソース言語から英語に翻訳します。翻訳を返す関数の最後にあるコードを置き換えないようにご注意ください。

**C#**

```C#
// Use the Translator translate function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/translate?api-version=3.0&from=" + sourceLanguage + "&to=en" ;
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get translation
        JArray jsonResponse = JArray.Parse(responseContent);
        translation = (string)jsonResponse[0]["translations"][0]["text"];  
    }
}
```

**Python**

```Python
# Use the Translator translate function
path = '/translate'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0',
    'from': source_language,
    'to': ['en']
}

headers = {
    'Ocp-Apim-Subscription-Key': cog_key,
    'Ocp-Apim-Subscription-Region': cog_region,
    'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get translation
translation = response[0]["translations"][0]["text"]
```

3. 変更を保存して、**text-translation** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. 英語以外のレビューは英語に翻訳されていることに注意して、出力を観察します。

## <a name="more-information"></a>詳細情報

**Translator** サービスの使用の詳細については、[Translator のドキュメント](https://docs.microsoft.com/azure/cognitive-services/translator/)を参照してください。
