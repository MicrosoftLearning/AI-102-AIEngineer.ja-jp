---
lab:
  title: 顔の検出、分析、認識
  module: Module 10 - Detecting, Analyzing, and Recognizing Faces
ms.openlocfilehash: b9565f41eb67b916278508c729860a3471a9e0bd
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819503"
---
# <a name="detect-analyze-and-recognize-faces"></a>顔の検出、分析、認識

人間の顔を検出、分析、認識する機能は、AI のコア機能です。 この演習では、画像内の顔を操作するために使用できる 2 つの Azure Cognitive Services である **Computer Vision** サービスと **Face** サービスについて説明します。

## <a name="clone-the-repository-for-this-course"></a>このコースのリポジトリを複製する

まだ行っていない場合は、このコースのコード リポジトリを複製する必要があります。

1. Visual Studio Code を起動します。
2. パレットを開き (Shift + Ctrl + P)、**Git: Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/AI-102-AIEngineer` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、 **[今はしない]** を選択します。

## <a name="provision-a-cognitive-services-resource"></a>Cognitive Services リソースをプロビジョニングする

サブスクリプションに **Cognitive Services** リソースがまだない場合は、プロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **[&#65291;リソースの作成]** ボタンを選択し、*Cognitive Services* を検索して、次の設定で **Cognitive Services** リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります - 提供されているものをお使いください)*
    - **[リージョン]**: 使用できるリージョンを選択します
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースがデプロイされたら、そこに移動して、その **キーとエンドポイント** のページを表示します。 次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Computer Vision SDKを使用する準備をする

この演習では、Computer Vision SDK を使用して画像内の顔を分析する部分的に実装されたクライアント アプリケーションを完成させます。

> **注**:**C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **エクスプローラー** ペインで、**19-face** フォルダーを参照し、言語の設定に応じて **C#** または **Python** フォルダーを展開します。
2. **computer-vision** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Computer Vision SDK パッケージをインストールします。

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-computervision==0.7.0
    ```
    
3. **computer-vision** フォルダーの内容を表示し、構成設定用のファイルが含まれていることを確認してください。
    - **C#** : appsettings.json
    - **Python**: .env

4. 構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの **エンドポイント** と認証 **キー** を反映します。 変更を保存します。

5. **computer-vision** フォルダーに、クライアント アプリケーションの次のコード ファイルが含まれていることを確認してください。

    - **C#** : Program.cs
    - **Python**: detect-faces.py

6. コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision SDK を使用するために必要な名前空間をインポートします

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

## <a name="view-the-image-you-will-analyze"></a>ビューの画像は、あなたが分析します

この演習では、Computer Vision サービスを使用して、人の画像を分析します。

1. Visual Studio Code で、**computer-vision** フォルダーとそれに含まれる **images** フォルダーを展開します。
2. **people.jpg** 画像を選択して表示します。

## <a name="detect-faces-in-an-image"></a>画像内の顔を検出する

これで、SDK を使用して Computer Vision サービスを呼び出し、画像内の顔を検出する準備が整いました。

1. クライアント アプリケーションのコード ファイル (**Program.cs** または **detect-faces.py**) の **Main** 関数で、構成設定をロードするためのコードが提供されていることを確認してください。 次に、コメント「**Authenticate Computer Vision client**」を見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision クライアント オブジェクトを作成および認証します

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

2. **Main** 関数の、追加したコードの下で、コードが画像ファイルへのパスを指定し、**AnalyzeFaces** という名前の関数に画像パスを渡していることを確認してください。 この関数はまだ完全には実装されていません。

3. **AnalyzeFaces** 関数のコメント「**Specify features to be retrieved (faces)** 」の下に、次のコードを追加します。

    **C#**

    ```C#
    // Specify features to be retrieved (faces)
    List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
    {
        VisualFeatureTypes.Faces
    };
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (faces)
    features = [VisualFeatureTypes.faces]
    ```

4. **AnalyzeFaces** 関数のコメント「**Get image analysis**」の下に、次のコードを追加します。

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // Get faces
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // Draw and annotate each face
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person aged approximately {face.Age}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

    # Get faces
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person aged approximately {}'.format(face.age)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```

