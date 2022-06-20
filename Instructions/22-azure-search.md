---
lab:
  title: Azure Cognitive Search ソリューションを作成する
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: bba5786ed34cbbe806c74e2b2eec6286cb92554b
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "137819566"
---
# <a name="create-an-azure-cognitive-search-solution"></a>Azure Cognitive Search ソリューションを作成する

すべての組織は、意思決定を行い、疑問に答え、効率的に機能するために情報を利用しています。 ほとんどの組織にとって問題となっているのは、情報の不足ではなく、情報が格納されている大量の一連のドキュメント、データベース、およびその他のソースから情報を検索して抽出するという課題です。

たとえば、*Margie's Travel* は世界各地の都市への旅行の手配に特化した旅行代理店だとします。 長い時間をかけて、同社は、パンフレットなどのドキュメントや顧客から送信されたホテルのレビューに含まれている大量の情報を蓄積してきました。 このデータは、旅行代理店の従業員や顧客にとって旅行を計画する際に価値のある分析情報源となりますが、データ量が膨大であるため、特定の顧客の疑問に答える際に関連情報を見つけることが困難な場合があります。

この課題に対処するために、Margie'sTravel は Azure Cognitive Search を使用して、AI ベースのコグニティブ スキルを使用してドキュメントにインデックスを付け、強化して検索を容易にするソリューションを実装します。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-azure-resources"></a>Azure リソースを作成する

Margie's Travel 用に作成するソリューションでは、お使いの Azure サブスクリプションに次のリソースが必要です。

- **Azure Cognitive Search リソース**。インデックス作成とクエリ実行を管理します。
- **Cognitive Services** リソース。AI によって生成された分析情報を使用してデータ ソースのデータをエンリッチするために、検索ソリューションで使用できるスキルを AI サービスに提供します。
- 検索対象のドキュメントが保存されている BLOB コンテナーを持つ **ストレージ アカウント**。

> **重要**: Azure Cognitive Search と Cognitive Services のリソースは同じ場所にある必要があります。

### <a name="create-an-azure-cognitive-search-resource"></a>Azure Cognitive Search リソースを作成する

1. Web ブラウザーで Azure portal (`https://portal.azure.com`) を開き、自分の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*search* を検索して、次の設定で **Azure Cognitive Search** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *新しいリソース グループを作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がない可能性があります - 提供されているものを使ってください)*
    - **サービス名**: *一意の名前を入力します*
    - **場所**: *場所を選択します - Azure Cognitive Search リソースと Cognitive Services リソースは同じ場所にある必要があります*
    - **価格レベル**: Basic

3. デプロイが完了するまで待ち、デプロイされたリソースに移動します。
4. Azure portal の Azure Cognitive Search リソースのブレードの **[概要]** ページを確認します。 ここでは、ビジュアル インターフェイスを使用して、検索ソリューションのさまざまなコンポーネントを作成、テスト、管理、および監視できます。データソース、インデックス、インデクサー、スキルセットを含みます。

### <a name="create-a-cognitive-services-resource"></a>Cognitive Services リソースの作成

サブスクリプションに **Cognitive Services** リソースがまだない場合は、プロビジョニングする必要があります。 検索ソリューションではこれを使用して、AI によって生成された分析情報でデータストア内のデータをエンリッチします。

