---
title: "Azure Active Directory B2B 协作 API 和自定义 | Microsoft 文档"
description: "Azure Active Directory B2B 协作可让业务合作伙伴有选择性地访问本方的企业应用程序，为跨公司合作关系提供支持"
services: active-directory
documentationcenter: 
author: sasubram
manager: mtillman
editor: 
tags: 
ms.assetid: 
ms.service: active-directory
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: identity
ms.date: 04/11/2017
ms.author: sasubram
ms.openlocfilehash: 9e1dcd38e7c05a234e4b8d7a6190770d813a227f
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/11/2017
---
# <a name="azure-active-directory-b2b-collaboration-api-and-customization"></a>Azure Active Directory B2B 协作 API 和自定义

已经有许多客户和我们说他们想要以最适合其组织的方式自定义邀请过程。 使用我们的 API，便可以实现该想法。 [https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation)

## <a name="capabilities-of-the-invitation-api"></a>邀请 API 的功能
该 API 提供以下功能：

1. 使用*任何*电子邮件地址邀请外部用户。

    ```
    "invitedUserDisplayName": "Sam"
    "invitedUserEmailAddress": "gsamoogle@gmail.com"
    ```

2. 自定义希望用户在接受其邀请后登陆的位置。

    ```
    "inviteRedirectUrl": "https://myapps.microsoft.com/"
    ```

3. 选择通过我们将包含可自定义消息的标准邀请邮件

    ```
    "sendInvitationMessage": true
    ```

  发送给收件人

    ```
    "customizedMessageBody": "Hello Sam, let's collaborate!"
    ```

4. 并选择你想要让其了解你邀请了此协作者的抄送人员。

5. 或者完全自定义邀请，并通过选择不通过 Azure AD 发送通知加入工作流。

    ```
    "sendInvitationMessage": false
    ```

  在这种情况下，将通过可嵌入电子邮件模板的 API、即时消息或所选择的其他分发方法收到兑换 URL。

6. 最后，如果是管理员，可以选择以成员身份邀请用户。

    ```
    "invitedUserType": "Member"
    ```


## <a name="authorization-model"></a>授权模型
可以在以下授权模式下运行 API：

### <a name="app--user-mode"></a>“应用 + 用户”模式
在此模式下，任何使用 API 的用户都需要有创建 B2B 邀请的权限。

### <a name="app-only-mode"></a>“仅应用”模式
在仅应用上下文中，应用需要 User.ReadWrite.All 或 Directory.ReadWrite.All 作用域才能使邀请成功。

有关详细信息，请参阅：https://graph.microsoft.io/docs/authorization/permission_scopes


## <a name="powershell"></a>PowerShell
现在可以使用 PowerShell 轻松地将外部用户添加并邀请到组织。 使用 cmdlet 创建邀请：

```
New-AzureADMSInvitation
```

可以使用以下选项：

* -InvitedUserDisplayName
* -InvitedUserEmailAddress
* -SendInvitationMessage
* -InvitedUserMessageInfo

还可在 [https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation) 中查看邀请 API 参考

## <a name="next-steps"></a>后续步骤

在 Azure AD B2B 协作网站上浏览我们的其他文章：

* [什么是 Azure AD B2B 协作？](active-directory-b2b-what-is-azure-ad-b2b.md)
* [Azure Active Directory 管理员如何添加 B2B 协作用户？](active-directory-b2b-admin-add-users.md)
* [信息工作者如何添加 B2B 协作用户？](active-directory-b2b-iw-add-users.md)
* [B2B 协作邀请电子邮件的元素](active-directory-b2b-invitation-email.md)
* [B2B 协作邀请兑换](active-directory-b2b-redemption-experience.md)
* [Azure AD B2B 协作授权](active-directory-b2b-licensing.md)
* [Azure Active Directory B2B 协作疑难解答](active-directory-b2b-troubleshooting.md)
* [Azure Active Directory B2B 协作常见问题 (FAQ)](active-directory-b2b-faq.md)
* [适用于 B2B 协作用户的多重身份验证](active-directory-b2b-mfa-instructions.md)
* [在没有邀请的情况下添加 B2B 协作用户](active-directory-b2b-add-user-without-invite.md)
* [有关 Azure Active Directory 中应用程序管理的文章索引](active-directory-apps-index.md)
