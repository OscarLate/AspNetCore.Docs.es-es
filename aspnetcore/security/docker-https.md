---
title: Hospedaje de imágenes de ASP.NET Core con Docker a través de HTTPS
author: rick-anderson
description: Aprenda a hospedar imágenes de ASP.NET Core con Docker a través de HTTPS
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/05/2019
uid: security/docker-https
ms.openlocfilehash: c13ba02845eef5c53a939feec2be8a01bc4ca128
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/18/2019
ms.locfileid: "71082529"
---
# <a name="hosting-aspnet-core-images-with-docker-over-https"></a>Hospedaje de imágenes de ASP.NET Core con Docker a través de HTTPS

Por [Rick Anderson](https://twitter.com/RickAndMSFT)

[De forma predeterminada](/aspnet/core/security/enforcing-ssl)ASP.net Core usa https. [Https](https://en.wikipedia.org/wiki/HTTPS) se basa en los [certificados](https://en.wikipedia.org/wiki/Public_key_certificate) de confianza, identidad y cifrado.

En este documento se explica cómo ejecutar imágenes de contenedor creadas previamente con HTTPS.

Consulte [desarrollo de aplicaciones ASP.net Core con Docker a través de https](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/aspnetcore-docker-https-development.md) para escenarios de desarrollo.

Este ejemplo requiere [docker 17,06](https://docs.docker.com/release-notes/docker-ce) o posterior del [cliente de Docker](https://www.docker.com/products/docker).

## <a name="prerequisites"></a>Requisitos previos

El [SDK de .net Core 2,2](https://www.microsoft.com/net/download) o posterior es necesario para algunas de las instrucciones de este documento.

## <a name="certificates"></a>Certificados

Se requiere un certificado de una [entidad de certificación](https://en.wikipedia.org/wiki/Certificate_authority) para el [hospedaje de producción](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) en un dominio.  [Vamos a cifrar](https://letsencrypt.org/) es una entidad de certificación que ofrece certificados gratuitos.

En este documento se usan [certificados de desarrollo autofirmados](https://en.wikipedia.org/wiki/Self-signed_certificate) para hospedar imágenes `localhost`pregeneradas en. Las instrucciones son similares a usar certificados de producción.

Para los certificados de producción:

* La `dotnet dev-certs` herramienta no es necesaria.
* No es necesario almacenar los certificados en la ubicación usada en las instrucciones. Cualquier ubicación debería funcionar, aunque no se recomienda almacenar certificados en el directorio del sitio.

El volumen de instrucciones monta los certificados en contenedores. Puede agregar certificados a las imágenes de contenedor con `COPY` un comando en un Dockerfile. No se recomienda copiar certificados en una imagen:

* Dificulta el uso de la misma imagen para realizar pruebas con certificados de desarrollador.
* Dificulta el uso de la misma imagen para hospedar con certificados de producción.
* Existe un riesgo significativo de divulgación de certificados.

## <a name="running-pre-built-container-images-with-https"></a>Ejecución de imágenes de contenedor pregeneradas con HTTPS

Utilice las siguientes instrucciones para la configuración del sistema operativo.

### <a name="windows-using-linux-containers"></a>Windows con contenedores de Linux

Generar certificado y configurar equipo local:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

En los comandos anteriores, reemplace `{ password here }` por una contraseña.

Ejecute la imagen de contenedor con ASP.NET Core configurado para HTTPS:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

La contraseña debe coincidir con la contraseña usada para el certificado.

### <a name="macos-or-linux"></a>macOS o Linux

Generar certificado y configurar equipo local:

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

`dotnet dev-certs https --trust`solo se admite en macOS y Windows. Debe confiar en los certificados en Linux de la manera que sea compatible con el distribución. Es probable que necesite confiar en el certificado en el explorador.

En los comandos anteriores, reemplace `{ password here }` por una contraseña.

Ejecute la imagen de contenedor con ASP.NET Core configurado para HTTPS:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

La contraseña debe coincidir con la contraseña usada para el certificado.

### <a name="windows-using-windows-containers"></a>Windows con contenedores de Windows

Generar certificado y configurar equipo local:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

En los comandos anteriores, reemplace `{ password here }` por una contraseña.

Ejecute la imagen de contenedor con ASP.NET Core configurado para HTTPS:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=\https\aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:C:\https\ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

La contraseña debe coincidir con la contraseña usada para el certificado.