5. 変更を保存し、**computer-vision** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-faces.py
    ```

6. 検出された顔の数を示す出力を確認します。
7. コード ファイルと同じフォルダーに生成された **detected_faces.jpg** ファイルを表示して、注釈付きの顔を確認します。 この場合、コードは顔の属性を使用して画像内の各人物の年齢を推定し、境界ボックスの座標を使用して各顔の周りに長方形を描画しています。

## <a name="prepare-to-use-the-face-sdk"></a>Face SDK を使用する準備をする

**Computer Vision** サービスは (他の多くの画像分析機能とともに) 基本的な顔検出を提供しますが、**Face** サービスは顔の分析と認識のためのより包括的な機能を提供します。

1. Visual Studio Code の **エクスプローラー** ペインで、**19-face** フォルダーを参照し、言語の設定に応じて **C#** または **Python** フォルダーを展開します。
2. **face-api** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Face SDK パッケージをインストールします。

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.4.1
    ```
    
3. **face-api** フォルダーの内容を表示し、構成設定用のファイルが含まれていることを確認してください。
    - **C#** : appsettings.json
    - **Python**: .env

4. 構成ファイルを開き、含まれている構成値を更新して、Cognitive Services リソースの **エンドポイント** と認証 **キー** を反映します。 変更を保存します。

5. **face-api** フォルダーに、クライアント アプリケーションの次のコード ファイルが含まれていることを確認してください。

    - **C#** : Program.cs
    - **Python**: analyze-faces.py

6. コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Computer Vision SDK を使用するために必要な名前空間をインポートします

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. **Main** 関数で、構成設定をロードするためのコードが提供されていることを確認してください。 次に、コメント「**Authenticate Face client**」を見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、**FaceClient** オブジェクトを作成および認証します。

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. **Main** 関数で、追加したコードの下に、コード内の関数を呼び出して Face サービスの機能を調べることができるメニューがコードに表示されることを確認してください。 この演習の残りの部分では、これらの関数を実装します。

## <a name="detect-and-analyze-faces"></a>顔を検出して分析する

Face サービスの最も基本的な機能の 1 つは、画像内の顔を検出し、年齢、感情表現、髪の色、眼鏡の存在などの属性を決定することです。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **1** を選択した場合に実行されるコードを調べます。 このコードは **DetectFaces** 関数を呼び出し、パスを画像ファイルに渡します。
2. コード ファイルで **DetectFaces** 関数を見付け、コメント「**Specify facial features to be retrieved**」の下に、次のコードを追加します。

    **C#**

    ```C#
    // Specify facial features to be retrieved
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Age,
        FaceAttributeType.Emotion,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.age,
                FaceAttributeType.emotion,
                FaceAttributeType.glasses]
    ```

3. **DetectFaces** 関数で、追加したコードの下に、コメント「**Get faces**」を見つけて、次のコードを追加します。

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            // Get face properties
            Console.WriteLine($"\nFace ID: {face.FaceId}");
            Console.WriteLine($" - Age: {face.FaceAttributes.Age}");
            Console.WriteLine($" - Emotions:");
            foreach (var emotion in face.FaceAttributes.Emotion.ToRankedList())
            {
                Console.WriteLine($"   - {emotion}");
            }

            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face ID: {face.FaceId}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            print('\nFace ID: {}'.format(face.face_id))
            detected_attributes = face.face_attributes.as_dict()
            age = 'age unknown' if 'age' not in detected_attributes.keys() else int(detected_attributes['age'])
            print(' - Age: {}'.format(age))

            if 'emotion' in detected_attributes:
                print(' - Emotions:')
                for emotion_name in detected_attributes['emotion']:
                    print('   - {}: {}'.format(emotion_name, detected_attributes['emotion'][emotion_name]))
            
            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face ID: {}'.format(face.face_id)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. **DetectFaces** 関数に追加したコードを調べます。 画像ファイルを分析し、年齢、感情、眼鏡の存在など、画像ファイルに含まれるすべての顔を検出します。 各顔に割り当てられた一意の顔識別子を含む、各顔の詳細が表示されます。顔の位置は、境界ボックスを使用して画像に示されます。
5. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    *C# 出力に、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これは無視してもかまいません。*

    **Python**

    ```
    python analyze-faces.py
    ```

