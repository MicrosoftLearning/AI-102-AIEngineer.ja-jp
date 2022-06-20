---
lab:
  title: Cognitive Services を監視する
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: e0e0042421a4f7150fc3b95cef80887c81a78f3a
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "137819340"
---
# <a name="monitor-cognitive-services"></a>Cognitive Services を監視する

Azure Cognitive Services は、アプリケーション インフラストラクチャ全体の重要な部分になる可能性があります。 アクティビティを監視し、注意が必要な問題についてアラートを受け取ることができることが重要です。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボの作業をしている環境に既にクローンしている場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐクローンしてください。

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
3. 必要なチェックボックスを選択して、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースがデプロイされたら、そこに移動して、その **キーとエンドポイント** ページを表示します。 エンドポイント URI をメモします。後で必要になります。

## <a name="configure-an-alert"></a>アラートを構成する

Cognitive Services リソースのアクティビティを検出できるように、アラート ルールを定義して監視を開始しましょう。

1. Azure portal で、Cognitive Services リソースに移動し、( **[監視]** セクションで) **警告** ページを表示します。
2. **[+ 新しいアラート ルール]** を選択します
3. **警告ルールの作成** ページの **[スコープ]** で、Cognitive Services リソースが一覧表示されていることを確認します。
4. **[条件]** で、 **[条件の追加]** をクリックし、右側に表示される **[信号を選択]** ペインを確認します。ここで、監視する信号タイプを選択できます。
5. **[信号タイプ]** リストで **[アクティビティ ログ]** を選択し、次にフィルター処理されたリストで **[リスト キー]** を選択します。
6. 過去 6 時間のアクティビティを確認します。
7. **[アクション]** タブを選択します。*アクション グループ* を指定できることにご注意ください。 これにより、アラートが発生したときの自動アクションを構成できます。たとえば、電子メール通知を送信します。 この演習ではそれを行いません。ただし、実稼働環境でこれを行うと便利な場合があります。
8. **[詳細]** タブで、 **[警告ルール名]** を **[キー一覧警告]** に設定します。
9. **[Review + create]\(レビュー + 作成\)** を選択します。 
10. 警告の構成を確認します。 **[作成]** を選択し、警告ルールが作成されるのを待ちます。
11. Visual Studio Code で、**03-monitor** フォルダーを右クリックし、統合ターミナルを開きます。 次に、次のコマンドを入力して、Azure CLI を使用して Azure サブスクリプションにサインインします。

    ```
    az login
    ```

    Web ブラウザーのタブが開き、Azure にサインインするように求められます。 そうしてから、ブラウザー タブを閉じて、Visual Studio Code に戻ります。

    > **ヒント**: 複数のサブスクリプションがある場合は、確実に Cognitive Services リソースが含まれるサブスクリプションで作業している必要があります。  このコマンドを使用して、現在のサブスクリプションを判別します。
    >
    > ```
    > az account show
    > ```
    >
    > サブスクリプションを変更する必要がある場合は、このコマンドを実行して、 *&lt;subscriptionName&gt;* を正しいサブスクリプション名に変更します。
    >
    > ```
    > az account set --subscription <subscriptionName>
    > ```

10. これで、次のコマンドを使用して、Cognitive Services キーのリストを取得できます。 *&lt;resourceName&gt;* を Cognitive Services リソースの名前に置き換え、 *&lt;resourceGroup&gt;* を作成したリソース グループの名前に置き換えます。

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

このコマンドは、Cognitive Services リソースのキーのリストを返します。

11. Azure portal を含むブラウザーに戻り、**警告ページ** を更新します。 表に **Sev 4** 警告が表示されているはずです (表示されていない場合は、最大 5 分待ってから再度更新してください)。
12. アラートを選択して詳細を確認します。

## <a name="visualize-a-metric"></a>メトリックの視覚化

アラートを定義するだけでなく、Cognitive Services リソースのメトリックを表示して、その使用率を監視できます。

1. Azure portal の Cognitive Services リソースのページで、 **[メトリック]** ( **[監視]** セクション内) を選択します。
2. 既存のグラフがない場合は、 **[+ 新しいグラフ]** を選択します。 次に、 **[メトリック]** リストで、視覚化できる可能な指標を確認し、 **[合計呼び出し数]** を選択します。
3. **[集計]** リストで **[Count]\(カウント\)** を選択します。  これにより、Cognitive Services リソースへの合計呼び出し数を監視できるようになります。これは、一定期間にサービスがどれだけ使用されているかを判断するのに役立ちます。
4. Cognitive Services へのリクエストを生成するには、HTTP リクエストに **curl** コマンド ライン ツールを使用します。 Visual Studio Code で **03-monitor** フォルダーから **rest-test.cmd** を開き、そこに含まれる **curl** コマンド (以下に表示) を編集し、 *&lt;yourEndpoint&gt;* と *&lt;yourKey&gt;* をお使いのエンドポイント URI と **Key1** キーに置き換えて、Cognitive Services リソースで Text Analytics API を使用します。

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.1/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':           [{'id':1,'text':'hello'}]}"
    ```

5. 変更を保存してから、**03-monitor** フォルダーの統合ターミナルで次のコマンドを実行します。

    ```
    rest-test
    ```

このコマンドは、入力データで検出された言語に関する情報を含む JSON ドキュメントを返します (これは英語でなければなりません)。

6. **rest-test** コマンドを複数回再実行して、呼び出しアクティビティを生成します ( **^** キーを使用して前のコマンドを切り替えることができます)。
7. Azure portal の **[メトリック]** ページに戻り、**合計呼び出し数** のグラフを更新します。 *curl* を使用して行った呼び出しがグラフに反映されるまで、数分かかる場合があります。更新されて含まれるまで、グラフを更新し続けます。

## <a name="more-information"></a>詳細情報

Cognitive Services を監視するためのオプションの 1 つは、*診断ログ* を使用することです。 有効にすると、診断ログは Cognitive Services のリソース使用状況に関する豊富な情報をキャプチャし、便利な監視およびデバッグ ツールになります。 診断ログを設定して情報を生成するまでに 1 時間以上かかる場合があるため、この演習では取り上げていませんが、[Cognitive Services のドキュメント](https://docs.microsoft.com/azure/cognitive-services/diagnostic-logging)で詳細をご覧になれます。
