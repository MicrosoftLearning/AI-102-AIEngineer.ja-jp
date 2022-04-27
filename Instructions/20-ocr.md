---
lab:
  title: 画像内のテキストの読み取り
  module: Module 11 - Reading Text in Images and Documents
ms.openlocfilehash: 0dc45d60e307769ebfde165201b97c4a3ff49675
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819369"
---
# <a name="read-text-in-images"></a>画像内のテキストの読み取り

光学式文字認識 (OCR) は、画像およびドキュメント内のテキストの読み取りを処理する Computer Vision のサブセットです。 **Computer Vision** サービスにより、テキストを読み取るための 2 つの API が提供されます。これについては、この演習で説明します。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

まだ行っていない場合は、このコースのコード リポジトリを複製する必要があります。

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
5. リソースがデプロイされたら、そこに移動して、その **キーとエンドポイント** のページを表示します。 次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Computer Vision SDKを使用する準備をする

この演習では、Computer VisionSDK を使用してテキストを読み取る部分的に実装されたクライアント アプリケーションを完成させます。

> **注**:**C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **エクスプローラー** ペインで、**20-ocr** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **read-text** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Computer Vision SDK パッケージをインストールします。

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```

3. **read-text** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの **エンドポイント** と認証 **キー** を反映します。 変更を保存します。
4. **read-text** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることにご注意ください。

    - **C#** : Program.cs
    - **Python**: read-text.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision SDK を使用するために必要な名前空間をインポートします

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
from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes
from msrest.authentication import CognitiveServicesCredentials
```

5. クライアント アプリケーションのコードファイルで **Main** 関数に、構成設定を読み込むためのコードが提供されていることにご注意ください。 次に、コメント「**Authenticate Computer Vision client**」を見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision クライアント オブジェクトを作成および認証します

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
    
## <a name="use-the-ocr-api"></a>OCR API を使用する

**OCR** APIは、 *.jpg*、 *.png*、 *.gif*、 *.bmp* 形式の画像で少量から中量の印刷テキストを読み取るために最適化された光学式文字認識 API です。 幅広い言語をサポートし、画像内のテキストを読み取ることに加えて、各テキスト領域の方向を決定し、画像に対するテキストの回転角に関する情報を返すことができます

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **1** を選択した場合に実行されるコードを調べます。 このコードにより **GetTextOcr** 関数が呼び出され、パスが画像ファイルに渡されます。
2. **read-text/images** フォルダーで、**Lincoln.jpg** を開いて、コードによって処理される画像を表示します。
3. コード ファイルに戻り、**GetTextOcr** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します。

**C#**

```C#
// Use OCR API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var ocrResults = await cvClient.RecognizePrintedTextInStreamAsync(detectOrientation:false, image:imageData);

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Magenta, 3);

    foreach(var region in ocrResults.Regions)
    {
        foreach(var line in region.Lines)
        {
            // Show the position of the line of text
            int[] dims = line.BoundingBox.Split(",").Select(int.Parse).ToArray();
            Rectangle rect = new Rectangle(dims[0], dims[1], dims[2], dims[3]);
            graphics.DrawRectangle(pen, rect);

            // Read the words in the line of text
            string lineText = "";
            foreach(var word in line.Words)
            {
                lineText += word.Text + " ";
            }
            Console.WriteLine(lineText.Trim());
        }
    }

    // Save the image with the text locations highlighted
    String output_file = "ocr_results.jpg";
    image.Save(output_file);
    Console.WriteLine("Results saved in " + output_file);
}
```

**Python**

```Python
# Use OCR API to read text in image
with open(image_file, mode="rb") as image_data:
    ocr_results = cv_client.recognize_printed_text_in_stream(image_data)

# Prepare image for drawing
fig = plt.figure(figsize=(7, 7))
img = Image.open(image_file)
draw = ImageDraw.Draw(img)

# Process the text line by line
for region in ocr_results.regions:
    for line in region.lines:

        # Show the position of the line of text
        l,t,w,h = list(map(int, line.bounding_box.split(',')))
        draw.rectangle(((l,t), (l+w, t+h)), outline='magenta', width=5)

        # Read the words in the line of text
        line_text = ''
        for word in line.words:
            line_text += word.text + ' '
        print(line_text.rstrip())

# Save the image with the text locations highlighted
plt.axis('off')
plt.imshow(img)
outputfile = 'ocr_results.jpg'
fig.savefig(outputfile)
print('Results saved in', outputfile)
```

