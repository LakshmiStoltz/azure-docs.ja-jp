---
title: Visual Studio WebApi プロジェクトで Azure AD の使用を開始する
description: Visual Studio 接続済みサービスを使用して Azure Active Directory を接続または作成した後に、Web API プロジェクトで Azure AD の使用を開始する方法について説明します。
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.openlocfilehash: 3e925a920d85ba90d54ea844b5c2592806496d7f
ms.sourcegitcommit: 0770a7d91278043a83ccc597af25934854605e8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/13/2021
ms.locfileid: "124786220"
---
# <a name="get-started-with-azure-active-directory-webapi-projects"></a>Azure Active Directory の使用開始 (WebApi プロジェクト)

> [!div class="op_single_selector"]
> - [作業の開始](vs-active-directory-webapi-getting-started.md)
> - [変更内容](vs-active-directory-webapi-what-happened.md)

この記事では、Visual Studio の **[プロジェクト] > [接続済みサービス]** コマンドを使用して Active Directory を ASP.NET WebAPI プロジェクトに追加した後のガイダンスを説明します。 プロジェクトにサービスをまだ追加していない場合は、いつでも行うことができます。

接続済みサービスを追加したときにプロジェクトに加えられる変更について詳しくは、「[WebAPI プロジェクトの変更点](vs-active-directory-webapi-what-happened.md)」をご覧ください。

## <a name="requiring-authentication-to-access-controllers"></a>コントローラーへのアクセスに対して認証を要求する

プロジェクトに含まれるすべてのコントローラーには、 `[Authorize]` 属性が設定されています。 この属性により、ユーザーがこれらのコントローラーによって定義された API にアクセスする際に認証が求められます。 これらのコントローラーに匿名でアクセスできるようにするには、コントローラーからこの属性を削除します。 より細かなレベルでアクセス許可を設定するには、コントローラー クラスではなく、認証を必要とするそれぞれのメソッドに対してこの属性を割り当てます。

## <a name="next-steps"></a>次のステップ

- [Azure AD の認証シナリオ](./authentication-vs-authorization.md)
- [ASP.NET Web アプリへの "Microsoft でサインイン" の追加](quickstart-v2-aspnet-webapp.md)