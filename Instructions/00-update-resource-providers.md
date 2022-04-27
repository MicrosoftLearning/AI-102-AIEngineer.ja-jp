---
lab:
  title: リソース プロバイダーを有効にする
  module: Setup
ms.openlocfilehash: 2cd48d0d9f67b9bab3e64452bf6199e1f438492a
ms.sourcegitcommit: e06fbeaa3bc15e4dbe99f72216dfeeb27f58babd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/15/2022
ms.locfileid: "138765493"
---
# <a name="enable-resource-providers"></a>リソース プロバイダーを有効にする

Azure サブスクリプションに登録する必要のあるリソース プロバイダーがいくつかあります。 次の手順に従って、登録されていることを確認してください。

1. ご利用の Azure サブスクリプションに関連付けられている Microsoft 資格情報を使用して、Azure portal `https://portal.azure.com` にサインインします。
2. **[ホーム]** ページの **[サブスクリプション]** を選択します (または **[&#8801;]** メニューを展開し、 **[すべてのサービス]** を選択し、 **[全般]** カテゴリの **[サブスクリプション]** を選択します)。
3. Azure サブスクリプションを選択します (複数のサブスクリプションがある場合は、Azure Pass を利用して作成したサブスクリプションを選択します)。
4. サブスクリプションのブレードの左側のウィンドウにある **[設定]** セクションで、 **[リソース プロバイダー]** を選択します。
5. リソース プロバイダーのリストで、次のプロバイダーが登録されていることを確認します (登録されていない場合は、それらを選択して **[登録]** をクリックします)。
    - Microsoft.BotService
    - Microsoft.Web
    - Microsoft.ManagedIdentity
    - Microsoft.Search
    - Microsoft.Storage
    - Microsoft.CognitiveServices
    - Microsoft.AlertsManagement
    - microsoft.insights
    - Microsoft.KeyVault
    - Microsoft.ContainerInstance
