---
lab:
  title: Computer Vision を使用する画像の分析
  module: Module 8 - Getting Started with Computer Vision
ms.openlocfilehash: f2ee18ff682d53e9fd554749ed2b9cbaa9b03611
ms.sourcegitcommit: 7191e53bc33cda92e710d957dde4478ee2496660
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/09/2022
ms.locfileid: "147041668"
---
# <a name="analyze-images-with-computer-vision"></a>Computer Vision を使用する画像の分析

Computer Vision は、ソフトウェアシステムが画像を分析することで視覚入力を解釈できるようにする人工知能機能です。 Microsoft Azure では、**Computer Vision** Cognitive Services は、キャプションとタグを提案する画像の分析、一般的なオブジェクト、ランドマーク、有名人、ブランドの検出、アダルトコンテンツの存在など、一般的な Computer Vision 用の構築済みモデルを提供します。 Computer Vision サービスを使用して、画像の色と形式を分析し、"スマートにトリミングされた" サムネイル画像を生成することもできます。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

このラボで作業している環境に **AI-102-AIEngineer** コードのリポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

1. Visual Studio Code を起動します。
2. パレットを開き (Shift + Ctrl + P)、**Git: Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、**[今はしない]** を選択します。

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
5. リソースがデプロイされたら、そこに移動して、その **キーとエンドポイント** のページを表示します。 次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Computer Vision SDKを使用する準備をする

この演習では、Computer VisionSDKを使用して画像を分析する部分的に実装されたクライアントアプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **エクスプローラー** ペインで、**15-computer-vision** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **image-analysis** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Computer Vision SDK パッケージをインストールします。

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```
    
3. **image-analysis** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください。
    - **C#**: appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの **エンドポイント** と認証 **キー** を反映します。 変更を保存します。
4. **image-analysis** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#**: Program.cs
    - **Python**: image-analysis.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、"**Import namespaces**" というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision SDK を使用するために必要な名前空間をインポートします

**C#**

```C#
// import namespaces
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```

**Python**

```Python
# import namespaces
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
from msrest.authentication import CognitiveServicesCredentials
```
    
## <a name="view-the-images-you-will-analyze"></a>ビューの画像は、あなたが分析します

この演習では、Computer Vision サービスを使用して複数の画像を分析します。

1. Visual Studio Code で、**image-analysis** フォルダーとそれに含まれる **images** フォルダーを展開します。
2. 各画像ファイルを順番に選択して表示し、Visual Studio Code で表示します。

## <a name="analyze-an-image-to-suggest-a-caption"></a>画像を分析してキャプションを提案する

これで、SDK を使用して Computer Vision サービスを呼び出す準備が整いました。

1. クライアント アプリケーションのコード ファイル (**Program.cs** または **image-analysis.py**) の **Main** 関数で、構成設定をロードするためのコードが提供されていることに注意してください。 次に、コメント "**Authenticate Computer Vision client**" を見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision クライアント オブジェクトを作成および認証します

**C#**

```C#
// Authenticate Computer Vision client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Authenticate Computer Vision client
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```

2. **Main** 関数で、追加したコードの下で、コードが画像ファイルへのパスを指定し、画像パスを他の 2 つの関数 (**AnalyzeImage** と **GetThumbnail**) に渡すことに注意してください。 これらの関数はまだ完全には実装されていません。

3. **AnalyzeImage** 関数のコメント "**Specify features to be retrieved**" の下に、次のコードを追加します。

**C#**

```C#
// Specify features to be retrieved
List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
{
    VisualFeatureTypes.Description,
    VisualFeatureTypes.Tags,
    VisualFeatureTypes.Categories,
    VisualFeatureTypes.Brands,
    VisualFeatureTypes.Objects,
    VisualFeatureTypes.Adult
};
```

**Python**

```Python
# Specify features to be retrieved
features = [VisualFeatureTypes.description,
            VisualFeatureTypes.tags,
            VisualFeatureTypes.categories,
            VisualFeatureTypes.brands,
            VisualFeatureTypes.objects,
            VisualFeatureTypes.adult]
```
    
4. **AnalyzeImage** 関数のコメント "**Get image analysis**" の下に、次のコードを追加します (後でコードを追加する場所を示すコメントを含む)。

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // get image captions
    foreach (var caption in analysis.Description.Captions)
    {
        Console.WriteLine($"Description: {caption.Text} (confidence: {caption.Confidence.ToString("P")})");
    }

    // Get image tags


    // Get image categories


    // Get brands in the image


    // Get objects in the image


    // Get moderation ratings
    

}            
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

# Get image description
for caption in analysis.description.captions:
    print("Description: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get image categories 


# Get brands in the image


# Get objects in the image


# Get moderation ratings

```
    
