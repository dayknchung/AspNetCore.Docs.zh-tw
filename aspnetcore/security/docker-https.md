---
title: 透過 HTTPS 使用 Docker 裝載 ASP.NET Core 映射
author: rick-anderson
description: 瞭解如何透過 HTTPS 使用 Docker 裝載 ASP.NET Core 映射
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/05/2019
no-loc:
- Let's Encrypt
uid: security/docker-https
ms.openlocfilehash: 2f338e8883ca926c0f9a7ab339f58b088151cc87
ms.sourcegitcommit: 235623b6e5a5d1841139c82a11ac2b4b3f31a7a9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/08/2020
ms.locfileid: "77089197"
---
# <a name="hosting-aspnet-core-images-with-docker-over-https"></a>透過 HTTPS 使用 Docker 裝載 ASP.NET Core 映射

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core 預設會使用[HTTPS](/aspnet/core/security/enforcing-ssl)。 [HTTPS](https://en.wikipedia.org/wiki/HTTPS)依賴[憑證](https://en.wikipedia.org/wiki/Public_key_certificate)來進行信任、身分識別和加密。

本檔說明如何使用 HTTPS 來執行預先建立的容器映射。

如需開發案例，請參閱[使用 Docker OVER HTTPS 開發 ASP.NET Core 應用程式](https://github.com/dotnet/dotnet-docker/blob/master/samples/run-aspnetcore-https-development.md)。

此範例需要 docker [17.06](https://docs.docker.com/release-notes/docker-ce)或更新版本的[docker 用戶端](https://www.docker.com/products/docker)。

## <a name="prerequisites"></a>Prerequisites

本檔中的部分指示需要[.Net Core 2.2 SDK](https://www.microsoft.com/net/download)或更新版本。

## <a name="certificates"></a>憑證

需要[憑證授權單位](https://wikipedia.org/wiki/Certificate_authority)單位的憑證，才能針對網域進行[生產環境裝載](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/)。 [Let's Encrypt](https://letsencrypt.org/)是提供免費憑證的憑證授權單位單位。

本檔使用[自我簽署的開發憑證](https://en.wikipedia.org/wiki/Self-signed_certificate)，透過 `localhost`來裝載預先建立的映射。 這些指示與使用生產憑證類似。

針對生產憑證：

* 不需要 `dotnet dev-certs` 工具。
* 憑證不需要儲存在指示所使用的位置。 任何位置都應該可行，雖然不建議在您的網站目錄中儲存憑證。

下列章節中包含的指示會使用 Docker 的 `-v` 命令列選項，將憑證掛接到容器中。 您可以使用*Dockerfile*中的 `COPY` 命令，將憑證新增至容器映射，但不建議這麼做。 基於下列原因，不建議將憑證複製到映射：

* 使用相同的映射來測試開發人員憑證並不容易。
* 使用相同的映射來裝載實際執行憑證很容易。
* 憑證洩漏有嚴重的風險。

## <a name="running-pre-built-container-images-with-https"></a>使用 HTTPS 執行預先建立的容器映射

針對您的作業系統設定，請使用下列指示。

### <a name="windows-using-linux-containers"></a>使用 Linux 容器的 Windows

產生憑證並設定本機電腦：

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

在上述命令中，將 `{ password here }` 取代為密碼。

使用針對 HTTPS 設定的 ASP.NET Core 來執行容器映射：

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

密碼必須符合用於憑證的密碼。

### <a name="macos-or-linux"></a>macOS 或 Linux

產生憑證並設定本機電腦：

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

只有在 macOS 和 Windows 上才支援 `dotnet dev-certs https --trust`。 您必須以散發版本支援的方式信任 Linux 上的憑證。 您很可能需要信任您瀏覽器中的憑證。

在上述命令中，將 `{ password here }` 取代為密碼。

使用針對 HTTPS 設定的 ASP.NET Core 來執行容器映射：

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

密碼必須符合用於憑證的密碼。

### <a name="windows-using-windows-containers"></a>使用 Windows 容器的 windows

產生憑證並設定本機電腦：

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

在上述命令中，將 `{ password here }` 取代為密碼。

使用針對 HTTPS 設定的 ASP.NET Core 來執行容器映射：

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=\https\aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:C:\https\ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

密碼必須符合用於憑證的密碼。
