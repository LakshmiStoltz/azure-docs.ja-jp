---
title: ファイアウォール規則の管理 - Hyperscale (Citus) - Azure Database for PostgreSQL
description: Azure portal を使用して Azure Database for PostgreSQL - Hyperscale (Citus) のファイアウォール規則を作成および管理します
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 10/15/2021
ms.openlocfilehash: 23874f83e55fb8ff1ea3f60872cdfc10a64dfc80
ms.sourcegitcommit: 37cc33d25f2daea40b6158a8a56b08641bca0a43
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/15/2021
ms.locfileid: "130071581"
---
# <a name="manage-public-access-for-azure-database-for-postgresql---hyperscale-citus"></a>Azure Database for PostgreSQL - Hyperscale (Citus) のパブリック アクセスを管理する

サーバーレベルのファイアウォール規則を使用して、パブリック インターネット内の指定した IP アドレス (または IP アドレス範囲) から Hyperscale (Citus) コーディネーター ノードへの[パブリック アクセス](concepts-hyperscale-firewall-rules.md)を管理できます。

## <a name="prerequisites"></a>前提条件
このハウツー ガイドの手順を実行するには、以下が必要です。
- サーバー グループ ([Azure Database for PostgreSQL - Hyperscale (Citus) サーバー グループの作成](quickstart-create-hyperscale-portal.md)に関する記事を参照)。

## <a name="create-a-server-level-firewall-rule-in-the-azure-portal"></a>Azure Portal でサーバーレベルのファイアウォール規則を作成する

> [!NOTE]
> これらの設定には、Azure Database for PostgreSQL - Hyperscale (Citus) サーバー グループの作成時にもアクセスできます。 **[ネットワーク]** タブの下で、 **[パブリック エンドポイント]** をクリックします。

> :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/0-create-public-access.png" alt-text="Azure portal - [ネットワーク] タブ":::

1. PostgreSQL サーバー グループのページで、[セキュリティ] 見出しの下の **[ネットワーク]** をクリックして、ファイアウォール規則を開きます。

   :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/1-connection-security.png" alt-text="Azure portal - [ネットワーク] のクリック":::

2. 必要に応じて、**[Enable access to the worker nodes]\(ワーカー ノードへのアクセスを有効にする\)** をオンにします。 このオプションを使用すると、ファイアウォール規則ですべてのワーカー ノードとコーディネーター ノードへのアクセスが許可されます。

3. **[現在のクライアント IP アドレスを追加する]** をクリックすると、Azure システムによって認識されたコンピューターのパブリック IP アドレスでファイアウォール規則が作成されます。

あるいは、(オプション B の右側にある) **[+ Add 0.0.0.0 - 255.255.255.255]** をクリックすると、IP だけでなく、インターネット全体でコーディネーター ノードのポート 5432 にアクセスできるようになります。 この場合も、クライアントでは、クラスターを使用するために正しいユーザー名とパスワードでログインする必要があります。 ただし、世界中からのアクセスは短時間のみ、運用環境以外のデータベースだけに許可することをお勧めします。

4. 構成を保存する前に、IP アドレスを確認します。 場合によっては、Azure Portal で見られる IP アドレスは、インターネットおよび Azure サーバーにアクセスするときに使用する IP アドレスと異なることがあります。 そのため、開始 IP と終了 IP を変更してルール関数を予想どおりにする必要があります。
   検索エンジンまたはその他のオンライン ツールを使用して、自分の IP アドレスを確認します (たとえば、"what is my IP" を検索します)。

   :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/3-what-is-my-ip.png" alt-text="「What is my IP」の Bing 検索":::

5. さらにアドレス範囲を追加します。 このファイアウォール規則では、単一の IP アドレスまたはアドレスの範囲を指定できます。 規則を単一の IP アドレスに限定する場合は、[開始 IP] と [終了 IP] のフィールドに同じアドレスを入力します。 ファイアウォールを開くと、管理者、ユーザー、およびアプリケーションは、ポート 5432 でコーディネーター ノードにアクセスできるようになります。

6. ツール バーの **[保存]** をクリックしてこのサーバーレベルのファイアウォール規則を保存します。 ファイアウォール規則の更新が成功したという確認を待機します。

## <a name="connecting-from-azure"></a>Azure からの接続

Azure でホストされているアプリケーション (Azure Web Apps アプリケーションや Azure VM で実行されるアプリケーションなど) に、Hyperscale (Citus) データベースへのアクセス権を簡単に付与する方法があります。 ポータルの **[ネットワーク]** ペインで **[Azure サービスおよびリソースにこのサーバー グループへのアクセスを許可する]** チェック ボックスをオンにして、**[保存]** を選択します。

> [!IMPORTANT]
> このオプションでは、他のお客様のサブスクリプションからの接続を含む、Azure からのすべての接続を許可するようにファイアウォールを構成します。 このオプションを選択する場合は、ログインおよびユーザーのアクセス許可が、承認されたユーザーのみにアクセスを制限していることを確認してください。

## <a name="manage-existing-server-level-firewall-rules-through-the-azure-portal"></a>Azure ポータルで既存のサーバー レベルのファイアウォール規則を管理する
ファイアウォール規則を管理する手順を繰り返します。
* 現在のコンピューターを追加するには、 **[現在のクライアント IP アドレスを追加する]** のボタンをクリックします。 **[保存]** をクリックして変更を保存します。
* さらに IP アドレスを追加するには、規則名、開始および終了 IP アドレスを入力します。 **[保存]** をクリックして変更を保存します。
* 既存の規則を変更するには、規則内の任意のフィールドをクリックし、変更します。 **[保存]** をクリックして変更を保存します。
* 既存の規則を削除するには、省略記号 [...] をクリックし、 **[削除]** をクリックします。 **[保存]** をクリックして変更を保存します。

## <a name="next-steps"></a>次のステップ
- 接続の問題のトラブルシューティング方法を含む、[ファイアウォール規則の概念](concepts-hyperscale-firewall-rules.md)について確認します。