5. 変更を保存して、**image-analysis** フォルダーの統合ターミナルに戻り、引数 **images/street.jpg** でプログラムを実行するには、次のコマンドを入力します。

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. 出力を観察します。これには、**street.jpg** 画像の推奨キャプションが含まれている必要があります。
7. 今度は引数 **images/building.jpg** を使用してプログラムを再度実行し、**building.jpg** 画像に対して生成されるキャプションを確認します。
8. 前の手順を繰り返して、**images/person.jpg** ファイルのキャプションを生成します。

## <a name="get-suggested-tags-for-an-image"></a>画像の推奨タグを取得

画像の内容に関する手がかりを提供する関連 *タグ* を特定すると役立つ場合があります。

1. **AnalyzeImage** 関数のコメント "**Get image tags**" の下に、次のコードを追加します。

**C#**

```C
// Get image tags
if (analysis.Tags.Count > 0)
{
    Console.WriteLine("Tags:");
    foreach (var tag in analysis.Tags)
    {
        Console.WriteLine($" -{tag.Name} (confidence: {tag.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get image tags
if (len(analysis.tags) > 0):
    print("Tags: ")
    for tag in analysis.tags:
        print(" -'{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、画像のキャプションに加えて、提案されたタグのリストが表示されることを確認します。

## <a name="get-image-categories"></a>Get image categories

Computer Vision サービスは、画像の *"カテゴリ"* を提案でき、各カテゴリ内で有名なランドマークを識別できます。

1. **AnalyzeImage** 関数のコメント "**Get image categories**" の下に、次のコードを追加します。

**C#**

```C
// Get image categories
List<LandmarksModel> landmarks = new List<LandmarksModel> {};
Console.WriteLine("Categories:");
foreach (var category in analysis.Categories)
{
    // Print the category
    Console.WriteLine($" -{category.Name} (confidence: {category.Score.ToString("P")})");

    // Get landmarks in this category
    if (category.Detail?.Landmarks != null)
    {
        foreach (LandmarksModel landmark in category.Detail.Landmarks)
        {
            if (!landmarks.Any(item => item.Name == landmark.Name))
            {
                landmarks.Add(landmark);
            }
        }
    }
}

// If there were landmarks, list them
if (landmarks.Count > 0)
{
    Console.WriteLine("Landmarks:");
    foreach(LandmarksModel landmark in landmarks)
    {
        Console.WriteLine($" -{landmark.Name} (confidence: {landmark.Confidence.ToString("P")})");
    }
}

```

**Python**

```Python
# Get image categories
if (len(analysis.categories) > 0):
    print("Categories:")
    landmarks = []
    for category in analysis.categories:
        # Print the category
        print(" -'{}' (confidence: {:.2f}%)".format(category.name, category.score * 100))
        if category.detail:
            # Get landmarks in this category
            if category.detail.landmarks:
                for landmark in category.detail.landmarks:
                    if landmark not in landmarks:
                        landmarks.append(landmark)

    # If there were landmarks, list them
    if len(landmarks) > 0:
        print("Landmarks:")
        for landmark in landmarks:
            print(" -'{}' (confidence: {:.2f}%)".format(landmark.name, landmark.confidence * 100))

```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、画像のキャプションとタグに加えて、推奨されるカテゴリのリストが、認識されているランドマーク (具体的には **building.jpg** の画像) とともに表示されることを確認します。

## <a name="get-brands-in-an-image"></a>画像でブランドを取得する

一部のブランドは、ブランド名が表示されていなくても、ロゴから視覚的に認識できます。 Computer Vision サービスは、何千もの有名なブランドを特定するためにトレーニングされています。

1. **AnalyzeImage** 関数で、コメント "**Get brands in the image**" 下に、次のコードを追加します。

**C#**

```C
// Get brands in the image
if (analysis.Brands.Count > 0)
{
    Console.WriteLine("Brands:");
    foreach (var brand in analysis.Brands)
    {
        Console.WriteLine($" -{brand.Name} (confidence: {brand.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get brands in the image
if (len(analysis.brands) > 0):
    print("Brands: ")
    for brand in analysis.brands:
        print(" -'{}' (confidence: {:.2f}%)".format(brand.name, brand.confidence * 100))
```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、識別されたブランド (具体的には、**person.jpg** 画像) を確認します。

## <a name="detect-and-locate-objects-in-an-image"></a>画像内のオブジェクトを検出して特定する

*物体検出* は、画像内の個々のオブジェクトが識別され、その場所が境界ボックスで示される特定の形式の Computer Vision です。

1. **AnalyzeImage** 関数のコメント "**Get objects in the image**" の下に、次のコードを追加します。

**C#**

```C
// Get objects in the image
if (analysis.Objects.Count > 0)
{
    Console.WriteLine("Objects in image:");

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.Black);

    foreach (var detectedObject in analysis.Objects)
    {
        // Print object name
        Console.WriteLine($" -{detectedObject.ObjectProperty} (confidence: {detectedObject.Confidence.ToString("P")})");

        // Draw object bounding box
        var r = detectedObject.Rectangle;
        Rectangle rect = new Rectangle(r.X, r.Y, r.W, r.H);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.ObjectProperty,font,brush,r.X, r.Y);

    }
    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file);   
}
```

**Python**

```Python
# Get objects in the image
if len(analysis.objects) > 0:
    print("Objects in image:")

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 8))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    color = 'cyan'
    for detected_object in analysis.objects:
        # Print object name
        print(" -{} (confidence: {:.2f}%)".format(detected_object.object_property, detected_object.confidence * 100))
        
        # Draw object bounding box
        r = detected_object.rectangle
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.object_property,(r.x, r.y), backgroundcolor=color)
    # Save annotated image
    plt.imshow(image)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、検出されるオブジェクトを監視します。 実行するたびに、コードファイルと同じフォルダーに生成された **objects.jpg** ファイルを表示して、注釈付きのオブジェクトを確認します。

