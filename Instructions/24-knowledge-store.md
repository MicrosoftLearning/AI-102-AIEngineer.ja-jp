---
lab:
  title: Azure Cognitive Search 用ナレッジ ストアの作成
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: 8d7008ee592d4a76f24ca26a4b192c7dd5cbe9ea
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819338"
---
# <a name="create-a-knowledge-store-with-azure-cognitive-search"></a>Azure Cognitive Search 用ナレッジ ストアの作成

Azure Cognitive Search は、コグニティブ スキルの強化パイプラインを使用して、ドキュメントから AI で生成されたフィールドを抽出し、検索インデックスに含めます。 インデックスはインデックス作成プロセスの主な出力と見なせますが、それに含まれるエンリッチされたデータも他の方法で役立てることができます。 次に例を示します。

- 基本的にインデックスは、それぞれがインデックス付きレコードを表す JSON オブジェクトのコレクションであるため、オブジェクトを JSON ファイルとしてエクスポートし、Azure Data Factory などのツールによるデータ オーケストレーション プロセスへの統合に使用するのに役立つ可能性があります。
- Microsoft Power BI などのツールで分析やレポートを行うために、インデックス レコードをテーブルのリレーショナル スキーマに正規化することもできます。
- インデックス作成プロセスでは埋め込み画像をドキュメントから抽出するので、それらの画像をファイルとして保存することもできます。

この演習では、パンフレットやホテルのレビューの情報を使用して顧客が旅行を計画できるようにする架空の旅行代理店である *Margie's Travel* のナレッジ ストアを実装します。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

**AI-102-AIEngineer** コード リポジトリをこのラボの作業をしている環境に既にクローンしている場合は、Visual Studio Code で開きます。それ以外の場合は、次の手順に従って今すぐクローンしてください。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、 **[今はしない]** を選択します。

## <a name="create-azure-resources"></a>Azure リソースを作成する

> **注**: 以前に「**[Azure Cognitive Search ソリューションを作成する](22-azure-search.md)**」の演習を完了し、サブスクリプションにこれらの Azure リソースがまだある場合は、このセクションをスキップして、「**検索ソリューションの作成**」セクションから開始できます。 それ以外の場合は、以下の手順に従って、必要な Azure リソースをプロビジョニングします。

1. Web ブラウザーで Azure portal (`https://portal.azure.com`) を開き、自分の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. サブスクリプションの **リソース グループ** を表示します。
3. リソース グループが提供されている制限付きサブスクリプションを使用している場合は、リソース グループを選択してそのプロパティを表示します。 それ以外の場合は、選択した名前で新しいリソース グループを作成し、作成されたらそのグループに移動します。
4. リソース グループの **[概要]** ページで、**サブスクリプション ID** と **場所** を記録します。 これらの値は、後続の手順でリソース グループの名前とともに必要になります。
5. Visual Studio Code で、**24-knowledge-store** フォルダーを展開し、**setup.cmd** を選択します。 このバッチ スクリプトを使用して、必要な Azure リソースを作成するために必要な Azure コマンド ライン インターフェイス (CLI) コマンドを実行します。
6. **24-knowledge-store** フォルダーを右クリックし、 **[統合ターミナルで開く]** を選択します。
7. ターミナル ペインで、次のコマンドを入力して、お使いの Azure サブスクリプションへの認証された接続を確立します。

    ```
    az login --output none
    ```

8. メッセージ表示されたら、Azure サブスクリプションにサインインします。 その後、Visual Studio Code に戻り、サインイン プロセスが完了するまで待ちます。
9. 次のコマンドを実行して、Azure の場所を一覧表示します。

    ```
    az account list-locations -o table
    ```

10. 出力で、リソース グループの場所に対応する **Name** の値を見つけます (たとえば、*米国東部* の場合、対応する名前は *eastus* です)。
11. **setup.cmd** スクリプトで、**subscription_id**、**resource_group**、**location** 変数の宣言を、サブスクリプション ID、リソース グループ名、場所名の適切な値で変更します。 次に、変更を保存します。
12. **24-knowledge-store** フォルダーのターミナルで、次のコマンドを入力してスクリプトを実行します。

    ```
    setup
    ```
    > **注**: Search CLI モジュールはプレビュー中であり、 *- 実行中 ..* プロセスでスタックする可能性が存在して います。 これが 2 分以上続く場合は、Ctrl + C キーを押して長時間実行中の操作をキャンセルし、スクリプトを終了するかどうかを尋ねられたら **N** を選択します。 その後、正常に完了するはずです。
    >
    > スクリプトが失敗した場合は、正しい変数名で保存したことを確認して、再試行してください。