6. プロンプトが表示されたら、「**1**」を入力し、出力を観察します。これには、検出された各顔の ID と属性が含まれている必要があります。
7. コード ファイルと同じフォルダーに生成された **detected_faces.jpg** ファイルを表示して、注釈付きの顔を確認します。

## <a name="compare-faces"></a>顔比較

一般的なタスクは、顔を比較し、類似している顔を見つけることです。 このシナリオでは、画像内の人物を *特定する* 必要はありません。複数の画像が *同じ* 人物 (または少なくとも似ている人物) を示しているかどうかを判断するだけです。 

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **2** を選択した場合に実行されるコードを調べます。 このコードは **CompareFaces** 関数を呼び出し、パスを 2 つの画像ファイル (**person1.jpg** と **people.jpg**) に渡します。
2. コード ファイルで **CompareFaces** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します。

**C#**

```C
// Determine if the face in image 1 is also in image 2
DetectedFace image_i_face;
using (var image1Data = File.OpenRead(image1))
{    
    // Get the first face in image 1
    var image1_faces = await faceClient.Face.DetectWithStreamAsync(image1Data);
    if (image1_faces.Count > 0)
    {
        image_i_face = image1_faces[0];
        Image img1 = Image.FromFile(image1);
        Graphics graphics = Graphics.FromImage(img1);
        Pen pen = new Pen(Color.LightGreen, 3);
        var r = image_i_face.FaceRectangle;
        Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        String output_file = "face_to_match.jpg";
        img1.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file); 

        //Get all the faces in image 2
        using (var image2Data = File.OpenRead(image2))
        {    
            var image2Faces = await faceClient.Face.DetectWithStreamAsync(image2Data);

            // Get faces
            if (image2Faces.Count > 0)
            {

                var image2FaceIds = image2Faces.Select(f => f.FaceId).ToList<Guid?>();
                var similarFaces = await faceClient.Face.FindSimilarAsync((Guid)image_i_face.FaceId,faceIds:image2FaceIds);
                var similarFaceIds = similarFaces.Select(f => f.FaceId).ToList<Guid?>();

                // Prepare image for drawing
                Image img2 = Image.FromFile(image2);
                Graphics graphics2 = Graphics.FromImage(img2);
                Pen pen2 = new Pen(Color.LightGreen, 3);
                Font font2 = new Font("Arial", 4);
                SolidBrush brush2 = new SolidBrush(Color.Black);

                // Draw and annotate each face
                foreach (var face in image2Faces)
                {
                    if (similarFaceIds.Contains(face.FaceId))
                    {
                        // Draw and annotate face
                        var r2 = face.FaceRectangle;
                        Rectangle rect2 = new Rectangle(r2.Left, r2.Top, r2.Width, r2.Height);
                        graphics2.DrawRectangle(pen2, rect2);
                        string annotation = "Match!";
                        graphics2.DrawString(annotation,font2,brush2,r2.Left, r2.Top);
                    }
                }

                // Save annotated image
                String output_file2 = "matched_faces.jpg";
                img2.Save(output_file2);
                Console.WriteLine(" Results saved in " + output_file2);   
            }
        }

    }
}
```

**Python**

```Python
# Determine if the face in image 1 is also in image 2
with open(image_1, mode="rb") as image_data:
    # Get the first face in image 1
    image_1_faces = face_client.face.detect_with_stream(image=image_data)
    image_1_face = image_1_faces[0]

    # Highlight the face in the image
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_1)
    draw = ImageDraw.Draw(image)
    color = 'lightgreen'
    r = image_1_face.face_rectangle
    bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
    draw = ImageDraw.Draw(image)
    draw.rectangle(bounding_box, outline=color, width=5)
    plt.imshow(image)
    outputfile = 'face_to_match.jpg'
    fig.savefig(outputfile)

# Get all the faces in image 2
with open(image_2, mode="rb") as image_data:
    image_2_faces = face_client.face.detect_with_stream(image=image_data)
    image_2_face_ids = list(map(lambda face: face.face_id, image_2_faces))

    # Find faces in image 2 that are similar to the one in image 1
    similar_faces = face_client.face.find_similar(face_id=image_1_face.face_id, face_ids=image_2_face_ids)
    similar_face_ids = list(map(lambda face: face.face_id, similar_faces))

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_2)
    draw = ImageDraw.Draw(image)

    # Draw and annotate matching faces
    for face in image_2_faces:
        if face.face_id in similar_face_ids:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline='lightgreen', width=10)
            plt.annotate('Match!',(r.left, r.top + r.height + 15), backgroundcolor='white')

    # Save annotated image
    plt.imshow(image)
    outputfile = 'matched_faces.jpg'
    fig.savefig(outputfile)
```