4. **GetTextOcr** 関数に追加したコードを調べます。 画像ファイルから印刷されたテキストの領域を検出し、領域ごとにテキストの行を抽出して、画像上のその位置を強調表示します。 次に、各行の単語を抽出して印刷します。
5. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

*C# 出力に、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これは無視してもかまいません。*

**Python**

```
python read-text.py
```

6. プロンプトが表示されたら、**1** を入力して、画像から抽出されたテキストである出力を確認します。
7. コード ファイルと同じフォルダーに生成された **ocr_results.jpg** ファイルを表示して、画像内の注釈付きのテキスト行を確認します。

## <a name="use-the-read-api"></a>Read API を使用する

**Read** API では、OCR API よりも新しいテキスト認識モデルが使用されており、多くのテキストを含む大きな画像に対してより優れたパフォーマンスを発揮します。 また、 *.pdf* ファイルからのテキスト抽出をサポートし、印刷されたテキスト (複数の言語) と手書きのテキスト (英語) の両方を認識できます。

**Read** API は、テキスト認識を開始する要求が送信される非同期操作モデルを使用します。その後、リクエストから返された操作 ID を使用して、進行状況を確認し、結果を取得できます。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **2** を選択した場合に実行されるコードを調べます。 このコードは **GetTextRead** 関数を呼び出し、パスを PDF ドキュメント ファイルに渡します。
2. **read-text/images** フォルダーで、**Rome.pdf** を右クリックし、 **[ファイルエクスプローラーで表示]** を選択します。 次に、ファイル エクスプローラーで、PDF ファイルを開いて表示します。
3. Visual Studio Code のコードファイルに戻り、**GetTextRead** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します。

**C#**

```C#
// Use Read API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var readOp = await cvClient.ReadInStreamAsync(imageData);

    // Get the async operation ID so we can check for the results
    string operationLocation = readOp.OperationLocation;
    string operationId = operationLocation.Substring(operationLocation.Length - 36);

    // Wait for the asynchronous operation to complete
    ReadOperationResult results;
    do
    {
        Thread.Sleep(1000);
        results = await cvClient.GetReadResultAsync(Guid.Parse(operationId));
    }
    while ((results.Status == OperationStatusCodes.Running ||
            results.Status == OperationStatusCodes.NotStarted));

    // If the operation was successfuly, process the text line by line
    if (results.Status == OperationStatusCodes.Succeeded)
    {
        var textUrlFileResults = results.AnalyzeResult.ReadResults;
        foreach (ReadResult page in textUrlFileResults)
        {
            foreach (Line line in page.Lines)
            {
                Console.WriteLine(line.Text);
            }
        }
    }
}  
```

**Python**

```Python
# Use Read API to read text in image
with open(image_file, mode="rb") as image_data:
    read_op = cv_client.read_in_stream(image_data, raw=True)

    # Get the async operation ID so we can check for the results
    operation_location = read_op.headers["Operation-Location"]
    operation_id = operation_location.split("/")[-1]

    # Wait for the asynchronous operation to complete
    while True:
        read_results = cv_client.get_read_result(operation_id)
        if read_results.status not in [OperationStatusCodes.running, OperationStatusCodes.not_started]:
            break
        time.sleep(1)

    # If the operation was successfuly, process the text line by line
    if read_results.status == OperationStatusCodes.succeeded:
        for page in read_results.analyze_result.read_results:
            for line in page.lines:
                print(line.text)
```
    
4. **GetTextRead** 関数に追加したコードを調べます。 読み取り操作の要求を送信し、操作が完了するまでステータスを繰り返しチェックします。 成功した場合、コードは各ページを繰り返し処理し、次に各行を繰り返し処理して結果を処理します。
5. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

6. ダイアログが表示されたら、**2** を入力し、ドキュメントから抽出されたテキストである出力を確認します。

## <a name="read-handwritten-text"></a>手書きテキストの読み取り

**Read** API は、印刷されたテキスト以外にも、英語の手書きのテキストを抽出できます。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **3** を選択した場合に実行されるコードを調べます。 このコードは **GetTextRead** 関数を呼び出し、パスを画像ファイルに渡します。
2. **read-text/images** フォルダーで、**Note.jpg** を開いて、コードによって処理される画像を表示します。
3. **read-text** フォルダーの統合ターミナルで、次のコマンドを入力してプログラムを実行します。

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

4. ダイアログが表示されたら、**3** を入力し、ドキュメントから抽出されたテキストである出力を確認します。

## <a name="more-information"></a>詳細情報

**Computer Vision** サービスを使用してテキストを読み取る方法の詳細については、[Computer Vision のドキュメント](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text)を参照してください。
