---
title: Activación de middleware con un contenedor de terceros en ASP.NET Core
author: guardrex
description: Aprenda a usar middleware fuertemente tipado con la implementación de una activación basada en Factory y un contenedor de terceros en ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 09/22/2019
uid: fundamentals/middleware/extensibility-third-party-container
ms.openlocfilehash: e54a2bd366457fa2d898b7ee26e95021aec5389b
ms.sourcegitcommit: d34b2627a69bc8940b76a949de830335db9701d3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/23/2019
ms.locfileid: "71187085"
---
# <a name="middleware-activation-with-a-third-party-container-in-aspnet-core"></a>Activación de middleware con un contenedor de terceros en ASP.NET Core

Por [Luke Latham](https://github.com/guardrex)

::: moniker range=">= aspnetcore-3.0"

En este artículo se explica cómo usar <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> y <xref:Microsoft.AspNetCore.Http.IMiddleware> como un punto de extensibilidad para la activación de [middleware](xref:fundamentals/middleware/index) con un contenedor de terceros. Para información de introducción sobre `IMiddlewareFactory` y `IMiddleware`, consulte <xref:fundamentals/middleware/extensibility>.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/middleware/extensibility-third-party-container/samples/) ([cómo descargarlo](xref:index#how-to-download-a-sample))

En la aplicación de ejemplo se muestra una activación de middleware por medio de una implementación de `IMiddlewareFactory`, `SimpleInjectorMiddlewareFactory`. En el ejemplo se usa el contenedor de inserción de dependencias [Simple Injector](https://simpleinjector.org).

La implementación de middleware del ejemplo registra el valor proporcionado por un parámetro de cadena de consulta (`key`). El middleware usa un contexto de base de datos insertado (un servicio con ámbito) para registrar el valor de cadena de consulta en una base de datos en memoria.

> [!NOTE]
> En la aplicación de ejemplo se usa [Simple Injector](https://github.com/simpleinjector/SimpleInjector) única y exclusivamente con fines de demostración. El uso de Simple Injector no está avalado. Los métodos de activación de middleware descritos en la documentación de Simple Injector y los problemas de GitHub está recomendado por los responsables de Simple Injector. Para más información, vea la [documentación de Simple Injector](https://simpleinjector.readthedocs.io/en/latest/index.html) y el [repositorio de GitHub de Simple Injector](https://github.com/simpleinjector/SimpleInjector).

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> proporciona métodos para crear middleware.

En la aplicación de ejemplo, se implementa un Middleware Factory para crear una instancia de `SimpleInjectorActivatedMiddleware`. Ese Middleware Factory usa el contenedor de Simple Injector para resolver el middleware:

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Middleware/SimpleInjectorMiddlewareFactory.cs?name=snippet1&highlight=5-8,12)]

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> define el middleware para la canalización de solicitudes de la aplicación.

Middleware activado por una implementación de `IMiddlewareFactory` (*Middleware/SimpleInjectorActivatedMiddleware.cs*):

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Middleware/SimpleInjectorActivatedMiddleware.cs?name=snippet1)]

Se crea una extensión para el middleware (*Middleware/MiddlewareExtensions.cs*):

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Middleware/MiddlewareExtensions.cs?name=snippet1)]

`Startup.ConfigureServices` debe realizar varias tareas:

* Configurar el contenedor de Simple Injector.
* Registrar tanto el Factory como el Middleware.
* Poner disponible el contexto de base de datos de la aplicación en el contenedor de Simple Injector.

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Startup.cs?name=snippet1)]

El middleware se registra en la canalización de procesamiento de solicitudes en `Startup.Configure`:

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Startup.cs?name=snippet2&highlight=12)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

En este artículo se explica cómo usar <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> y <xref:Microsoft.AspNetCore.Http.IMiddleware> como un punto de extensibilidad para la activación de [middleware](xref:fundamentals/middleware/index) con un contenedor de terceros. Para información de introducción sobre `IMiddlewareFactory` y `IMiddleware`, consulte <xref:fundamentals/middleware/extensibility>.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/middleware/extensibility-third-party-container/samples/) ([cómo descargarlo](xref:index#how-to-download-a-sample))

En la aplicación de ejemplo se muestra una activación de middleware por medio de una implementación de `IMiddlewareFactory`, `SimpleInjectorMiddlewareFactory`. En el ejemplo se usa el contenedor de inserción de dependencias [Simple Injector](https://simpleinjector.org).

La implementación de middleware del ejemplo registra el valor proporcionado por un parámetro de cadena de consulta (`key`). El middleware usa un contexto de base de datos insertado (un servicio con ámbito) para registrar el valor de cadena de consulta en una base de datos en memoria.

> [!NOTE]
> En la aplicación de ejemplo se usa [Simple Injector](https://github.com/simpleinjector/SimpleInjector) única y exclusivamente con fines de demostración. El uso de Simple Injector no está avalado. Los métodos de activación de middleware descritos en la documentación de Simple Injector y los problemas de GitHub está recomendado por los responsables de Simple Injector. Para más información, vea la [documentación de Simple Injector](https://simpleinjector.readthedocs.io/en/latest/index.html) y el [repositorio de GitHub de Simple Injector](https://github.com/simpleinjector/SimpleInjector).

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> proporciona métodos para crear middleware.

En la aplicación de ejemplo, se implementa un Middleware Factory para crear una instancia de `SimpleInjectorActivatedMiddleware`. Ese Middleware Factory usa el contenedor de Simple Injector para resolver el middleware:

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/SimpleInjectorMiddlewareFactory.cs?name=snippet1&highlight=5-8,12)]

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> define el middleware para la canalización de solicitudes de la aplicación.

Middleware activado por una implementación de `IMiddlewareFactory` (*Middleware/SimpleInjectorActivatedMiddleware.cs*):

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/SimpleInjectorActivatedMiddleware.cs?name=snippet1)]

Se crea una extensión para el middleware (*Middleware/MiddlewareExtensions.cs*):

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/MiddlewareExtensions.cs?name=snippet1)]

`Startup.ConfigureServices` debe realizar varias tareas:

* Configurar el contenedor de Simple Injector.
* Registrar tanto el Factory como el Middleware.
* Poner disponible el contexto de base de datos de la aplicación en el contenedor de Simple Injector.

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

El middleware se registra en la canalización de procesamiento de solicitudes en `Startup.Configure`:

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Startup.cs?name=snippet2&highlight=12)]

::: moniker-end

## <a name="additional-resources"></a>Recursos adicionales

* [Middleware](xref:fundamentals/middleware/index)
* [Factory-based middleware activation](xref:fundamentals/middleware/extensibility) (Activación de middleware basada en Factory)
* [Repositorio de GitHub de Simple Injector](https://github.com/simpleinjector/SimpleInjector)
* [Documentación de Simple Injector](https://simpleinjector.readthedocs.io/en/latest/index.html)