3. **CompareFaces** 関数に追加したコードを調べます。 画像 1 で最初の顔を見つけ、新しい画像ファイルに **face_to_match.jpg** という名前の注釈が付けられます。 次に、画像 2 のすべての顔を検索し、それらの顔 ID を使用して画像 1 に類似した顔を検索します。 類似したものに注釈が付けられ、**matched_faces.jpg** という名前の新しい画像に保存されます。
4. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    *C# 出力に、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これは無視してもかまいません。*

    **Python**

    ```
    python analyze-faces.py
    ```
    
5. プロンプトが表示されたら、「**2**」を入力して出力を確認します。 次に、コード ファイルと同じフォルダーに生成された **face_to_match.jpg** と **matched_faces.jpg** のファイルを表示して、一致した顔を確認します。
6. メニュー オプション **2** の **Main** 関数のコードを編集して **person2.jpg** と **people.jpg** を比較し、プログラムを再実行して結果を表示します。

## <a name="train-a-facial-recognition-model"></a>顔認識モデルをトレーニングする

AI アプリケーションで顔を認識できる特定の人々のモデルを維持する必要があるシナリオがあるかもしれません。 たとえば、顔認識を使用して安全な建物内の従業員の身元を確認する生体認証セキュリティシステムを促進するためです。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **3** を選択した場合に実行されるコードを調べます。 このコードは **TrainModel** 関数を呼び出し、Cognitive Services リソースに登録される新しい **PersonGroup** の名前と ID、および従業員名のリストを渡します。
2. **face-api/images** フォルダーに、employees と同じ名前のフォルダーがあることを確認します。 各フォルダにーは、指定された従業員の複数の画像が含まれています。
3. コード ファイルで **TrainModel** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します。

**C#**

```C
// Delete group if it already exists
var groups = await faceClient.PersonGroup.ListAsync();
foreach(var group in groups)
{
    if (group.PersonGroupId == groupId)
    {
        await faceClient.PersonGroup.DeleteAsync(groupId);
    }
}

// Create the group
await faceClient.PersonGroup.CreateAsync(groupId, groupName);
Console.WriteLine("Group created!");

// Add each person to the group
Console.Write("Adding people to the group...");
foreach(var personName in imageFolders)
{
    // Add the person
    var person = await faceClient.PersonGroupPerson.CreateAsync(groupId, personName);

    // Add multiple photo's of the person
    string[] images = Directory.GetFiles("images/" + personName);
    foreach(var image in images)
    {
        using (var imageData = File.OpenRead(image))
        { 
            await faceClient.PersonGroupPerson.AddFaceFromStreamAsync(groupId, person.PersonId, imageData);
        }
    }

}

    // Train the model
Console.WriteLine("Training model...");
await faceClient.PersonGroup.TrainAsync(groupId);

// Get the list of people in the group
Console.WriteLine("Facial recognition model trained with the following people:");
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    Console.WriteLine($"-{person.Name}");
}
```

**Python**

```Python
# Delete group if it already exists
groups = face_client.person_group.list()
for group in groups:
    if group.person_group_id == group_id:
        face_client.person_group.delete(group_id)

# Create the group
face_client.person_group.create(group_id, group_name)
print ('Group created!')

# Add each person to the group
print('Adding people to the group...')
for person_name in image_folders:
    # Add the person
    person = face_client.person_group_person.create(group_id, person_name)

    # Add multiple photo's of the person
    folder = os.path.join('images', person_name)
    person_pics = os.listdir(folder)
    for pic in person_pics:
        img_path = os.path.join(folder, pic)
        img_stream = open(img_path, "rb")
        face_client.person_group_person.add_face_from_stream(group_id, person.person_id, img_stream)

# Train the model
print('Training model...')
face_client.person_group.train(group_id)

# Get the list of people in the group
print('Facial recognition model trained with the following people:')
people = face_client.person_group_person.list(group_id)
for person in people:
    print('-', person.name)
```