## <a name="get-moderation-ratings-for-an-image"></a>画像のモデレート評価を取得する

一部の画像はすべての視聴者に適しているとは限らないため、成人向けまたは暴力的な性質の画像を特定するためにモデレートを適用する必要がある場合があります。

1. **AnalyzeImage** 関数のコメント "**Get moderation ratings**" の下に、次のコードを追加します。

**C#**

```C
// Get moderation ratings
string ratings = $"Ratings:\n -Adult: {analysis.Adult.IsAdultContent}\n -Racy: {analysis.Adult.IsRacyContent}\n -Gore: {analysis.Adult.IsGoryContent}";
Console.WriteLine(ratings);
```

**Python**

```Python
# Get moderation ratings
ratings = 'Ratings:\n -Adult: {}\n -Racy: {}\n -Gore: {}'.format(analysis.adult.is_adult_content,
                                                                    analysis.adult.is_racy_content,
                                                                    analysis.adult.is_gory_content)
print(ratings)
```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、各画像の評価を確認します。

> **注**: 前のタスクでは、単一の方法を使用して画像を分析し、コードを段階的に追加して結果を解析および表示しました。 SDK には、キャプションの提案、タグの識別、オブジェクトの検出などの個別のメソッドも用意されています。つまり、最も適切なメソッドを使用して必要な情報のみを返し、返す必要のあるデータ ペイロードのサイズを減らすことができます。 詳細については、[.NET SDK のドキュメント](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/computervision?view=azure-dotnet)または [Python SDK のドキュメント](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/computervision?view=azure-python)を参照してください。

## <a name="generate-a-thumbnail-image"></a>サムネイル画像の生成

場合によっては、*サムネイル* という名前の画像の小さいバージョンを作成し、それをトリミングして、新しい画像の寸法内に主要な視覚的主題を含める必要があります。

1. コード ファイルで、**GetThumbnail** 関数を見つけます。コメント "**Generate a thumbnail**" の下に、次のコードを追加します。

**C#**

```C
// Generate a thumbnail
using (var imageData = File.OpenRead(imageFile))
{
    // Get thumbnail data
    var thumbnailStream = await cvClient.GenerateThumbnailInStreamAsync(100, 100,imageData, true);

    // Save thumbnail image
    string thumbnailFileName = "thumbnail.png";
    using (Stream thumbnailFile = File.Create(thumbnailFileName))
    {
        thumbnailStream.CopyTo(thumbnailFile);
    }

    Console.WriteLine($"Thumbnail saved in {thumbnailFileName}");
}
```

**Python**

```Python
# Generate a thumbnail
with open(image_file, mode="rb") as image_data:
    # Get thumbnail data
    thumbnail_stream = cv_client.generate_thumbnail_in_stream(100, 100, image_data, True)

# Save thumbnail image
thumbnail_file_name = 'thumbnail.png'
with open(thumbnail_file_name, "wb") as thumbnail_file:
    for chunk in thumbnail_stream:
        thumbnail_file.write(chunk)

print('Thumbnail saved in.', thumbnail_file_name)
```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、各画像のコード ファイルと同じフォルダーに生成された **thumbnail.jpg** ファイルを開きます。

## <a name="more-information"></a>詳細情報

この演習では、Computer Vision サービスの画像分析および操作機能のいくつかについて説明しました。 このサービスには、テキストの読み取り、顔の検出、およびその他の Computer Vision タスクの機能も含まれています。

**Computer Vision** サービスの使用の詳細については、[Computer Vision のドキュメント](https://docs.microsoft.com/azure/cognitive-services/computer-vision/)を参照してください。
