---
title: Asistente de etiquetas de vínculo en ASP.NET Core
author: rick-anderson
ms.author: riande
description: Descubra los atributos del asistente de etiquetas de vínculo de ASP.NET Core y el papel que desempeña cada atributo al ampliar el comportamiento de la etiqueta de vínculo de código HTML.
ms.custom: mvc
ms.date: 09/24/2019
uid: mvc/views/tag-helpers/builtin-th/link-tag-helper
ms.openlocfilehash: e1e2e58b4ab9087e1f9de5b5c03b587feb88f1b9
ms.sourcegitcommit: fae6f0e253f9d62d8f39de5884d2ba2b4b2a6050
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/25/2019
ms.locfileid: "71256487"
---
# <a name="link-tag-helper-in-aspnet-core"></a>Asistente de etiquetas de vínculo en ASP.NET Core

Por [Rick Anderson](https://twitter.com/RickAndMSFT)

El [asistente de etiquetas de vínculo](xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper) genera un vínculo a un archivo CSS principal o de reserva. Normalmente, el archivo CSS principal se encuentra en una red [Content Delivery Network](/office365/enterprise/content-delivery-networks#what-exactly-is-a-cdn) (CDN).

[!INCLUDE[](~/includes/cdn.md)]

El asistente de etiquetas de vínculo permite especificar una red CDN para el archivo CSS y una reserva cuando dicha red no está disponible. El asistente de etiquetas de vínculo proporciona la ventaja de rendimiento de una red CDN con la solidez del hospedaje local.

El siguiente marcado de Razor muestra el elemento `head` de un archivo de diseño creado con la plantilla de aplicación web ASP.NET Core:

[!code-html[](link-tag-helper/sample/_Layout.cshtml?name=snippet)]

Lo siguiente es código HTML representado del código anterior (en un entorno que no es de desarrollo):

[!code-csharp[](link-tag-helper/sample/HtmlPage1.html)]

En el código anterior, el asistente de etiquetas de vínculo generó el elemento `<meta name="x-stylesheet-fallback-test" content="" class="sr-only" />` y el siguiente código JavaScript que se usa para comprobar el archivo *bootstrap.min.css* solicitado está disponible en la red CDN. En este caso, el archivo CSS estaba disponible, por lo que el asistente de etiquetas generó el elemento `<link />` con el archivo CSS de la red CDN.

## <a name="commonly-used-link-tag-helper-attributes"></a>Atributos del asistente de etiquetas de vínculo utilizados habitualmente

Consulte [Asistente de etiquetas de vínculo](xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper) para todos los atributos, propiedades y métodos del asistente de etiquetas de vínculo.

### <a name="href"></a>href

Dirección preferida del recurso vinculado. La dirección se pasa al código HTML generado en todos los casos.

### <a name="asp-fallback-href"></a>asp-fallback-href

La dirección URL de una hoja de estilos CSS en la que se va a realizar la reserva en caso de error en la dirección URL principal.

### <a name="asp-fallback-test-class"></a>asp-fallback-test-class

El nombre de clase definido en la hoja de estilos que se va a usar para la prueba de reserva. Para más información, consulte <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestClass>.

### <a name="asp-fallback-test-property"></a>asp-fallback-test-property

El nombre de la propiedad de CSS que se va a usar para la prueba de reserva. Para más información, consulte <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestProperty>.

### <a name="asp-fallback-test-value"></a>asp-fallback-test-value

El valor de la propiedad de CSS que se va a usar para la prueba de reserva. Para más información, consulte <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestValue>.

### <a name="asp-fallback-test-value"></a>asp-fallback-test-value

El valor de la propiedad de CSS que se va a usar para la prueba de reserva. Para obtener más información, vea <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestValue>

## <a name="additional-resources"></a>Recursos adicionales

* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/controllers/areas>
* <xref:razor-pages/index>
* <xref:mvc/compatibility-version>
