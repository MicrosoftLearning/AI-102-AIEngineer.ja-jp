---
lab:
  title: Cognitive Services セキュリティの管理
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: dcab47cf20f54d6bcbed9a3e40081b703fc2d5ba
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "137819508"
---
# <a name="manage-cognitive-services-security"></a>Cognitive Services セキュリティの管理

セキュリティはどのアプリケーションにとっても重要な考慮事項であり、開発者は、Cognitive Services などのリソースへのアクセスがそれを必要とする人だけに制限されていることを確認する必要があります。

Cognitive Services へのアクセスは通常、認証キーを介して制御されます。認証キーは、Cognitive Services リソースを最初に作成したときに生成されます。

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
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]**: 使用できるリージョンを選択します
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。

## <a name="manage-authentication-keys"></a>認証キーの管理

Cognitive Services リソースを作成すると、2 つの認証キーが生成されました。 これらは、Azure portal で、または Azure コマンド ライン インターフェイス (CLI) を使用して管理できます。

1. Azure portal で Cognitive Services リソースに移動し、その **[キーとエンドポイント]** ページを表示します。 このページには、リソースに接続して、開発したアプリケーションからリソースを使用するために必要な情報が含まれています。 具体的な内容は次のとおりです。
    - クライアント アプリケーションが要求を送信できる HTTP *エンドポイント*。
    - 認証に使用できる 2 つの *キー* (クライアント アプリケーションはどちらのキーも使用できます。 一般的な方法は、1 つを開発用に、もう 1 つを本番用に使用する方法です。 開発者が作業を終了した後、開発キーを簡単に再生成して、継続的なアクセスを防ぐことができます)。
    - リソースがホストされている *場所*。 これは、一部の (すべてではない) API へのリクエストに必要です。
2. Visual Studio Code で、**02-cognitive-security** フォルダーを右クリックし、統合ターミナルを開きます。 次に、次のコマンドを入力して、Azure CLI を使用して Azure サブスクリプションにサインインします。

    ```
    az login
    ```

    Web ブラウザーのタブが開き、Azure にサインインするように求められます。 そうしてから、ブラウザー タブを閉じて、Visual Studio Code に戻ります。

    > **ヒント**: 複数のサブスクリプションがある場合は、確実に Cognitive Services リソースが含まれるサブスクリプションで作業している必要があります。  このコマンドを使用して、現在のサブスクリプションを判別します。その一意の ID は、返される JSON の **ID** 値です。

    > **警告**: `az login` で証明書の検証エラーが発生する場合は、数分待ってからもう一度試してみてください。
    >
    > ```
    > az account show
    > ```
    >
    > サブスクリプションを変更する必要がある場合は、このコマンドを実行して、 *&lt;Your_Subscription_Id&gt;* を正しいサブスクリプション ID に変更します。
    >
    > ```
    > az account set --subscription <Your_Subscription_Id>
    > ```
    >
    > または、後続の各 Azure CLI コマンドで、サブスクリプション ID を *--subscription* パラメーターとして明示的に指定することもできます。

3. これで、次のコマンドを使用して、Cognitive Services キーのリストを取得できます。 *&lt;resourceName&gt;* を Cognitive Services リソースの名前に置き換え、 *&lt;resourceGroup&gt;* を作成したリソース グループの名前に置き換えます。

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

このコマンドは、Cognitive Services リソースのキーのリストを返します。**key1** と **key2** という名前の 2 つのキーがあります。

