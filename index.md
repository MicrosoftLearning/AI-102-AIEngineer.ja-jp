---
title: オンライン ホステッド インストラクション
permalink: index.html
layout: home
ms.openlocfilehash: ba3d2a5154e5e9b08e750f7d9ced8b62048dd66d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819469"
---
# <a name="content-directory"></a>コンテンツ ディレクトリ

このリポジトリには、Microsoft のコース「[AI-102 Microsoft Azure AI ソリューションの設計と実装](https://docs.microsoft.com/learn/certifications/courses/ai-102t00)」の実践的な演習と、同等の [Microsoft Learn のマイペースで進められるモジュール](https://aka.ms/AzureLearn_AIEngineer)が含まれています。 演習は学習教材に沿って設計されており、教材で説明されているテクノロジを使って学習できます。

この演習を完了するには、Microsoft Azure サブスクリプションが必要です。 講師からサブスクリプションが提供されていない場合は、[https://azure.microsoft.com](https://azure.microsoft.com) で無料の試用版にサインアップできます。

## <a name="labs"></a>ラボ

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| モジュール | ラボ |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