13. スクリプトが完了したら、表示された出力を確認し、ご自分の Azure リソースに関する次の情報をメモします (これらの値は後で必要になります)。
    - ストレージ アカウント名
    - ストレージ接続文字列
    - Cognitive Services アカウント
    - Cognitive Services キー
    - Search Service エンドポイント
    - Search Service の管理者キー
    - Search Service のクエリ キー

14. Azure portal で、リソースグループを更新し、Azure Storage アカウント、Azure Cognitive Services リソース、および Azure Cognitive Search リソースが含まれていることを確認します。

## <a name="create-a-search-solution"></a>検索ソリューションの作成

必要な Azure リソースが揃ったので、次のコンポーネントで構成される検索ソリューションを作成できます。

- Azure ストレージ コンテナー内のドキュメントを参照する **データ ソース**。
- ドキュメントから AI で生成されたフィールドを抽出するスキルの強化パイプラインを定義する **スキルセット**。 スキルセットは、*ナレッジ ストア* で生成される *予測* も定義します。
- 検索可能なドキュメント レコードのセットを定義する **インデックス**。
- データ ソースからドキュメントを抽出し、スキル セットを適用して、インデックスにデータを入力する **インデクサー**。 インデックス作成のプロセスは、ナレッジ ストアのスキル セットで定義された予測も保持します。

この演習では、Azure Cognitive Search REST インターフェイスを使用して、JSON リクエストを送信することでこれらのコンポーネントを作成します。

### <a name="prepare-json-for-rest-operations"></a>REST 操作用の JSON の準備

REST インターフェイスを使用して、Azure CognitiveSearch コンポーネントの JSON 定義を送信します。

1. Visual Studio Code の **24-knowledge-store** フォルダーで、**create-search** フォルダーを展開し、**data_source.json** を選択します。 このファイルには、**margies-knowledge-data** という名前のデータソースの JSON 定義が含まれています。
2. **YOUR_CONNECTION_STRING** プレースホルダーを Azure ストレージ アカウントの接続文字列に置き換えます。これは次のようになります。

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *接続文字列は、Azure portal のストレージ アカウントの **[アクセス キー]** ページにあります。*

3. 更新された JSON ファイルを保存して閉じます。
4. **create-search** フォルダーで、**skillset.json** を開きます。 このファイルには、**margies-knowledge-skillset** という名前のスキルセットの JSON 定義が含まれています。
5. スキルセット定義の先頭にある **cognitiveServices** 要素で、**YOUR_COGNITIVE_SERVICES_KEY** プレースホルダーを Cognitive Services リソースのいずれかのキーに置き換えます。

    *キーは、Azure portal の Cognitive Services リソースの **[キーとエンドポイント]** ページにあります。*