4. Cognitive Services をテストするには、HTTP 要求に **curl** コマンド ライン ツールを使用できます。 **02-cognitive-security** フォルダーで **rest-test.cmd** を開き、そこに含まれる **curl** コマンド (以下に表示) を編集し、 *&lt;yourEndpoint&gt;* と *&lt;yourKey&gt;* をお使いのエンドポイント URI と **Key1** キーに置き換えて、Cognitive Services リソースで Text Analytics API を使用します。

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.0/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':[{'id':1,'text':'hello'}]}"
    ```

5. 変更を保存してから、**02-cognitive-security** フォルダーの統合ターミナルで次のコマンドを実行します。

    ```
    rest-test
    ```

このコマンドは、入力データで検出された言語に関する情報を含む JSON ドキュメントを返します (これは英語でなければなりません)。

6. キーが危険にさらされた場合、またはキーを持っている開発者がアクセスを必要としなくなった場合は、ポータルで、または Azure CLI を使用してキーを再生成できます。 次のコマンドを実行して、**key1** キーを再生成します (リソースの *&lt;resourceName&gt;* と *&lt;resourceGroup&gt;* を置き換えます)。

    ```
    az cognitiveservices account keys regenerate --name <resourceName> --resource-group <resourceGroup> --key-name key1
    ```

Cognitive Services リソースのキーのリストが返されます。**key1** は最後に取得してから変更されていることに注意してください。

7. 古いキーを使用して **rest-test** コマンドを再実行し ( **^** キーを使用して前のコマンドを切り替えることができます)、失敗することを確認します。
8. **rest-test.cmd** の *curl* コマンドを編集して、キーを新しい **key1** 値に置き換え、変更を保存します。 次に、**rest-test** コマンドを再実行し、成功することを確認します。

> **ヒント**: この演習では、 **--resource-group** などの Azure CLI パラメーターのフルネームを使用しました。  **-g** などの短い代替手段を使用して、コマンドの冗長性を低くすることもできます (ただし、理解が少し難しくなります)。  [Cognitive Services CLI コマンド リファレンス](https://docs.microsoft.com/cli/azure/cognitiveservices?view=azure-cli-latest)には、各 Cognitive Services CLI コマンドのパラメーター オプションがリストされています。

## <a name="secure-key-access-with-azure-key-vault"></a>Azure Key Vault を使用した安全なキー アクセス

認証にキーを使用することで、Cognitive Services を利用するアプリケーションを開発できます。 ただし、これは、アプリケーションコードがキーを取得できる必要があることを意味します。 1 つのオプションは、アプリケーションがデプロイされている環境変数または構成ファイルにキーを格納することですが、このアプローチでは、キーが不正アクセスに対して脆弱なままになります。 Azure でアプリケーションを開発する場合のより良いアプローチは、キーを Azure Key Vault に安全に保存し、*マネージド ID* (つまり、アプリケーション自体が使用するユーザー アカウント) を介してキーへのアクセスを提供することです。

### <a name="create-a-key-vault-and-add-a-secret"></a>Key Vault を作成してシークレットを追加する

最初に、Key Vault を作成して Cognitive Services キーの *シークレット* を追加する必要があります。

1. Cognitive Services リソースの **key1** 値をメモします (またはクリップボードにコピーします)。
2. Azure portal の **[ホーム]** ページで、 **[&#65291; リソースの作成]** ボタンを選択し、*Key Vault* を検索して、次の設定で **Key Vault** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *Cognitive Services リソースと同じリソース グループ*
    - **キー コンテナー名**: *一意の名前を入力します*
    - **リージョン**: *Cognitive Services リソースと同じリージョン*
    - **価格レベル**: Standard
3. デプロイが完了するのを待ってから、Key Vault リソースに移動します。
4. 左側のナビゲーション ウィンドウで、**[シークレット]** ([設定] セクション内) を選択します。
5. **[+生成/インポート]** を選択し、次の設定で新しいシークレットを追加します。
    - **アップロード オプション**: 手動
    - **名前**: Cognitive-Services-Key *(後でこの名前に基づいてシークレットを取得するコードを実行するため、これを正確に一致させることが重要です)*
    - **値**: ***key1** Cognitive Services キー"*

### <a name="create-a-service-principal"></a>サービス プリンシパルの作成

Key Vault 内のシークレットにアクセスするには、アプリケーションはシークレットにアクセスできるサービス プリンシパルを使用する必要があります。 Azure コマンド ライン インターフェイス (CLI) を使用して、サービス プリンシパルを作成し、そのオブジェクト ID を検索し、Azure Vault のシークレットへのアクセスを許可します。

1. Visual Studio Code に戻り、**02-cognitive-security** フォルダーの対話型ターミナルで、 *&lt;spName&gt;* をアプリケーション ID に適した名前 (たとえば、*ai-app*) に置き換えて、次の Azure CLI コマンドを実行します。 また、 *&lt;subscriptionId&gt;* と *&lt;resourceGroup&gt;* を、サブスクリプション ID と、Cognitive Services および Key Vault リソースを含むリソース グループの正しい値に置き換えます。

    > **ヒント**: サブスクリプション ID が不明な場合は、**az account show** コマンドを使用してサブスクリプション情報を取得します。サブスクリプション ID は出力の **id** 属性です。

    ```
    az ad sp create-for-rbac -n "api://<spName>" --role owner --scopes subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
    ```

このコマンドの出力には、新しいサービス プリンシパルに関する情報が含まれます。 次のようになります。

    ```
    {
        "appId": "abcd12345efghi67890jklmn",
        "displayName": "ai-app",
        "name": "http://ai-app",
        "password": "1a2b3c4d5e6f7g8h9i0j",
        "tenant": "1234abcd5678fghi90jklm"
    }
    ```

**appId**、**password**、**tenant** の値を記録しておきます。後で必要になります (このターミナルを閉じると、パスワードを取得できなくなります。したがって、ここで値をメモすることが重要です。出力を Visual Studio Code の新しいテキスト ファイルに貼り付けて、後で必要な値を確実に見つけられるようにすることができます。)

2. サービス プリンシパルの **オブジェクト ID** を取得するには、次の Azure CLI コマンドを実行し、 *&lt;appId&gt;* をお使いのサービス プリンシパルのアプリ ID の値に置き換えます。

    ```
    az ad sp show --id <appId> --query objectId --out tsv
    ```

3. 新しいサービス プリンシパルに Key Vault のシークレットにアクセスするためのアクセス許可を割り当てるには、次の Azure CLI コマンドを実行し、 *&lt;keyVaultName&gt;* を Azure Key Vault リソースの名前に置き換え、 *&lt;objectId&gt;* をサービス プリンシパルのオブジェクト ID の値に置き換えます。

    ```
    az keyvault set-policy -n <keyVaultName> --object-id <objectId> --secret-permissions get list
    ```

### <a name="use-the-service-principal-in-an-application"></a>アプリケーションでサービス プリンシパルを使用する

これで、アプリケーションでサービス プリンシパル ID を使用する準備が整い、Key Vault にシークレット Cognitive Services キーを入力し、それを使用して Cognitive Services リソースに接続します。

> **注**: この演習では、サービス プリンシパルの資格情報をアプリケーション構成に格納し、それらを使用して、アプリケーション コードで **ClientSecretCredential** IDを認証します。 これは開発とテストには問題ありませんが、実際の運用アプリケーションでは、管理者は *マネージド ID* をアプリケーションに割り当て、パスワードをキャッシュしたり保存したりせずに、サービス プリンシパル ID を使用してリソースにアクセスします。

1. Visual Studio Code で、**02-cognitive-security** フォルダーと、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **keyvault-client** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Azure Key Vault と Text Analytics API を使用するために必要なパッケージを Cognitive Services リソースにインストールします。

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    dotnet add package Azure.Identity --version 1.5.0
    dotnet add package Azure.Security.KeyVault.Secrets --version 4.2.0-beta.3
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    pip install azure-identity==1.5.0
    pip install azure-keyvault-secrets==4.2.0
    ```

