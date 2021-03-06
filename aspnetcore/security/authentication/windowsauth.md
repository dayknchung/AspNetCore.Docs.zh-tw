---
title: 在 ASP.NET Core 中設定 Windows 驗證
author: scottaddie
description: 了解如何設定 ASP.NET Core 中的 Windows 驗證的 IIS 和 HTTP.sys。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 07/01/2019
uid: security/authentication/windowsauth
ms.openlocfilehash: 30f1f554a29412ed6b84115d457d2da1aba91c17
ms.sourcegitcommit: eb3e51d58dd713eefc242148f45bd9486be3a78a
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2019
ms.locfileid: "67500508"
---
# <a name="configure-windows-authentication-in-aspnet-core"></a>在 ASP.NET Core 中設定 Windows 驗證

作者：[Scott Addie](https://twitter.com/Scott_Addie) 和 [Luke Latham](https://github.com/guardrex)

::: moniker range=">= aspnetcore-3.0"

Windows 驗證 （也稱為交涉、 Kerberos 或 NTLM 驗證） 可以設定與裝載的 ASP.NET Core 應用程式[IIS](xref:host-and-deploy/iis/index)， [Kestrel](xref:fundamentals/servers/kestrel)，或[HTTP.sys](xref:fundamentals/servers/httpsys).

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Windows 驗證 （也稱為交涉、 Kerberos 或 NTLM 驗證） 可以設定與裝載的 ASP.NET Core 應用程式[IIS](xref:host-and-deploy/iis/index)或是[HTTP.sys](xref:fundamentals/servers/httpsys)。

::: moniker-end

Windows 驗證仰賴作業系統來驗證 ASP.NET Core 應用程式的使用者。 當您的伺服器在公司網路上執行時，您可以透過 Active Directory 網域身分識別進行 Windows 驗證或透過 Windows 帳戶來識別使用者。 Windows 驗證最適合用於使用者、用戶端應用程式與 Web 伺服器皆屬於相同 Windows 網域的內部網路環境。

> [!NOTE]
> 使用 HTTP/2，不支援 Windows 驗證。 可以傳送 HTTP/2 回應的驗證挑戰，但用戶端驗證之前，必須降級為 HTTP/1.1。

## <a name="iisiis-express"></a>IIS/IIS Express

加入驗證服務所叫用<xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*>(<xref:Microsoft.AspNetCore.Server.IISIntegration?displayProperty=fullName>命名空間) 中`Startup.ConfigureServices`:

```csharp
services.AddAuthentication(IISDefaults.AuthenticationScheme);
```

### <a name="launch-settings-debugger"></a>啟動設定 （偵錯工具）

啟動設定的設定只會影響*Properties/launchSettings.json*適用於 IIS Express 檔案，並不會設定 IIS 的 Windows 驗證。 伺服器組態中會說明[IIS](#iis)一節。

**Web 應用程式**可透過 Visual Studio 或.NET Core CLI 的範本可以設定為支援 Windows 驗證，這會更新*Properties/launchSettings.json*檔案自動的。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

**新的專案**

1. 建立新的專案。
1. 選取 [ASP.NET Core Web 應用程式]  。 選取 [下一步]  。
1. 提供的名稱**專案名稱**欄位。 確認**位置**項目是否正確，或提供專案的位置。 選取 [建立]  。
1. 選取 **變更**下方**驗證**。
1. 在 **變更驗證**視窗中，選取**Windows 驗證**。 選取 [確定]  。
1. 選取 [Web 應用程式]  。
1. 選取 [建立]  。

執行應用程式。 使用者名稱會出現在呈現的應用程式使用者介面。

**現有專案**

專案的屬性會啟用 Windows 驗證，並停用匿名驗證：

1. 以滑鼠右鍵按一下 [方案總管]  中的專案，然後選取 [屬性]  。
1. 選取 [偵錯]  索引標籤。
1. 清除核取方塊**啟用匿名驗證**。
1. 選取核取方塊**啟用 Windows 驗證**。
1. 儲存並關閉 [屬性] 頁面。

或者，設定屬性，在`iisSettings`的節點*launchSettings.json*檔案：

[!code-json[](windowsauth/sample_snapshot/launchSettings.json?highlight=2-3)]

# <a name="visual-studio-code--net-core-clitabvisual-studio-codenetcore-cli"></a>[Visual Studio Code / .NET Core CLI](#tab/visual-studio-code+netcore-cli)

**新的專案**

執行[dotnet 新](/dotnet/core/tools/dotnet-new)命令搭配`webapp`引數 （ASP.NET Core Web 應用程式） 和`--auth Windows`切換：

```console
dotnet new webapp --auth Windows
```

**現有專案**

更新`iisSettings`的節點*launchSettings.json*檔案：

[!code-json[](windowsauth/sample_snapshot/launchSettings.json?highlight=2-3)]

---

當修改現有的專案，請確認專案檔包含的套件參考[Microsoft.AspNetCore.App 中繼套件](xref:fundamentals/metapackage-app)**或是** [Microsoft.AspNetCore.Authentication](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication/) NuGet 套件。

### <a name="iis"></a>IIS

IIS 會使用[ASP.NET Core 模組](xref:host-and-deploy/aspnet-core-module)主機 ASP.NET Core 應用程式。 Windows 驗證針對透過 IIS *web.config*檔案。 下列各節將示範如何：

* 提供本機*web.config*部署應用程式時，請在伺服器啟動 Windows 驗證的檔案。
* 使用 IIS 管理員設定*web.config*已經部署到伺服器的 ASP.NET Core 應用程式的檔案。

如果您尚未這麼做，請啟用 IIS 可裝載 ASP.NET Core 應用程式。 如需詳細資訊，請參閱<xref:host-and-deploy/iis/index>。

啟用 Windows 驗證的 IIS 角色服務。 如需詳細資訊，請參閱 <<c0> [ 啟用 IIS 角色服務 （請參閱步驟 2） 中的 Windows 驗證](xref:host-and-deploy/iis/index#iis-configuration)。

[IIS Integration 中介軟體](xref:host-and-deploy/iis/index#enable-the-iisintegration-components)預設設定來自動驗證要求。 如需詳細資訊，請參閱[裝載 ASP.NET Core 與 IIS 的 Windows 上：IIS 選項 (AutomaticAuthentication)](xref:host-and-deploy/iis/index#iis-options)。

ASP.NET Core 模組預設設定為轉送至應用程式的 Windows 驗證語彙基元。 如需詳細資訊，請參閱[ASP.NET Core 模組組態參考：AspNetCore 元素的屬性](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element)。

使用**任一**下列其中一個方法：

* **發行和部署專案，再**新增下列*web.config*至專案根目錄的檔案：

  [!code-xml[](windowsauth/sample_snapshot/web_2.config)]

  當專案發行.NET Core SDK (不含`<IsTransformWebConfigDisabled>`屬性設定為`true`專案檔中)，發行*web.config*檔案包含`<location><system.webServer><security><authentication>`一節。 如需詳細資訊`<IsTransformWebConfigDisabled>`屬性，請參閱<xref:host-and-deploy/iis/index#webconfig-file>。

* **發行和部署專案之後,** 執行伺服器端設定使用 IIS 管理員：

  1. 在 [IIS 管理員] 中，選取 IIS 站台之下**站台**節點**連線**資訊看板。
  1. 按兩下**驗證**中**IIS**區域。
  1. 選取 **匿名驗證**。 選取 **停用**中**動作**資訊看板。
  1. 選取  **Windows 驗證**。 選取 **啟用**中**動作**資訊看板。

  IIS 管理員在採取這些動作，會修改應用程式的*web.config*檔案。 A`<system.webServer><security><authentication>`節點新增與更新的設定，如`anonymousAuthentication`和`windowsAuthentication`:

  [!code-xml[](windowsauth/sample_snapshot/web_1.config?highlight=4-5)]

  `<system.webServer>`區段新增至*web.config*由 IIS 管理員中的檔案超出應用程式的`<location>`發佈應用程式時，由.NET Core SDK 加入的區段。 因為區段會新增外部`<location>`節點，設定會由任何繼承[子應用程式](xref:host-and-deploy/iis/index#sub-applications)目前的應用程式。 若要防止繼承，移動加入`<security>`區段內的`<location><system.webServer>`.NET Core SDK 提供的一節。

  將 IIS 設定使用 IIS 管理員時，它只會影響應用程式的*web.config*伺服器上的檔案。 後續部署應用程式可能會覆寫伺服器上的設定，如果伺服器的複本*web.config*專案的取代*web.config*檔案。 使用**任一**下列其中一個方法來管理設定：

  * 使用 IIS 管理員中的設定重設*web.config*檔案之後部署上覆寫該檔案。
  * 新增*web.config 檔案*應用程式在本機使用的設定。

::: moniker range=">= aspnetcore-3.0"

## <a name="kestrel"></a>Kestrel

 [Microsoft.AspNetCore.Authentication.Negotiate](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Negotiate) NuGet 套件可以搭配[Kestrel](xref:fundamentals/servers/kestrel)以支援在 Windows、 Linux 和 macOS 上使用 Negotiate、 Kerberos 和 NTLM Windows 驗證。

> [!WARNING]
> 認證可以保存在連接上的要求。 *交涉驗證必須不使用 proxy 使用，除非 proxy 會使用 Kestrel 的 1 對 1 連接同質 （持續連線）。*

> [!NOTE]
> 如果基礎伺服器以原生方式支援 Windows 驗證，而且如果已啟用，會偵測到的交涉處理常式。 如果伺服器支援 Windows 驗證，但已停用，詢問是否要啟用伺服器實作擲回錯誤。 在伺服器中啟用 Windows 驗證時，Negotiate 處理常式無障礙地轉送給它。

 加入驗證服務所叫用<xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*>(`Microsoft.AspNetCore.Authentication.Negotiate`命名空間) 和`AddNegotitate`(`Microsoft.AspNetCore.Authentication.Negotiate`命名空間) 中`Startup.ConfigureServices`:

 ```csharp
services.AddAuthentication(NegotiateDefaults.AuthenticationScheme)
    .AddNegotiate();
```

新增驗證中介軟體，藉由呼叫<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*>在`Startup.Configure`:

 ```csharp
app.UseAuthentication();

app.UseMvc();
```

如需有關中介軟體的詳細資訊，請參閱<xref:fundamentals/middleware/index>。

允許匿名要求。 使用[ASP.NET Core 授權](xref:security/authorization/introduction)挑戰驗證的匿名要求。

### <a name="windows-environment-configuration"></a>Windows 環境設定

[Microsoft.AspNetCore.Authentication.Negotiate](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Negotiate)元件會執行使用者模式驗證。 服務主體名稱 (Spn) 必須將執行服務，而不是電腦帳戶的使用者帳戶。 執行`setspn -S HTTP/mysrevername.mydomain.com myuser`在系統管理命令介面中。

### <a name="linux-and-macos-environment-configuration"></a>Linux 和 macOS 的環境設定

加入 Windows 網域中的 Linux 或 macOS 機器的指示位於[到 SQL Server 使用 Windows 驗證 Kerberos 連接 Azure Data Studio](/sql/azure-data-studio/enable-kerberos?view=sql-server-2017#join-your-os-to-the-active-directory-domain-controller)文章。 指示在網域上建立的 Linux 機器的機器帳戶。 Spn 必須新增至該電腦帳戶。

> [!NOTE]
> 遵循中的指導方針時[到 SQL Server 使用 Windows 驗證 Kerberos 連接 Azure Data Studio](/sql/azure-data-studio/enable-kerberos?view=sql-server-2017#join-your-os-to-the-active-directory-domain-controller)文章中，取代`python-software-properties`使用`python3-software-properties`如有需要。

一旦在 Linux 或 macOS 機器已加入網域，需要額外的步驟，以提供[keytab 檔案](https://blogs.technet.microsoft.com/pie/2018/01/03/all-you-need-to-know-about-keytab-files/)利用 Spn:

* 在網域控制站，用電腦帳戶來加入新的 web 服務的 Spn:
  * `setspn -S HTTP/mywebservice.mydomain.com mymachine`
  * `setspn -S HTTP/mywebservice@MYDOMAIN.COM mymachine`
* 使用[ktpass](/windows-server/administration/windows-commands/ktpass)產生 keytab 檔案：
  * `ktpass -princ HTTP/mywebservice.mydomain.com@MYDOMAIN.COM -pass myKeyTabFilePassword -mapuser MYDOMAIN\mymachine$ -pType KRB5_NT_PRINCIPAL -out c:\temp\mymachine.HTTP.keytab -crypto AES256-SHA1`
  * 某些欄位中必須指定大寫所示。
* 將 keytab 檔案複製到 Linux 或 macOS 電腦。
* 選取透過環境變數的 keytab 檔案： `export KRB5_KTNAME=/tmp/mymachine.HTTP.keytab`
* 叫用`klist`以顯示目前可供使用的 Spn。

> [!NOTE]
> Keytab 檔案包含網域存取認證，而且必須據此加以保護。

::: moniker-end

## <a name="httpsys"></a>HTTP.sys

[HTTP.sys](xref:fundamentals/servers/httpsys)支援使用 Negotiate、 NTLM、 或基本驗證的核心模式 Windows 驗證。

加入驗證服務所叫用<xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*>(<xref:Microsoft.AspNetCore.Server.HttpSys?displayProperty=fullName>命名空間) 中`Startup.ConfigureServices`:

```csharp
services.AddAuthentication(HttpSysDefaults.AuthenticationScheme);
```

使用 Windows 驗證使用 HTTP.sys 的應用程式的 web 主機設定 (*Program.cs*)。 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> 處於<xref:Microsoft.AspNetCore.Server.HttpSys?displayProperty=fullName>命名空間。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](windowsauth/sample_snapshot/Program_GenericHost.cs?highlight=13-19)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](windowsauth/sample_snapshot/Program_WebHost.cs?highlight=9-15)]

::: moniker-end

> [!NOTE]
> HTTP.sys 使用 Kerberos 驗證通訊協定委派給核心模式驗證。 Kerberos 和 HTTP.sys 不支援使用者模式驗證。 必須使用電腦帳戶來解密 Kerberos 權杖/票證，該權杖/票證取自 Active Directory，並由用戶端將其轉送至伺服器來驗證使用者。 請註冊主機的服務主體名稱 (SPN)，而非應用程式的使用者。

> [!NOTE]
> HTTP.sys 不支援 Nano Server 1709 版或更新版本上。 若要使用 Windows 驗證和 HTTP.sys 使用 Nano Server，請使用[Server Core (microsoft/windowsservercore) 容器](https://hub.docker.com/r/microsoft/windowsservercore/)。 如需有關 Server Core 的詳細資訊，請參閱[什麼是 Windows Server 中的 Server Core 安裝選項？](/windows-server/administration/server-core/what-is-server-core)。

## <a name="authorize-users"></a>授權使用者

匿名存取的設定狀態決定的方式`[Authorize]`和`[AllowAnonymous]`應用程式中使用屬性。 下列兩節會說明如何處理不允許和允許設定狀態的匿名存取。

### <a name="disallow-anonymous-access"></a>不允許匿名存取

當您啟用 Windows 驗證，並已停用匿名存取，`[Authorize]`和`[AllowAnonymous]`屬性沒有任何作用。 如果 IIS 站台設定為不允許匿名存取，要求永遠不會到達應用程式。 基於這個理由，`[AllowAnonymous]`屬性不適用。

### <a name="allow-anonymous-access"></a>允許匿名存取

當啟用 Windows 驗證和匿名存取時，使用`[Authorize]`和`[AllowAnonymous]`屬性。 `[Authorize]`屬性可讓您保護應用程式需要驗證的端點。 `[AllowAnonymous]`屬性覆寫`[Authorize]`允許匿名存取的應用程式中的屬性。 屬性使用方式詳細資料，請參閱<xref:security/authorization/simple>。

> [!NOTE]
> 根據預設，缺少授權，才能存取頁面的使用者會看到空的 HTTP 403 回應。 [StatusCodePages 中介軟體](xref:fundamentals/error-handling#usestatuscodepages)可以設定為使用者提供更好的 「 拒絕存取 」 體驗。

## <a name="impersonation"></a>模擬

ASP.NET Core 不會實作模擬。 應用程式執行的所有要求，使用應用程式集區或處理序身分識別的應用程式的身分識別。 如果應用程式應該執行代表使用者的動作，使用[WindowsIdentity.RunImpersonated](xref:System.Security.Principal.WindowsIdentity.RunImpersonated*)中[終端機內嵌中介軟體](xref:fundamentals/middleware/index#create-a-middleware-pipeline-with-iapplicationbuilder)在`Startup.Configure`。 在此內容中執行單一動作，然後關閉 內容。

[!code-csharp[](windowsauth/sample_snapshot/Startup.cs?highlight=10-19)]

`RunImpersonated` 不支援非同步作業，而不應該用於複雜的案例。 比方說，包裝整個要求或中介軟體鏈結不支援或建議。

::: moniker range=">= aspnetcore-3.0"

雖然[Microsoft.AspNetCore.Authentication.Negotiate](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Negotiate)套件會啟用 Windows 驗證，Windows 才支援 Linux 和 macOS 的模擬。

::: moniker-end

## <a name="claims-transformations"></a>宣告轉換

::: moniker range=">= aspnetcore-3.0"

當使用 IIS 時，裝載<xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*>不在內部呼叫以初始化使用者。 因此，預設會在未啟動每個驗證之後，使用 <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> 實作來轉換宣告。 如需宣告轉換就會啟動的程式碼範例和詳細資訊，請參閱<xref:host-and-deploy/aspnet-core-module#in-process-hosting-model>。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

當 IIS 同處理序模式中，裝載<xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*>不在內部呼叫以初始化使用者。 因此，預設會在未啟動每個驗證之後，使用 <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> 實作來轉換宣告。 如需裝載同處理序時，會啟用宣告轉換的程式碼範例和詳細資訊，請參閱<xref:host-and-deploy/aspnet-core-module#in-process-hosting-model>。

::: moniker-end

## <a name="additional-resources"></a>其他資源

* [dotnet publish](/dotnet/core/tools/dotnet-publish)
* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/visual-studio-publish-profiles>