1. Azure portal のホーム ページに戻り、**[&#65291;リソースの作成]** ボタンを選択し、*cognitive services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: ''*Azure Cognitive Search リソースと同じリソース グループ*''
    - **リージョン**: ''*Azure Cognitive Search リソースと同じ場所*''
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
2. 必要なチェック ボックスをオンにして、リソースを作成します。
3. デプロイが完了するまで待ち、デプロイの詳細を表示します。

### <a name="create-a-storage-account"></a>ストレージ アカウントの作成

1. Azure portal のホーム ページに戻り、**[&#65291;リソースの作成]** ボタンを選択し、''*ストレージ アカウント*'' を検索して、次の設定で **ストレージ アカウント** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: **Azure Cognitive Search および Cognitive Services リソースと同じリソース グループ*
    - **ストレージ アカウント名**: *一意の名前を入力します*
    - **[リージョン]**: 使用できるリージョンを選択します
    - **パフォーマンス**: 標準
    - **レプリケーション**: ローカル冗長ストレージ (LRS)
2. デプロイが完了するまで待ち、デプロイされたリソースに移動します。
3. **[概要]** ページで、**サブスクリプション ID** に注意してください。これはストレージ アカウントがプロビジョニングされているサブスクリプションを識別します。
4. **[アクセスキー]** ページで、ストレージ アカウント用に 2 つのキーが生成されていることに注意してください。 次に、 **[キーの表示]** を選択してキーを表示します。

    > **ヒント**: **ストレージ アカウント** ブレードを開いたままにします。次の手順では、サブスクリプション ID とキーの 1 つが必要になります。

## <a name="upload-documents-to-azure-storage"></a>Azure Storage にドキュメントをアップロードする

必要なリソースが揃ったので、いくつかのドキュメントを　Azure　Storage　アカウントにアップロードできます。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**22-create-a-search-solution** フォルダーを展開し、**UploadDocs.cmd** を選択します。
2. バッチ ファイルを編集して、**YOUR_SUBSCRIPTION_ID**、**YOUR_AZURE_STORAGE_ACCOUNT_NAME**、および　**YOUR_AZURE_STORAGE_KEY** プレースホルダーを、以前に作成したストレージ アカウントの適切なサブスクリプション ID、Azure ストレージ アカウント名、および Azure ストレージ アカウント キーの値に置き換えます。
3. 変更を保存してから、**22-create-a-search-solution** フォルダーを右クリックして、統合ターミナルを開きます。
4. 次のコマンドを入力して、Azure CLI を使用して Azure サブスクリプションにサインインします。

    ```
    az login
    ```

Web ブラウザーのタブが開き、Azure にサインインするように求められます。 そうしてから、ブラウザー タブを閉じて、Visual Studio Code に戻ります。

5. 次のコマンドを入力して、バッチファイルを実行します。 これにより、ストレージ アカウントに BLOB コンテナーが作成され、**データ** フォルダー内のドキュメントがそこにアップロードされます。

    ```
    UploadDocs
    ```

## <a name="index-the-documents"></a>ドキュメントのインデックスを作成する

ドキュメントが配置されたので、インデックスを作成して検索ソリューションを作成できます。

1. Azure portal で、Azure Cognitive Search リソースを参照します。 その後、その **[概要]** ページで、**[データのインポート]** を選択します。
2. **[データへの接続]** ページの **[データ ソース]** リストで、**[Azure Blob Storage]** を選択します。 その後、次の値でデータ ストアの詳細を入力します。
    - **データ ソース**: Azure BLOB ストレージ
    - **データ ソース名**: margies-data
    - **抽出するデータ**: コンテンツとメタデータ
    - **解析モード**: 既定
    - **接続文字列**: ***[既存の接続を選択します]** を選択します。次に、ストレージ アカウントを選択し、UploadDocs.cmd スクリプトによって作成された **margies** コンテナーを選択します。*
    - **マネージド ID 認証:** なし
    - **コンテナー名**: margies
    - **BLOB フォルダー**: ''これは空白のままにします''
    - **説明**: Margie's Travel の Web サイトのパンフレットとレビュー。
3. 次の手順 (*コグニティブ スキルを追加する*) に進みます。
4. **[Cognitive Services をアタッチする]** セクションで、ご利用の Cognitive Services リソースを選択します。
5. **[エンリッチメントの追加]** セクションで、次のことを行います。
    - **スキルセット名** を **margies-skillset** に変更します。
    - **[OCR を有効にし、すべてのテキストを merged_content フィールドにマージする]** オプションを選択します。
    - **ソース データ フィールド** が **merged_content** に設定されていることを確かめます。
    - **[エンリッチメントの粒度レベル]** を **[ソース フィールド]** のままにします。このフィールドには、インデックスを作成するドキュメントのコンテンツ全体が設定されます。ただし、これを変更して、ページや文など、より詳細なレベルで情報を抽出できることに注意してください。
    - 次のエンリッチされたフィールドを選択します。

        | コグニティブ スキル | パラメーター | フィールド名 |
        | --------------- | ---------- | ---------- |
        | 場所の名前を抽出 | | locations |
        | キー フレーズ抽出 | | keyphrases |
        | 言語を検出する | | language |
        | 画像からタグを生成する | | imageTags |
        | 画像からキャプションを生成する | | imageCaption |

6. 選択内容をもう一度確認します (後で変更するのは困難な場合があります)。 次に、次の手順 (*ターゲット インデックスのカスタマイズ*) に進みます。
7. **インデックス名** を **margies-index** に変更します。
8. **[キー]** が **metadata_storage_path** に設定されていることを確かめます。 **[Suggester 名]** は空白のままにし、 **[検索モード]** は既定のままにします。
9. インデックス フィールドに次の変更を加え、他のすべてのフィールドはデフォルト設定のままにします (**重要**: テーブル全体を表示するには、右にスクロールする必要がある場合があります)。

    | フィールド名 | [取得可能] | フィルターの適用 | 並べ替え可能 | ファセット可能 | 検索可能 |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | metadata_author | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | language | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | |

11. 選択内容を再確認し、特に注意して、各フィールドで正しい **取得可能**、**フィルター可能**、**並べ替え可能**、**ファセット可能**、および **検索可能** オプションが選択されていることを確認します (後で変更するのは難しい場合があります)。 次に、次の手順 (*インデクサーの作成*) に進みます。
12. **インデクサー名** を **margies-indexer** に変更します。
13. **[スケジュール]** は **[1 回]** に設定されているままにします。
14. **[詳細設定]** オプションを展開し、**[Base-64 エンコード キー]** オプションが選択されていることを確かめます (通常、エンコード キーを使用するとインデックスの効率が向上します)。
15. **[送信]** を選択して、データ ソース、スキルセット、インデックス、およびインデクサーを作成します。 インデクサーは自動的に実行され、インデックス作成パイプラインが実行されます。これにより、次のことが行われます。
    1. データ ソースからドキュメント メタデータ フィールドとコンテンツを抽出する
    2. コグニティブ スキルのスキルセットを実行して、追加のエンリッチされたフィールドを生成する
    3. 抽出されたフィールドをインデックスにマップする。
16. Azure Cognitive Search リソースの **[概要]** ページの下半分で、 **[インデクサー]** タブを表示します。ここに、新しく作成された **margies-indexer** が表示されるはずです。 数分待ち、 **[状態]** に成功が示されるまで **&orarr; [最新の情報に更新]** をクリックします。

## <a name="search-the-index"></a>インデックスを検索する

インデックスができたので、検索できます。

1. Azure Cognitive Search リソースの **[概要]** ブレードの上部で、 **[検索エクスプローラー]** を選択します。
2. 検索エクスプローラーの **[クエリ文字列]** ボックスに、「`*`」(1 つのアスタリスク) を入力し、 **[検索]** を選択します。

    このクエリは、インデックス内のすべてのドキュメントを JSON 形式で取得します。 結果を調べて、選択した認知スキルによって抽出されたドキュメント コンテンツ、メタデータ、および強化されたデータを含む各ドキュメントのフィールドをメモします。

3. クエリ文字列を `search=*&$count=true` に変更し、検索を送信します。

    今回の結果には、検索によって返されたドキュメントの数を示す **@odata.count** フィールドが結果の上部に含まれています。

4. 次のクエリ文字列を試してください：

    ```
    search=*&$count=true&$select=metadata_storage_name,metadata_author,locations
    ```

    今回の結果には、ファイル名、作成者、およびドキュメントの内容に記載されている場所のみが含まれます。 ファイル名と作成者は、ソース ドキュメントから抽出された **metadata_storage_name** および **metadata_author** フィールドにあります。 **locations** フィールドは、コグニティブ スキルによって生成されました。

5. 次に、次のクエリ文字列を試してください。

    ```
    search="New York"&$count=true&$select=metadata_storage_name,keyphrases
    ```

    この検索では、検索可能なフィールドのいずれかで「ニューヨーク」に言及しているドキュメントが検索され、ドキュメント内のファイル名とキーフレーズが返されます。

6. もう 1 つのクエリ文字列を試してみましょう。

    ```
    search="New York"&$count=true&$select=metadata_storage_name&$filter=metadata_author eq 'Reviewer'
    ```

    このクエリは、"ニューヨーク" に言及している *Reviewer* によって作成されたドキュメントのファイル名を返します。

## <a name="explore-and-modify-definitions-of-search-components"></a>検索コンポーネント定義の調査と変更

検索ソリューションのコンポーネントは、Azure portal　で表示および編集できる　JSON　定義に基づいています。

ポータルを使用して検索ソリューションを作成および変更できますが、検索オブジェクトを JSON で定義し、Azure Cognitive Service REST インターフェイスを使用してそれらを作成および変更することが望ましい場合がよくあります。

### <a name="get-the-endpoint-and-key-for-your-azure-cognitive-search-resource"></a>Azure Cognitive Search リソースのエンドポイントとキーを取得する

1. Azure portal で、Azure Cognitive Search リソースの **[概要]** ページに戻ります。ページの上部で、リソースの **Url** ( **https://resource_name.search.windows.net** のようになります) を見つけてクリップボードにコピーします。
2. Visual Studio Code の [エクスプローラー] ペインで、**22-create-a-search-solution** フォルダーとその **modify-search** サブフォルダーを展開し、**modify-search.cmd** を選択して開きます。 このスクリプト ファイルを使用して、JSON を Azure Cognitive Service REST インターフェイスに送信する *cURL* コマンドを実行します。
3. **modify-search.cmd** で、**YOUR_SEARCH_URL** プレースホルダーをクリップボードにコピーした URL に置き換えます。
4. Azure portal で、Azure Cognitive Search リソースの **[キー]** ページを表示し、**プライマリ管理者キー** をクリップボードにコピーします。
5. Visual Studio Code で、**YOUR_ADMIN_KEY** プレースホルダーをクリップボードにコピーしたキーに置き換えます。
6. 変更を **modify-search.cmd** に保存します (ただし、まだ実行しないでください)

### <a name="review-and-modify-the-skillset"></a>スキルセットを確認および変更する

1. Visual studio Code の **modify-search** フォルダーで、**skillset.json** を開きます。 これは、**margies-skillset** の JSON 定義を示しています。
2. スキルセット定義の上部にある **cognitiveServices** オブジェクトに注意してください。このオブジェクトは、Cognitive Services リソースをスキルセットに接続するために使用されます。
3. Azure portal で、Cognitive Services リソース (Azure Cognitive Search リソースではあり <u>ません</u>。) を開き、その **キー** ページを表示します。 次に、**キー 1** をクリップボードにコピーします。
4. Visual Studio Code の **skillset.json** で、**YOUR_COGNITIVE_SERVICES_KEY** プレースホルダーをクリップボードにコピーした Cognitive Services キーに置き換えます。
5. JSON ファイルをスクロールして、Azure portal の Azure Cognitive Search ユーザー インターフェイスを使用して作成したスキルの定義が含まれていることに注意してください。 スキルのリストの下部に、次の定義で追加のスキルが追加されました。

    ```
    {
        "@odata.type": "#Microsoft.Skills.Text.V3.SentimentSkill",
        "defaultLanguageCode": "en",
        "name": "get-sentiment",
        "description": "New skill to evaluate sentiment",
        "context": "/document",
        "inputs": [
            {
                "name": "text",
                "source": "/document/merged_content"
            },
            {
                "name": "languageCode",
                "source": "/document/language"
            }
        ],
        "outputs": [
            {
                "name": "sentiment",
                "targetName": "sentimentLabel"
            }
        ]
    }
    ```

新しいスキルは **get-sentiment** という名前で、ドキュメント内の **ドキュメント** レベルごとに、インデックスが作成されているドキュメントの **merged_content** フィールドで見つかったテキストを評価します (これには、ソース コンテンツと、コンテンツ内の画像から抽出されたテキストが含まれます)。 ドキュメントの抽出された **言語** (既定は英語) を使用し、コンテンツのセンチメントのスコアを評価します。 センチメント ラベルの値には、"positive"、"negative"、"neutral"、または "mixed" を指定できます。 このラベルは、**sentimentLabel** という名前の新しいフィールドとして出力されます。

6. **skillset.json** に加えた変更を保存します。

### <a name="review-and-modify-the-index"></a>インデックスを確認して変更する

1. Visual Studio Code の **modify-search** フォルダーで、**index.json** を開きます。 これは、**margies-index** の JSON 定義を示しています。
2. インデックスをスクロールして、フィールド定義を表示します。 一部のフィールドはソースドキュメントのメタデータとコンテンツに基づいており、その他のフィールドはスキルセットのスキルの結果です。
3. Azure portal で定義したフィールドのリストの最後に、2 つのフィールドが追加されていることに注意してください

    ```
    {
        "name": "sentiment",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "sortable": true
    },
    {
        "name": "url",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "searchable": false,
        "sortable": false
    }
    ```

4. **sentiment** フィールドは、スキルセットが追加された **get-sentiment** スキルからの出力を追加するために使用されます。 **url** フィールドは、データ ソースから抽出された **metadata_storage_path** 値に基づいて、インデックス付けされた各ドキュメントの URL をインデックスに追加するために使用されます。 インデックスには既に **metadata_storage_path** フィールドが含まれていますが、インデックス キーとして使用され、Base-64 でエンコードされているため、キーとして効率的ですが、実際の URL 値をフィールドとして使用する場合は、クライアント アプリケーションでデコードする必要があります。 エンコードされていない値に2番目のフィールドを追加すると、この問題が解決します。

### <a name="review-and-modify-the-indexer"></a>インデクサーを確認して変更する

1. Visual Studio Code の **modify-search** フォルダーで、**indexer.json** を開きます。 これは、**margies-indexer** の JSON 定義を示しています。これは、ドキュメントのコンテンツとメタデータから抽出されたフィールド (**fieldMappings** セクション) と、スキルセットのスキルによって抽出された値 (**outputFieldMappings** セクション) をインデックスのフィールドにマップします。
3. **fieldMappings** リストで、**metadata_storage_path** 値の base-64 エンコード キー フィールドへのマッピングに注意してください。 これは、**metadata_storage_path** をキーとして割り当て、Azure portal でキーをエンコードするオプションを選択したときに作成されました。 さらに、新しいマッピングは、同じ値を **url** フィールドに明示的にマップしますが、Base-64 エンコーディングは使用しません。

    ```
    {
        "sourceFieldName" : "metadata_storage_path",
        "targetFieldName" : "url"
    }
    
    ```

ソース ドキュメント内の他のすべてのメタデータおよびコンテンツ フィールド インデックス内の同じ名前のフィールドに暗黙的にマップされます。

4. スキルセットのスキルからの出力をインデックス フィールドにマップする **ouputFieldMappings** セクションを確認します。 これらのほとんどは、ユーザー インターフェイスで行った選択を反映していますが、センチメント スキルによって抽出された **sentimentLabel** 値を、インデックスに追加した **sentiment** フィールドにマッピングするために、次のマッピングが追加されています。

    ```
    {
        "sourceFieldName": "/document/sentimentLabel",
        "targetFieldName": "sentiment"
    }
    ```

### <a name="use-the-rest-api-to-update-the-search-solution"></a>REST API を使用して検索ソリューションを更新する

1. **modify-search** フォルダーを右クリックして、統合ターミナルを開きます。
2. **modify-search** フォルダーのターミナル ペインで、次のコマンドを入力して、**modify-search.cmd** スクリプトを実行します。このスクリプトは、JSON 定義を REST インターフェイスに送信し、インデックス作成を開始します。

    ```
    modify-search
    ```

3. スクリプトが終了したら、Azure portal の　Azure Cognitive Search リソースの **[概要]** ページに戻り、 **[インデクサー]** 　ページを表示します。 定期的に **[更新]** を選択して、インデックス作成操作の進行状況を追跡できます。 完了するまでに 1 分ほどかかる場合があります。

    *センチメントを評価するには大きすぎるいくつかのドキュメントに対して、いくつかの警告がある場合があります。多くの場合、感情分析は、ドキュメント全体ではなく、ページまたは文レベルで実行されます。ただし、この場合のシナリオでは、ほとんどのドキュメント (特にホテルのレビュー) は、有用なドキュメント レベルのセンチメント スコアを評価するのに十分なほど短いものです。*

### <a name="query-the-modified-index"></a>変更されたインデックスをクエリする

1. Azure Cognitive Search リソースのブレードの上部で、 **[検索エクスプローラー]** を選択します。
2. 検索エクスプローラーの **[クエリ文字列]** ボックスに、次のクエリ文字列を入力し、 **[検索]** を選択します。

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    このクエリを実行すると、*Reviewer* が作成した、*London* に言及していて正の **センチメント** ラベルが付いているすべてのドキュメント (つまり、ロンドンに言及している肯定的なレビュー) の **url**、**センチメント**、および **キーフレーズ** が取得されます。

3. **検索エクスプローラー** ページを閉じて、 **[概要]** ページに戻ります。

## <a name="create-a-search-client-application"></a>検索クライアント アプリケーションを作成する

有用なインデックスができたので、クライアント アプリケーションからそれを使用できます。 これを行うには、REST インターフェイスを使用し、要求を送信し、HTTP　を介して　JSO　N形式で応答を受信します。または、お好みのプログラミング言語用のソフトウェア開発キット (SDK) を使用することもできます。 この演習では、SDK を使用します。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

### <a name="get-the-endpoint-and-keys-for-your-search-resource"></a>検索リソースのエンドポイントとキーを取得する

1. Azure portal の Azure Cognitive Search　リソースの **[概要]** ページで、**https://*your_resource_name*.search.windows.net** のような **Url** 値に注意してください。 これは、検索リソースのエンドポイントです。
2. **[キー]** ページで、2 つの **管理者** キーと 1 つの **クエリ** キーがあることに注意してください。 *管理者* キーは、検索リソースを作成および管理するために使用されます。*クエリ* キーは、検索クエリを実行するだけでよいクライアント アプリケーションによって使用されます。

    *クライアント アプリケーションのエンドポイントとクエリ キーが必要になります。*

### <a name="prepare-to-use-the-azure-cognitive-search-sdk"></a>Azure Cognitive Search SDK を使用するための準備

1. Visual Studio Code の **[エクスプローラー]** ペインで、**22-create-a-search-solution** フォルダーを参照し、言語の設定に応じて **C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. **margies-travel** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Azure Cognitive Search SDK パッケージをインストールします。

    **C#**
    
    ```
    dotnet add package Azure.Search.Documents --version 11.1.1
    ```
    
    **Python**
    
    ```
    pip install azure-search-documents==11.0.0
    ```
    
3. **margies-travel** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Azure Cognitive Search リソースの **エンドポイント** と **クエリ キー** を反映します。 変更を保存します。

### <a name="explore-code-to-search-an-index"></a>インデックスを検索するためのコードの探索

**margies-travel** フォルダーには、検索機能を含む Web アプリケーション (Microsoft C# *ASP.NET Razor* Web アプリケーションまたは Python *Flask* アプリケーション) のコード ファイルが含まれています。

1. 選択したプログラミング言語に応じて、次のコードファイルを Web アプリケーションで開きます
    - **C#** :Pages/Index.cshtml.cs
    - **Python**: app.py
2. コード ファイルの上部にあるコメント「**Import search namespaces**」を見つけ、Azure Cognitive Search　SDK　で機能するようにインポートされた名前空間をメモします。
3. **search_query** 関数で、コメント「**Create a search client**」を見つけ、コードが Azure Cognitive Search リソースのエンドポイントとクエリ キーを使用して **SearchClient** オブジェクトを作成することに注意してください。
4. **search_query** 関数、コメント「**Submit search query**」を検索し、コードを確認して、次のオプションを使用して、指定されたテキストの検索を送信します。
    - 検索テキスト内の個々の単語を **すべて** 必要とする *検索モード* が見つかりました。
    - 検索で見つかったドキュメントの総数が結果に含まれます。
    - 結果は、提供されたフィルター式に一致するドキュメントのみを含むようにフィルター処理されます。
    - 結果は、指定された並べ替え順序に並べ替えられます。
    - **metadata_author** フィールドの各離散値は、フィルタリング用の事前定義された値を表示するために使用できる *ファセット* として返されます。
    - 検索語が強調表示された **merged_content** フィールドと **imageCaption** フィールドの最大 3 つの抽出が結果に含まれます。
    - 結果には、指定されたフィールドのみが含まれます。

### <a name="explore-code-to-render-search-results"></a>検索結果をレンダリングするためのコードを探索する

Web アプリには、検索結果を処理およびレンダリングするためのコードが既に含まれています。

1. 選択したプログラミング言語に応じて、次のコードファイルを Web アプリケーションで開きます
    - **C#** :Pages/Index.cshtml
    - **Python**: templates/search.html
2. コードを調べて、検索結果が表示されるページをレンダリングします。 次の点に注意してください。
    - ページは、ユーザーが新しい検索を送信するために使用できる検索フォームで始まります (Python バージョンのアプリケーションでは、このフォームは **base.html** テンプレートで定義されています)。これはページの先頭で参照されます。
    - 次に、2 番目のフォームがレンダリングされ、ユーザーは検索結果を絞り込むことができます。 このフォームのコードは、次の通りです。
        - 検索結果からドキュメントの数を取得して表示します。
        - **metadata_author** フィールドのファセット値を取得し、フィルタリングのオプション リストとして表示します。
        - 結果の並べ替えオプションのドロップダウンリストを作成します。
    - 次に、コードは検索結果を反復処理し、各結果を次のようにレンダリングします。
        - **metadata_storage_name** (ファイル名) フィールドを、**url** フィールドのアドレスへのリンクとして表示します。
        - **merged_content** フィールドと **imageCaption** フィールドで見つかった検索用語の *ハイライト* を表示して、コンテキストで検索用語を表示しやすくします。
        - **metadata_author**、**metadata_storage_size**、**metadata_storage_last_modified**、および **language** フィールドを表示します。
        - ドキュメントの **センチメント** ラベルを表示します。 positive、negative、neutral、mixed を指定できます。
        - 最初の 5 つの **keyphrases** (ある場合) を表示します。
        - 最初の 5 つの **locations** (ある場合) を表示します。
        - 最初の 5 つの **imageTags** (ある場合) を表示します。

### <a name="run-the-web-app"></a>Web アプリの実行

 1. **margies-travel** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    flask run
    ```

2. アプリが正常に起動したときに表示されるメッセージで、実行中の Web アプリケーション ( *http://localhost:5000/* または *http://127.0.0.1:5000/* ) へのリンクをたどって、Web ブラウザで Margies Travel サイトを開きます。
3. Margie's Travel の Web サイトで、検索ボックスに「**London hotel**」と入力し、**[Search]** をクリックします。
4. 検索結果を確認します。 これらには、ファイル名 (ファイル URL へのハイパーリンク付き)、検索語 (*London* および *hotel*) が強調されたファイル コンテンツの抽出、およびインデックス フィールドからのファイルの他の属性が含まれます。
5. 結果ページには、結果を絞り込むことができるいくつかのユーザーインターフェイス要素が含まれていることに注意してください。 これには以下が含まれます。
    - **metadata_author** フィールドのファセット値に基づく *フィルター*。 これは、*ファセット可能* フィールドを使用して、ユーザー インターフェイスで潜在的なフィルター値として表示できる離散値の小さなセットを含む *ファセット* フィールドのリストを返す方法を示しています。
    - 指定されたフィールドと並べ替え方向 (昇順または降順) に基づいて結果を *並べ替える* 機能。 既定の順序は *関連性* に基づいており、インデックス フィールドの検索用語の頻度と重要性を評価する *スコアリング プロファイル* に基づいて **search.score()** 値として計算されます。
6. **[レビュー担当者]** フィルターと **[Positive to negative]\(肯定的から否定的\)** 並べ替えオプションを選択し、 **[結果の絞り込み]** を選択します。
7. 結果がレビューのみを含むようにフィルタリングされ、センチメント ラベルで並べ替えられていることを確認します。
8. **[検索]** ボックスに、「**quiet hotel in New York**」の新しい検索を入力して、結果を確認します。
9. 次の検索用語を試してください。
    - **Tower of London** (この用語が一部のドキュメントで *キー フレーズ* として識別されていることに注意してください)。
    - **skyscraper** (この単語はどのドキュメントの実際のコンテンツにも表示されませんが、一部のドキュメントの画像用に生成された *画像のキャプション* と *画像タグ* に含まれていることに注意してください)。
    - **Mojave desert** (この用語が一部のドキュメントで *場所* として識別されていることに注意してください)。
10. Margie's Travel の Web サイトが含まれているブラウザー タブを閉じ、Visual Studio Code に戻ります。 次に、**margies-travel** フォルダー (dotnet または flask アプリケーションが実行されている) の Python ターミナルで、Ctrl+C を入力してアプリを停止します。

## <a name="more-information"></a>詳細情報

Azure Cognitive Search の詳細については、[Azure Cognitive Search のドキュメント](https://docs.microsoft.com/azure/search/search-what-is-azure-search)を参照してください。