3. **keyvault-client** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、構成値を更新します。次の設定を反映するために含まれます
    
    - Cognitive Services リソースの **エンドポイント**
    - **Azure Key Vault** リソースの名前
    - サービス プリンシパルの **テナント**
    - サービス プリンシパルの **appId**
    - サービス プリンシパルの **パスワード**

     変更を保存します。
4. **keyvault-client** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#** : Program.cs
    - **Python**: keyvault-client.py

    コード ファイルを開き、含まれているコードを確認して、次の詳細に注意してください。
    - インストールした SDK の名前空間がインポートされます
    - **Main** 関数のコードはアプリケーション構成設定を取得します。次に、サービス プリンシパルの資格情報を使用して、Key Vault から Cognitive Services キーを取得します。
    - **GetLanguage** 関数は、SDK を使用してサービスのクライアントを作成し、クライアントを使用して入力されたテキストの言語を検出します。
5. **keyvault-client** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python keyvault-client.py
    ```

6. プロンプトが表示されたら、テキストを入力し、サービスによって検出された言語を確認します。 たとえば、「Hello」、「Bonjour」、「Hola」と入力してみてください。
7. アプリケーションのテストが終了したら、「quit」と入力してプログラムを終了します。

## <a name="more-information"></a>詳細情報

Cognitive Services のセキュリティ保護の詳細については、[Cognitive Services のセキュリティ ドキュメント](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-security)を参照してください。