4. **TrainModel** 関数に追加したコードを調べます。 次のタスクを実行します。
    - サービスに登録されている **PersonGroup** のリストを取得し、指定されているものが存在する場合は削除します。
    - 指定された ID と名前でグループを作成します。
    - 指定した名前ごとにグループに人物を追加し、各人物の複数の画像を追加します。
    - グループ内の指名された人々とその顔画像に基づいて顔認識モデルをトレーニングします。
    - グループ内の名前付きの人のリストを取得して表示します。
5. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    *C# 出力に、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これは無視してもかまいません。*

    **Python**

    ```
    python analyze-faces.py
    ```

6. プロンプトが表示されたら、「**3**」を入力して出力を確認し、作成された **PersonGroup** に 2 人が含まれていることを確認してください。

## <a name="recognize-faces"></a>顔の認識

**PeopleGroup** を定義し、顔認識モデルをトレーニングしたので、これを使用して、画像内の名前付きの個人を認識することができます。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション「**4**」を選択した場合に実行されるコードを調べます。 このコードは **RecognizeFaces** 関数を呼び出し、画像ファイル (**people.jpg**) へのパスと、顔の識別に使用される **PeopleGroup** の ID を渡します。
2. コード ファイルで **RecognizeFaces** 関数を見つけ、コンソールにメッセージを出力する既存のコードの下に、次のコードを追加します。

**C#**

```C
// Detect faces in the image
using (var imageData = File.OpenRead(imageFile))
{    
    var detectedFaces = await faceClient.Face.DetectWithStreamAsync(imageData);

    // Get faces
    if (detectedFaces.Count > 0)
    {
        
        // Get a list of face IDs
        var faceIds = detectedFaces.Select(f => f.FaceId).ToList<Guid?>();

        // Identify the faces in the people group
        var recognizedFaces = await faceClient.Face.IdentifyAsync(faceIds, groupId);

        // Get names for recognized faces
        var faceNames = new Dictionary<Guid?, string>();
        if (recognizedFaces.Count> 0)
        {
            foreach(var face in recognizedFaces)
            {
                var person = await faceClient.PersonGroupPerson.GetAsync(groupId, face.Candidates[0].PersonId);
                Console.WriteLine($"-{person.Name}");
                faceNames.Add(face.FaceId, person.Name);

            }
        }

        
        // Annotate faces in image
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen penYes = new Pen(Color.LightGreen, 3);
        Pen penNo = new Pen(Color.Magenta, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Cyan);
        foreach (var face in detectedFaces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            if (faceNames.ContainsKey(face.FaceId))
            {
                // If the face is recognized, annotate in green with the name
                graphics.DrawRectangle(penYes, rect);
                string personName = faceNames[face.FaceId];
                graphics.DrawString(personName,font,brush,r.Left, r.Top);
            }
            else
            {
                // Otherwise, just annotate the unrecognized face in magenta
                graphics.DrawRectangle(penNo, rect);
            }
        }

        // Save annotated image
        String output_file = "recognized_faces.jpg";
        image.Save(output_file);
        Console.WriteLine("Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Detect faces in the image
with open(image_file, mode="rb") as image_data:

    # Get faces
    detected_faces = face_client.face.detect_with_stream(image=image_data)

    # Get a list of face IDs
    face_ids = list(map(lambda face: face.face_id, detected_faces))

    # Identify the faces in the people group
    recognized_faces = face_client.face.identify(face_ids, group_id)

    # Get names for recognized faces
    face_names = {}
    if len(recognized_faces) > 0:
        print(len(recognized_faces), 'faces recognized.')
        for face in recognized_faces:
            person_name = face_client.person_group_person.get(group_id, face.candidates[0].person_id).name
            print('-', person_name)
            face_names[face.face_id] = person_name

    # Annotate faces in image
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    for face in detected_faces:
        r = face.face_rectangle
        bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
        draw = ImageDraw.Draw(image)
        if face.face_id in face_names:
            # If the face is recognized, annotate in green with the name
            draw.rectangle(bounding_box, outline='lightgreen', width=3)
            plt.annotate(face_names[face.face_id],
                        (r.left, r.top + r.height + 15), backgroundcolor='white')
        else:
            # Otherwise, just annotate the unrecognized face in magenta
            draw.rectangle(bounding_box, outline='magenta', width=3)

    # Save annotated image
    plt.imshow(image)
    outputfile = 'recognized_faces.jpg'
    fig.savefig(outputfile)

    print('\nResults saved in', outputfile)
```
    