6. スキルセット内のスキルのコレクションの最後に、**define-projection** という名前の **Microsoft.Skills.Util.ShaperSkill** スキルを見つけます。 このスキルは、インデクサーによって処理される各ドキュメントのナレッジ ストアにパイプラインが保持されるプロジェクションに使用される、強化されたデータの JSON 構造を定義します。
7. スキル セット ファイルの下部に、ナレッジ ストア定義が含まれていることを確認します。**ナレッジ ストア** 定義には、ナレッジ ストアが作成される Azure Storage アカウントの接続文字列と、**プロジェクション** のコレクションが含まれています。 このスキルセットには、次の 3 つの "*プロジェクション グループ*" が含まれます。
    - スキルセットに含まれる Shaper スキルの **knowledge_projection** 出力に基づく "*オブジェクト*" プロジェクションを含むグループ。
    - ドキュメントから抽出された画像データの **normalized_images** コレクションに基づく "*ファイル*" プロジェクションを含むグループ。
    - 次の "*テーブル*" プロジェクションを含むグループ。
        - **KeyPhrases**: 自動的に生成されたキー列と、Shaper スキルの **knowledge_projection/key_phrases/** コレクション出力にマップされた **keyPhrase** 列が含まれています。
        - **Locations**: 自動的に生成されたキー列と、Shaper スキルの **knowledge_projection/key_phrases/** コレクション出力にマップされた **location** 列が含まれています。
        - **ImageTags**: 自動的に生成されたキー列と、Shaper スキルの **knowledge_projection/image_tags/** コレクション出力にマップされた **tag** 列が含まれています。
        - **Docs**: 自動的に生成されたキー列と、テーブルにまだ割り当てられていない Shaper スキルのすべての **knowledge_projection** 出力値が含まれています。
8. **storageConnectionString** 値の **YOUR_CONNECTION_STRING** プレースホルダーを、ストレージ アカウントの接続文字列に置き換えます。
9. 更新された JSON ファイルを保存して閉じます。
10. **create-search** フォルダーで、**index.json** を開きます。 このファイルには、**margies-knowledge-index** という名前のインデックスの JSON 定義が含まれています。
11. インデックスの JSON を確認し、変更を加えずにファイルを閉じます。
12. **create-search** フォルダーで、**indexer.json** を開きます。 このファイルには、**margies-knowledge-indexer** という名前のインデクサーの JSON 定義が含まれています。
13. インデクサーの JSON を確認し、変更を加えずにファイルを閉じます。

### <a name="submit-rest-requests"></a>REST リクエストの送信

検索ソリューションコンポーネントを定義する JSON オブジェクトを準備したので、JSON ドキュメントを REST インターフェイスに送信して作成できます。

1. **create-search** フォルダーで、**create-search.cmd** を開きます。 このバッチ スクリプトは、cURL ユーティリティを使用して、Azure Cognitive Search リソースの REST インターフェイスに JSON 定義を送信します。
2. **YOUR_SEARCH_URL** 変数と **YOUR_ADMIN_KEY** 変数のプレースホルダーを、Azure Cognitive Search リソースの **Url** と **管理キー** の 1 つに置き換えます。

    *これらの値は、Azure portal の Azure Cognitive Search リソースの **[概要]** ページと **[キー]** ページにあります。*

3. 更新したバッチファイルを保存します。
4. **create-search** フォルダーを右クリックし、 **[Open in Integrated Terminal]\(統合ターミナルで開く\)** を選択します。
5. **create-search** フォルダーのターミナル ペインで、次のコマンドを入力してバッチ スクリプトを実行します。

    ```
    create-search
    ```

6. スクリプトが完了したら、Azure portal の Azure Cognitive Search リソースのページで、 **[インデクサー]** ページを選択し、インデックス作成プロセスが完了するのを待ちます。

    ***[更新]** を選択して、インデックス作成操作の進行状況を追跡できます。完了するまでに 1 分ほどかかる場合があります。*

    > **ヒント**: スクリプトが失敗した場合は、**data_source.json** ファイルと **skillset.json** ファイル、および **create-search.cmd** ファイルに追加したプレースホルダーを確認してください。 間違いを修正した後、スクリプトを再実行する前に、Azure portal のユーザー インターフェイスを使用して、検索リソースで作成されたコンポーネントを削除する必要がある場合があります。

## <a name="view-the-knowledge-store"></a>ナレッジストアの表示

スキルセットを使用してナレッジ ストアを作成するインデクサーを実行すると、そのインデックス作成プロセスによって抽出されてエンリッチされたデータがナレッジ ストアのプロジェクションに保持されます。

### <a name="view-object-projections"></a>オブジェクト プロジェクションを表示する

Margie's Travel スキルセットで定義されている "*オブジェクト*" プロジェクションは、各インデックス付きドキュメントの JSON ファイルで構成されています。 これらのファイルは、スキルセットの定義で指定された Azure ストレージ アカウントの BLOB コンテナーに格納されます。

1. Azure portal で、以前に作成した Azure Storage アカウントを表示します。
2. **[ストレージ エクスプローラー]** タブ (左側のペイン) を選択して、Azure portal のストレージ エクスプローラー インターフェイスでストレージ アカウントを表示します。
2. **[BLOB コンテナー]** を展開して、そのストレージ アカウント内のコンテナーを表示します。 ソース データが格納されている **margies** コンテナーの他に、**margies-images** と **margies-knowledge** という 2 つの新しいコンテナーがあるはずです。 これらは、インデックス作成プロセスによって作成されたものです。
3. **margies-knowledge** コンテナーを選択します。 これには、各インデックス付きドキュメントのフォルダーが含まれています。
4. いずれかのフォルダーを開き、そこに含まれている **knowledge-projection.json** ファイルをダウンロードして開きます。 次に示すように、各 JSON ファイルには、スキルセットによって抽出されてエンリッチされたデータなどの、インデックス付きドキュメントの表現が含まれています。

```
{
    "file_id":"abcd1234....",
    "file_name":"Margies Travel Company Info.pdf",
    "url":"https://store....blob.core.windows.net/margies/...pdf",
    "language":"en",
    "sentiment":0.83164644241333008,
    "key_phrases":[
        "Margie’s Travel",
        "Margie's Travel",
        "best travel experts",
        "world-leading travel agency",
        "international reach"
        ],
    "locations":[
        "Dubai",
        "Las Vegas",
        "London",
        "New York",
        "San Francisco"
        ],
    "image_tags":[
        "outdoor",
        "tree",
        "plant",
        "palm"
        ]
}
```

このような "*オブジェクト*" プロジェクションの作成機能を使用することにより、エンタープライズ データ分析ソリューションに組み込むことのできるエンリッチされたデータ オブジェクトを生成することができます。これは、その JSON ファイルを Azure Data Factory パイプラインに取り込んでさらに処理を行ったり、データ ウェアハウスに読み込んだりすることで行えます。

### <a name="view-file-projections"></a>ファイル プロジェクションを表示する

スキルセットで定義されている "*ファイル*" プロジェクションでは、インデックス作成プロセス中にドキュメントから抽出された各画像の JPEG ファイルが作成されます。

1. Azure portal のストレージ エクスプローラー インターフェイスで、**margies-images** BLOB コンテナーを選択します。 このコンテナーには、画像を含む各ドキュメントのフォルダーが含まれています。
2. そのフォルダーのいずれかを開き、内容を表示します。各フォルダーに、少なくとも 1 つの \*.jpg ファイルが含まれています。
3. 画像ファイルのいずれかを開き、ドキュメントから抽出された画像が含まれていることを確認します。

このような "*ファイル*" プロジェクションの生成機能を使用することにより、インデックス作成で大量のドキュメントから埋め込み画像を効率的に抽出できます。

### <a name="view-table-projections"></a>テーブル プロジェクションを表示する

スキルセットで定義されている "*テーブル*" プロジェクションは、エンリッチされたデータのリレーショナル スキーマを形成します。

1. Azure portal のストレージ エクスプローラー インターフェイスで、**[テーブル]** を展開します。
2. **Docs** テーブルを選択して、その列を表示します。 この列には、標準的な Azure Storage テーブル列がいくつか含まれています。これらを非表示にするには、**[列のオプション]** を変更して、次の列のみを選択します。
    - **document_id** (インデックス作成プロセスによって自動的に生成されるキー列)
    - **file_id** (エンコードされたファイルの URL)
    - **file_name** (ドキュメントのメタデータから抽出されたファイル名)
    - **language** (ドキュメントの記述言語)
    - **sentiment** そのドキュメントに対して計算されたセンチメント スコア。
    - **url** Azure Storage のドキュメント BLOB の URL。
3. インデックス作成プロセスによって作成された他のテーブルを表示します。
    - **ImageTags** (個々の画像タグごとに、そのタグが使用されているドキュメントの **document_id** が入った行が 1 行含まれます)。
    - **KeyPhrases** (個々のキー フレーズごとに、そのフレーズが使用されているドキュメントの **document_id** が入った行が 1 行含まれます)。
    - **Locations** (個々の場所ごとに、その場所が使用されているドキュメントの **document_id** が入った行が 1 行含まれます)。

*テーブル* プロジェクションを作成する機能により、リレーショナル スキーマをクエリする分析およびレポート ソリューションを構築できます。たとえば、Microsoft Power BI を使用します。 自動的に生成されたキー列は、クエリでテーブルを結合するために使用できます。たとえば、特定のドキュメントに記載されているすべての場所を返すことができます。

## <a name="more-information"></a>詳細情報

Azure Cognitive Search を使用したナレッジ ストアの作成の詳細については、[Azure Cognitive Search のドキュメント](https://docs.microsoft.com/azure/search/knowledge-store-concept-intro)を参照してください。
