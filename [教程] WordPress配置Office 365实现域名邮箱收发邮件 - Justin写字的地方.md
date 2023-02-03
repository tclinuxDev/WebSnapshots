---
page-title: "[教程] WordPress配置Office 365实现域名邮箱收发邮件 - Justin写字的地方"
url: https://zblogs.top/set-up-wordpress-office-365-domain-email/
date: "2023-02-03 14:51:41"
---
4.  [配置 Mail Integration for Office 365/Outlook 插件](https://zblogs.top/set-up-wordpress-office-365-domain-email/#mail-integration-for-office-365-outlook "配置 Mail Integration for Office 365/Outlook 插件")
    

## 先决条件

本教程适用于 Office 365、Outlook 和 Microsoft Exchange 的付费个人版和企业版，不支持 Live、Outlook.com、Hotmail 等免费帐户，因为 Microsoft 尚未为此类帐户启用 Graph API。

本教程是通过一款名为 [Mail Integration for Office 365/Outlook](https://wordpress.org/plugins/mail-integration-365/) 的免费 WordPress 插件来进行配置的。要使用这款插件，需要先设置 Microsoft Azure Active Directory 帐户。

此外，由于 Microsoft 已禁用通过 SMTP 协议的基本身份验证（用户名和密码），这个 WordPres 插件是通过 OAuth 2.0 来进行身份验证，因此你需要确保网站已部署 SSL 证书，以实现通过安全连接来交换 OAuth 密钥。

如果你的网站还没部署 SSL 证书，可参照下面这个教程进行部署：

如果你使用的是 Office 365，你可能已经注册了 Microsoft Azure Active Directory 帐户；如果你还没有 Microsoft Azure Active Directory 帐户，可以先去[注册](https://azure.microsoft.com/zh-cn/services/active-directory/)，或申请一个免费的Microsoft 365 开发者 E5 订阅：

再次安利一下：

我使用的是免费的 Microsoft 365 开发者 E5 订阅来给网站配置 Office 365 实现域名邮箱收发邮件，如果你也想免费申请，可以参照下面这份文档：

申领 Microsoft 365 开发者 E5 订阅，不仅可以获得25个 Office 365 许可证，还可得到高达 5TB 的OneDrive 储存空间：

---

回到正题。

如果你已经搞定了 Microsoft Azure Active Directory 帐户，那你可以先前往你的 WordPress 网站后台，进入 **Plugins** > **Add New** 先将这款免费插件下载安装好，并**激活插件**。

## 创建一个域名邮箱

要使用自己的网站域名邮箱收发邮件，需要先将域名添加到 Microsoft 365 admin center，并创建一个域名邮箱：

**1.** [登录 Microsoft 365 admin center](https://admin.microsoft.com/)；前往**设置** > **域**，点击**添加域**，按照屏幕指示操作将自己的网站域名添加到 Microsoft 365 admin center。

![Microsoft 365 admin center 添加域](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Microsoft-365-admin-center-%E6%B7%BB%E5%8A%A0%E5%9F%9F.webp?resize=860%2C737&ssl=1)

**2.** 添加好域名后，转到活动用户页面添加一个用户（域名邮箱）。

![添加活动用户（域名邮箱）](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/%E6%B7%BB%E5%8A%A0%E6%B4%BB%E5%8A%A8%E7%94%A8%E6%88%B7%EF%BC%88%E5%9F%9F%E5%90%8D%E9%82%AE%E7%AE%B1%EF%BC%89-scaled.webp?resize=860%2C439&ssl=1)

> **注意：**在创建用户时应该需要给这个用户分配 Azure Active Directory 管理员权限，这样才能在 Azure Active Directory 中创建和注册应用程序。由于我用的是 Microsoft 365 E5 开发者账号，我是直接给这个用户分配了 Microsoft 365 E5 开发者许可证。

接下来，需要进一步配置 SPF, DKIM 以及 DMARC 以防止域名邮箱发出的邮件被接收邮箱列为 Spam。

## 配置 SPF, DKIM 以及 DMARC

由于我们前面的步骤使用了自定义域，因此需要前往域名解析提供商（e.g. 我目前使用的是 Cloudflare 进行域名解析）修改 DNS 配置，依次配置好 SPF, DKIM 和 DMARC，以确保我们域名邮箱发出的邮件不会被接收邮箱列为 Spam。

-   **发送方策略框架 SPF (Sender Policy Framework)** 有助于识别允许使用该域名发送电子邮件的IP地址；
-   **域密钥识别邮件 DKIM (DomainKeys Identified Mail)** 电子邮件身份验证的目标是证明邮件的内容没有被篡改，如果电子邮件是从域名管理员配置的授权服务器发送，则使用加密进行验证；
-   **基于域的消息认证、报告和一致性 DMARC (Domain-based Message Authentication, Reporting & Conformance)** 电子邮件身份验证的目标是确保 SPF 和 DKIM 信息与发件人地址匹配。

Microsoft 提供了详细的配置文档，想要深入了解的童鞋可以自行查阅以下文档，根据文档提示的操作进行配置：

-   [设置 SPF 以防欺骗](https://docs.microsoft.com/zh-cn/microsoft-365/security/office-365-security/set-up-spf-in-office-365-to-help-prevent-spoofing?view=o365-worldwide)
-   [使用 DKIM 验证从自定义域发送的出站电子邮件](https://docs.microsoft.com/zh-cn/microsoft-365/security/office-365-security/use-dkim-to-validate-outbound-email?view=o365-worldwide)
-   [使用 DMARC 验证电子邮件](https://docs.microsoft.com/zh-cn/microsoft-365/security/office-365-security/use-dmarc-to-validate-email?view=o365-worldwide)

下面我贴一下我的网站域名 DNS 相关配置：

项目

类型

名称

内容

TTL

SPF

TXT

zblogs.top

v=spf1 mx a include:spf.protection.outlook.com ~all

1 小时

DKIM

CNAME

selector1.\_domainkey

selector1-zblogs-top.\_domainkey.zblogs.onmicrosoft.com

1 分钟

DKIM

CNAME

selector2.\_domainkey

selector2-zblogs-top.\_domainkey.zblogs.onmicrosoft.com

1 分钟

DMARC

TXT

\_dmarc

v=DMARC1; p=none; pct=100

1 小时

配置完成后，可使用域名邮箱发送测试邮件到你的个人邮箱，测试一下域名邮箱发件会不会被接收邮箱列为 Spam。你也可以使用 [Mail Tester](https://www.mail-tester.com/) 工具来检查你的域名邮箱配置：

![Mail Tester score](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Mail-Tester-score-scaled.webp?resize=860%2C554&ssl=1)

可以达到8分，感觉还行哈哈

在确认使用域名邮箱发件大概率不会被列为 Spam 后，进入下一步。

## 配置 Mail Integration for Office 365/Outlook 插件

Mail Integration for Office 365/Outlook 插件的作用是通过你前面创建好的域名邮箱中继从 WordPress 发送的所有电子邮件。需要注意的是，虽然这个插件虽然支持发送附件，**但仅适用于大小不超过 3MB 的文件**（这是调用 Microsoft Graph API 的最大请求大小）。

OAuth 2.0 的身份验证方式要求 Mail Integration for Office 365/Outlook 插件使用下面列出的4条信息向我们的邮件服务器请求一组密钥：

-   **Client ID:** 这让 Microsoft 知道要访问哪个身份验证服务器来对客户端进行身份验证
-   **Client Secret:** 这是链接到您的 Microsoft 帐户的唯一密钥，只能由受信任的客户端持有。它允许 Microsoft 知道客户端是真实的并且有权请求所请求资源的密钥（在这种情况下是您的 Microsoft 邮件服务器）
-   **Tenant ID:** 这是标识您的 Microsoft 帐户的唯一密钥。
-   **Redirect URI**: 这是认证成功后授权服务器将密钥返回到的位置。

**开始配置：**

**1\.** 在 WordPress 后台，前往 **Settings** > **Mail Integration 365 Settings** 进入插件设置页面，其他可以先不看，先把 **Redirect Uri** 一栏的内容复制到剪切板，并粘贴到其他地方，下面的步骤会用上它。

![Mail Integration 365 Settings](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Mail-Integration-365-Settings.webp?resize=860%2C546&ssl=1)

**2.** 使用前面创建好的用户（域名邮箱）[登录 Azure Active Directory](https://portal.azure.com/#home)，在顶部搜索栏中输入“应用注册”，点击搜索结果中出现的应用注册图标。

![Azure Active Directory 应用注册](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Azure-Active-Directory-%E5%BA%94%E7%94%A8%E6%B3%A8%E5%86%8C.webp?resize=860%2C304&ssl=1)

**3.** 点击 **\+ 新注册**。

![Azure Active Directory 应用注册 - 新注册](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Azure-Active-Directory-%E5%BA%94%E7%94%A8%E6%B3%A8%E5%86%8C-%E6%96%B0%E6%B3%A8%E5%86%8C.webp?resize=860%2C260&ssl=1)

**4.** 为应用程序（客户端）**命名**；勾选**仅此组织目录（单一租户）中的帐户**；重定向URI先不填，点击左下角**注册**按钮。

![Azure Active Directory - 1注册应用程序](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Azure-Active-Directory-1%E6%B3%A8%E5%86%8C%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F.webp?resize=750%2C890&ssl=1)

**5.** 接下来开始创建客户端密码：点击左侧边栏的**证书与密码**，然后点击右侧**客户端密码** tab 下的 **\+ 新客户端密码**。

![Azure Active Directory 创建客户端密码](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Azure-Active-Directory-%E5%88%9B%E5%BB%BA%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%AF%86%E7%A0%81.webp?resize=860%2C939&ssl=1)

**6.** 添加**说明**和**截止期限**（即设置密钥过期时间），然后点击左下角的**添加**按钮。

![Azure Active Directory - 添加客户端密码2](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Azure-Active-Directory-%E6%B7%BB%E5%8A%A0%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%AF%86%E7%A0%812.webp?resize=860%2C939&ssl=1)

**7.** 复制密钥**值**（请勿复制密钥ID）到安全的地方。密钥信息只会在 Azure Active Directory 的证书和机密部分显示一次，并且在重新加载页面后将无法再次复制。因此你需要立刻将密钥值复制到安全的地方，以备接下来的步骤使用。

注意：此密钥不应公开共享，必须安全保存，并且与受信任的客户共享。

**8.** 前往客户端**概述**页面获取客户端 ID 和租户ID。

![Azure Active Directory - 获取客户端ID和租户ID](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Azure-Active-Directory-%E8%8E%B7%E5%8F%96%E5%AE%A2%E6%88%B7%E7%AB%AFID%E5%92%8C%E7%A7%9F%E6%88%B7ID.webp?resize=860%2C750&ssl=1)

**9.** 将步骤7-8 获取的**客户端密钥**（Client Secret）、**客户端 ID**（Client ID）以及租户 ID（Tenant ID）信息复制到 WordPress 后台 Mail Integration 365 Settings 页面的对应区域，点击左下方的 **Save Changes** 按钮保存。

![Mail Integration 365 Settings 页面配置](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Mail-Integration-365-Settings-%E9%A1%B5%E9%9D%A2%E9%85%8D%E7%BD%AE.webp?resize=860%2C984&ssl=1)

**10\.** 回到步骤8所在的 Azure Active Directory 页面。我们需要设置应用程序平台和重定向URI。

要设置应用程序平台，依次点击**身份验证** > **\+ 添加平台**，选择 **Web**。

![Azure Active Directory 配置平台](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Azure-Active-Directory-%E9%85%8D%E7%BD%AE%E5%B9%B3%E5%8F%B0-scaled.webp?resize=860%2C550&ssl=1)

**11.** 将步骤1提到的重定向URI复制到**配置 Web**窗口，注销 URL 和 隐式授权选项可不填。点击**配置**按钮完成配置。

![Azure Active Directory - 配置Web](https://i0.wp.com/zblogs.top/wp-content/uploads/2022/04/Azure-Active-Directory-%E9%85%8D%E7%BD%AEWeb-scaled.webp?resize=860%2C550&ssl=1)

**12.** 最后，打开一个无痕浏览页面（匿名模式），需要前往 WordPress 后台的 Mail Integration 365 Settings 页面。如果你已严格按照上述步骤操作，你现在应该会看到一个“授权”按钮。单击此按钮后，你将被重定向到登录页面。你需要在其中输入所选 Microsoft 帐户（你希望从中发送电子邮件的帐户，即你前面创建好的域名邮箱）的登录详细信息。

> 为避免浏览器自动使用原先已保存的登录信息，最好在登录 WordPress 之前打开一个新的无痕浏览/隐身浏览器窗口。这将确保当您单击插件设置页面上的授权按钮时，系统会提示您输入要使用的帐户的凭据，而不是使用您自己保存的帐户凭据自动登录并将 WordPress 链接到错误的帐户。

---

插件配置步骤参考了 [Mail Integration for Office 365/Outlook](https://wordpress.org/plugins/mail-integration-365/) 官方[配置文档](https://crossconnected.co.uk/mail-integration-365-wordpress-plugin/)。

Credits: [](https://www.freepik.com/vectors/email)[Email illustration vector created by pch.vector – www.freepik.com](https://www.freepik.com/vectors/email-illustration)