3. **RecognizeFaces** 関数に追加したコードを調べます。 画像内の顔を見つけて、ID のリストを作成します。 次に、people グループを使用して、顔 ID のリストで顔を識別しようとします。 認識された顔には、識別された人物の名前が注釈として付けられ、結果は **recognized_faces.jpg** に保存されます。
4. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    *C# 出力に、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これは無視してもかまいません。*

    **Python**

    ```
    python analyze-faces.py
    ```

5. プロンプトが表示されたら、「**4**」を入力して出力を確認します。 次に、コード ファイルと同じフォルダーに生成された **recognized_faces.jpg** ファイルを表示して、識別された人物を確認します。
6. メニューオプション **4** の **Main** 関数のコードを編集して **people2.jpg** の顔を認識し、プログラムを再実行して結果を表示します。 同じ人が認識されるべきです。

## <a name="verify-a-face"></a>顔の確認

顔認識は、身元の確認によく使用されます。 Face サービスを使用すると、画像内の顔を別の顔と比較したり、**PersonGroup** に登録されている人物から確認したりできます。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **5** を選択した場合に実行されるコードを調べます。 このコードは **VerifyFace** 関数を呼び出し、画像ファイル (**person1.jpg**) へのパスと、顔の識別に使用される **PeopleGroup** の ID を渡します。
2. コード ファイルで **VerifyFace** 関数を見つけ、コメント「**Get the ID of the person from the people group**」の下 (結果を出力するコードの上) に次のコードを追加します。

**C#**

```C
// Get the ID of the person from the people group
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    if (person.Name == personName)
    {
        Guid personId = person.PersonId;

        // Get the first face in the image
        using (var imageData = File.OpenRead(personImage))
        {    
            var faces = await faceClient.Face.DetectWithStreamAsync(imageData);
            if (faces.Count > 0)
            {
                Guid faceId = (Guid)faces[0].FaceId;

                //We have a face and an ID. Do they match?
                var verification = await faceClient.Face.VerifyFaceToPersonAsync(faceId, personId, groupId);
                if (verification.IsIdentical)
                {
                    result = "Verified";
                }
            }
        }
    }
}
```

**Python**

```Python
# Get the ID of the person from the people group
people = face_client.person_group_person.list(group_id)
for person in people:
    if person.name == person_name:
        person_id = person.person_id

        # Get the first face in the image
        with open(person_image, mode="rb") as image_data:
            faces = face_client.face.detect_with_stream(image=image_data)
            if len(faces) > 0:
                face_id = faces[0].face_id

                # We have a face and an ID. Do they match?
                verification = face_client.face.verify_face_to_person(face_id, person_id, group_id)
                if verification.is_identical:
                    result = 'Verified'
```

3. **VerifyFace** 関数に追加したコードを調べます。 指定された名前のグループ内の人の ID を検索します。 人物が存在する場合、コードは画像の最初の顔の顔 ID を取得します。 最後に、画像に顔がある場合、コードは指定された人物の ID に対して顔を検証します。
4. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python analyze-faces.py
    ```

5. プロンプトが表示されたら、「**5**」を入力して結果を確認します。
6. メニュー オプション **5** の **Main** 関数のコードを編集して、名前と画像 **person1.jpg** および **person2.jpg** のさまざまな組み合わせを試してみてください。

## <a name="more-information"></a>詳細情報

顔検出に **Computer Vision** サービスを使用する方法の詳細については、[Computer Vision のドキュメント](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)を参照してください。

**Face** サービスの詳細については、[Face のドキュメント](https://docs.microsoft.com/azure/cognitive-services/face/)を参照してください。
