---
lab:
  title: ラボ環境のセットアップ
  module: Setup
ms.openlocfilehash: 57363ce9fec94cb3a1a6ef7622a10bca48a6f055
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819429"
---
# <a name="lab-environment-setup"></a>ラボ環境のセットアップ

これらの演習は、ホストされたラボ環境で完了するように設計されています。 ご自分のパソコンで完成させたい場合は、以下のソフトウェアをインストールしてください。 独自の環境を使用すると、予期しないダイアログや動作が発生する場合があります。 また、独自の環境は大きく異なる可能性があるため、コースチームは発生する問題をサポートできない場合があります。

> **注:** 以下の手順は、Windows 10 コンピューター用です。 Linux または MacOS を使用することもできます。 選択した OS に応じたラボ手順を適応する必要があります。

### <a name="base-operating-system-windows-10"></a>基本オペレーティング システム (Windows 10)

#### <a name="windows-10"></a>Windows 10

Windows 10 をインストールし、すべての更新プログラムを適用します。

#### <a name="edge"></a>Edge

[Edge (Chromium)](https://microsoft.com/edge) をインストールします

### <a name="net-core-sdk"></a>.NET Core SDK

1. https://dotnet.microsoft.com/download ダウンロードしてインストールします (ランタイムだけでなく、.NET Core SDK をダウンロードします)

### <a name="c-redistributable"></a>C++ 再頒布可能パッケージ

1. https://aka.ms/vs/16/release/vc_redist.x64.exe から Visual C++ 再頒布可能パッケージ (x64) をダウンロードしてインストールします

### <a name="nodejs"></a>Node.JS

1. https://nodejs.org/en/download/ から最新の LTS バージョンをダウンロードします 
2. 既定のオプションを使用してインストールします

### <a name="python-and-required-packages"></a>Python (および必要なパッケージ)

1. https://docs.conda.io/en/latest/miniconda.html からバージョン3.8 をダウンロードします 
2. セットアップを実行してインストールします - **重要**: Miniconda を PATH 変数に追加し、Miniconda を既定の Python 環境として登録するオプションを選択します。
3. インストール後、Anaconda プロンプトを開き、次のコマンドを入力してパッケージをインストールします。 

```
pip install flask requests python-dotenv pylint matplotlib pillow
pip install --upgrade numpy
```

### <a name="azure-cli"></a>Azure CLI

1. https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest からダウンロードします 
2. 既定のオプションを使用してインストールします

### <a name="git"></a>Git

1. 既定のオプションを使用して https://git-scm.com/download.html からダウンロードしてインストールします


### <a name="visual-studio-code-and-extensions"></a>Visual Studio Code (および拡張機能)

1. https://code.visualstudio.com/Download からダウンロードします 
2. 既定のオプションを使用してインストールします 
3. インストール後、Visual Studio Code を起動し、 **[拡張機能]** タブ (CTRL + SHIFT + X) で、Microsoft から次の拡張機能を検索してインストールします。
    - Python
    - C#
    - Azure Functions
    - PowerShell


### <a name="bot-framework-emulator"></a>Bot Framework エミュレーター

https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md の手順に従って、お使いのオペレーティングシステム用の最新の安定バージョンの Bot Framework Emulator をダウンロードしてインストールします。

### <a name="bot-framework-composer"></a>Bot Framework Composer

https://docs.microsoft.com/en-us/composer/install-composer からインストールします。
