---
title: Middleware de ASP.NET Core
author: rick-anderson
description: Obtenga información sobre el middleware de ASP.NET Core y la canalización de solicitudes.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/08/2019
uid: fundamentals/middleware/index
ms.openlocfilehash: 8f5c3aabf17e78ae9675048602317c54f08e82a7
ms.sourcegitcommit: 7d3c6565dda6241eb13f9a8e1e1fd89b1cfe4d18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/11/2019
ms.locfileid: "72259818"
---
# <a name="aspnet-core-middleware"></a>Middleware de ASP.NET Core

Por [Rick Anderson](https://twitter.com/RickAndMSFT) y [Steve Smith](https://ardalis.com/)

El software intermedio es un software que se ensambla en una canalización de una aplicación para controlar las solicitudes y las respuestas. Cada componente puede hacer lo siguiente:

* Elegir si se pasa la solicitud al siguiente componente de la canalización.
* Realizar trabajos antes y después del siguiente componente de la canalización.

Los delegados de solicitudes se usan para crear la canalización de solicitudes. Estos también controlan las solicitudes HTTP.

Los delegados de solicitudes se configuran con los métodos de extensión <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run*>, <xref:Microsoft.AspNetCore.Builder.MapExtensions.Map*> y <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use*>. Un delegado de solicitudes se puede especificar en línea como un método anónimo (denominado middleware en línea) o se puede definir en una clase reutilizable. Estas clases reutilizables y métodos anónimos en línea se conocen como *software intermedio* o *componentes de software intermedio*. Cada componente de software intermedio de la canalización de solicitudes es responsable de invocar el siguiente componente de la canalización o de cortocircuitar la canalización, en caso de ser necesario. Cuando un middleware se cortocircuita, se llama *middleware de terminal* porque impide el procesamiento de la solicitud por parte de middleware adicional.

En <xref:migration/http-modules> se explica la diferencia entre las canalizaciones de solicitudes en ASP.NET Core y ASP.NET 4.x y se proporcionan ejemplos adicionales de middleware.

## <a name="create-a-middleware-pipeline-with-iapplicationbuilder"></a>Creación de una canalización de software intermedio con IApplicationBuilder

La canalización de solicitudes de ASP.NET Core consiste en una secuencia de delegados de solicitud a los que se llama de uno en uno. En el siguiente diagrama se muestra este concepto. El subproceso de ejecución sigue las flechas negras.

![En el patrón de procesamiento de solicitudes se muestra una solicitud entrante, el procesamiento a través de tres softwares intermedios y la respuesta saliente de la aplicación. Cada middleware ejecuta su lógica y pasa la solicitud al siguiente middleware con la instrucción next(). Después de que el tercer software intermedio procese la solicitud, esta vuelve a pasar por los dos softwares intermedios anteriores en orden inverso. Esto se hace a modo de procesamiento adicional después de las instrucciones next() y antes de salir de la aplicación como respuesta al cliente.](index/_static/request-delegate-pipeline.png)

Cada delegado puede realizar operaciones antes y después del siguiente. Los delegados que controlan excepciones deben llamarse al principio de la canalización para que puedan capturar las excepciones que se producen en las fases siguientes de la canalización.

La aplicación ASP.NET Core más sencilla posible configura un solo delegado de solicitudes que controla todas las solicitudes. En este caso no se incluye una canalización de solicitudes real. En su lugar, solo se llama a una única función anónima en respuesta a todas las solicitudes HTTP.

[!code-csharp[](index/snapshot/Middleware/Startup.cs)]

El primer delegado de <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run*> finaliza la canalización.

Encadene varios delegados de solicitudes con <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use*>. El parámetro `next` representa el siguiente delegado de la canalización. Si *no* llama al *siguiente* parámetro, puede cortocircuitar la canalización. Normalmente, puede realizar acciones antes y después del siguiente delegado, tal como se muestra en el ejemplo siguiente:

[!code-csharp[](index/snapshot/Chain/Startup.cs)]

Cuando un delegado no pasa una solicitud al siguiente delegado, se denomina *cortocircuitar la canalización de solicitudes*. Este proceso es necesario muchas veces, ya que previene la realización de trabajo innecesario. Por ejemplo, el [middleware de archivos estáticos](xref:fundamentals/static-files) puede actuar como *middleware de terminal* procesando una solicitud para un archivo estático y cortocircuitando el resto de la canalización. El middleware agregado a la canalización antes del middleware que finaliza el procesamiento sigue procesando código después de sus instrucciones `next.Invoke`. Sin embargo, consulte la siguiente advertencia sobre intentar escribir en una respuesta que ya se ha enviado.

> [!WARNING]
> No llame a `next.Invoke` después de haber enviado la respuesta al cliente. Si se modifica <xref:Microsoft.AspNetCore.Http.HttpResponse> después de haber iniciado la respuesta, se producirá una excepción. Por ejemplo, se producirá una excepción al realizar cambios tales como el establecimiento de encabezados o el código. Si escribe en el cuerpo de la respuesta después de llamar a `next`:
>
> * Puede provocar una infracción del protocolo. Por ejemplo, si escribe más de la longitud `Content-Length` establecida.
> * Puede dañar el formato del cuerpo. Por ejemplo, si escribe un pie de página en HTML en un archivo CSS.
>
> <xref:Microsoft.AspNetCore.Http.HttpResponse.HasStarted*> es una sugerencia útil para indicar si se han enviado los encabezados o se han realizado escrituras en el cuerpo.

## <a name="order"></a>Ordenar

El orden en el que se agregan los componentes de software intermedio en el método `Startup.Configure` define el orden en el que se invocarán los componentes de software intermedio en las solicitudes y el orden inverso de la respuesta. Por motivos de seguridad, rendimiento y funcionalidad, el orden es básico.

El siguiente método `Startup.Configure` agrega los componentes de middleware para escenarios de aplicaciones comunes:

::: moniker range=">= aspnetcore-3.0"

1. Control de errores y excepciones
   * Cuando la aplicación se ejecuta en el entorno de desarrollo:
     * El middleware de la página de excepciones para el desarrollador (<xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage*>) informa los errores en tiempo de ejecución de la aplicación.
     * El middleware de la página de errores de la base de datos informa de los errores en tiempo de ejecución de la base de datos.
   * Cuando la aplicación se ejecuta en el entorno de producción:
     * El middleware del controlador de excepciones (<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler*>) detecta las excepciones generadas en los middlewares siguientes.
     * El middleware del protocolo de seguridad de transporte estricta de HTTP (HSTS) (<xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts*>) agrega el encabezado `Strict-Transport-Security`.
1. El middleware de redireccionamiento de HTTPS (<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection*>) redirige las solicitudes HTTP a HTTPS.
1. El middleware de archivos estáticos (<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles*>) devuelve archivos estáticos y genera un cortocircuito más allá del procesamiento de la solicitud.
1. El middleware de directivas de cookies (<xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy*>) permite que la aplicación cumpla con las normas del Reglamento general de protección de datos (RGPD) de la Unión Europea.
1. Middleware de enrutamiento (`UseRouting`) para enrutar las solicitudes.
1. El middleware de autenticación (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*>) intenta autenticar al usuarios antes de que se le permita acceder a los recursos protegidos.
1. El middleware de autorización (`UseAuthorization`) autoriza a los usuarios a acceder a los recursos seguros.
1. El middleware de sesiones (<xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession*>) establece y mantiene el estado de sesión. Si la aplicación usa el estado de sesión, llame al middleware de sesiones después del middleware de directivas de cookies y antes del middleware de MVC.
1. Middleware de enrutamiento de punto de conexión (`UseEndpoints` con `MapRazorPages`) para agregar puntos de conexión de Razor Pages a la canalización de solicitudes.

<!--

FUTURE UPDATE

On the next topic overhaul/release update, add API crosslink to "Database Error Page Middleware" in Item 1 of the list ...

Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage*

... when available via the API docs.

-->

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseRouting();
    app.UseAuthentication();
    app.UseAuthorization();
    app.UseSession();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

En el código de ejemplo anterior, cada método de extensión de software intermedio se expone en <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> a través del espacio de nombres de <xref:Microsoft.AspNetCore.Builder?displayProperty=fullName>.

<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler*> es el primer componente de software intermedio que se agrega a la canalización. Por lo tanto, el software intermedio del controlador de excepciones detectará las excepciones que se produzcan en las llamadas posteriores.

El software intermedio de archivos estáticos se llama al principio de la canalización para que pueda controlar solicitudes y realizar cortocircuitos sin pasar por los componentes restantes. Este middleware **no** proporciona comprobaciones de autorización. Los archivos que proporciona el middleware de archivos estáticos, incluidos los de *wwwroot*, están disponibles de forma pública. Para obtener más información sobre cómo proteger este tipo de archivos, vea <xref:fundamentals/static-files>.

Si el software intermedio de archivos estáticos no controla la solicitud, se pasará al software intermedio de autenticación (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*>), que realizará la autenticación. Este software intermedio no cortocircuita las solicitudes sin autenticación. Aunque autentique solicitudes, la autorización (y también el rechazo) se producirán después de que MVC seleccione una página de Razor o un control y una acción de MVC concretos.

En el ejemplo siguiente se muestra un orden de software intermedio en el que el software intermedio de archivos estáticos controla las solicitudes de archivos estáticos antes del software intermedio de compresión de respuestas. Los archivos estáticos no se comprimen en este orden de software intermedio. Las respuestas de Razor Pages se pueden comprimir.

```csharp
public void Configure(IApplicationBuilder app)
{
    // Static files aren't compressed by Static File Middleware.
    app.UseStaticFiles();

    app.UseResponseCompression();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

1. Control de errores y excepciones
   * Cuando la aplicación se ejecuta en el entorno de desarrollo:
     * El middleware de la página de excepciones para el desarrollador (<xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage*>) informa los errores en tiempo de ejecución de la aplicación.
     * El middleware de la página de errores de la base de datos (`Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage`) informa los errores en tiempo de ejecución de la base de datos.
   * Cuando la aplicación se ejecuta en el entorno de producción:
     * El middleware del controlador de excepciones (<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler*>) detecta las excepciones generadas en los middlewares siguientes.
     * El middleware del protocolo de seguridad de transporte estricta de HTTP (HSTS) (<xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts*>) agrega el encabezado `Strict-Transport-Security`.
1. El middleware de redireccionamiento de HTTPS (<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection*>) redirige las solicitudes HTTP a HTTPS.
1. El middleware de archivos estáticos (<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles*>) devuelve archivos estáticos y genera un cortocircuito más allá del procesamiento de la solicitud.
1. El middleware de directivas de cookies (<xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy*>) permite que la aplicación cumpla con las normas del Reglamento general de protección de datos (RGPD) de la Unión Europea.
1. El middleware de autenticación (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*>) intenta autenticar al usuarios antes de que se le permita acceder a los recursos protegidos.
1. El middleware de sesiones (<xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession*>) establece y mantiene el estado de sesión. Si la aplicación usa el estado de sesión, llame al middleware de sesiones después del middleware de directivas de cookies y antes del middleware de MVC.
1. MVC (<xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc*>) para agregar MVC a la canalización de la solicitud.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseAuthentication();
    app.UseSession();
    app.UseMvc();
}
```

En el código de ejemplo anterior, cada método de extensión de software intermedio se expone en <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> a través del espacio de nombres de <xref:Microsoft.AspNetCore.Builder?displayProperty=fullName>.

<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler*> es el primer componente de software intermedio que se agrega a la canalización. Por lo tanto, el software intermedio del controlador de excepciones detectará las excepciones que se produzcan en las llamadas posteriores.

El software intermedio de archivos estáticos se llama al principio de la canalización para que pueda controlar solicitudes y realizar cortocircuitos sin pasar por los componentes restantes. Este middleware **no** proporciona comprobaciones de autorización. Los archivos que proporciona el middleware de archivos estáticos, incluidos los de *wwwroot*, están disponibles de forma pública. Para obtener más información sobre cómo proteger este tipo de archivos, vea <xref:fundamentals/static-files>.

Si el software intermedio de archivos estáticos no controla la solicitud, se pasará al software intermedio de autenticación (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*>), que realizará la autenticación. Este software intermedio no cortocircuita las solicitudes sin autenticación. Aunque autentique solicitudes, la autorización (y también el rechazo) se producirán después de que MVC seleccione una página de Razor o un control y una acción de MVC concretos.

En el ejemplo siguiente se muestra un orden de software intermedio en el que el software intermedio de archivos estáticos controla las solicitudes de archivos estáticos antes del software intermedio de compresión de respuestas. Los archivos estáticos no se comprimen en este orden de software intermedio. Las respuestas de MVC de <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute*> se pueden comprimir.

```csharp
public void Configure(IApplicationBuilder app)
{
    // Static files aren't compressed by Static File Middleware.
    app.UseStaticFiles();

    app.UseResponseCompression();

    app.UseMvcWithDefaultRoute();
}
```

::: moniker-end

## <a name="use-run-and-map"></a>Use, Run y Map

Puede configurar la canalización HTTP con <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use*>, <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run*> y <xref:Microsoft.AspNetCore.Builder.MapExtensions.Map*>. El método `Use` puede cortocircuitar la canalización (solo si no llama a un delegado de solicitudes `next`). `Run` es una convención y es posible que algunos componentes de middleware expongan métodos `Run[Middleware]` que se ejecutan al final de la canalización.

Las extensiones <xref:Microsoft.AspNetCore.Builder.MapExtensions.Map*> se usan como convenciones para la creación de ramas en la canalización. `Map` crea una rama de la canalización de solicitudes según las coincidencias de la ruta de solicitud proporcionada. Si la ruta de solicitud comienza con la ruta proporcionada, se ejecuta la creación de la rama.

[!code-csharp[](index/snapshot/Chain/StartupMap.cs)]

En la siguiente tabla se muestran las solicitudes y las respuestas de `http://localhost:1234` con el código anterior.

| Request             | Respuesta                     |
| ------------------- | ---------------------------- |
| localhost:1234      | Saludos del delegado sin Map. |
| localhost:1234/map1 | Prueba 1 de Map                   |
| localhost:1234/map2 | Prueba 2 de Map                   |
| localhost:1234/map3 | Saludos del delegado sin Map. |

Cuando se usa `Map`, los segmentos de ruta que coincidan se eliminan de `HttpRequest.Path` y se anexan a `HttpRequest.PathBase` para cada solicitud.

<xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen*> crea una rama de la canalización de solicitudes según el resultado del predicado proporcionado. Se puede usar cualquier predicado de tipo `Func<HttpContext, bool>` para asignar solicitudes a nuevas ramas de la canalización. En el ejemplo siguiente se usa un predicado para detectar la presencia de una `branch` variable de cadena de consulta:

[!code-csharp[](index/snapshot/Chain/StartupMapWhen.cs)]

En la siguiente tabla se muestran las solicitudes y las respuestas de `http://localhost:1234` con el código anterior.

| Request                       | Respuesta                     |
| ----------------------------- | ---------------------------- |
| localhost:1234                | Saludos del delegado sin Map. |
| localhost:1234/?branch=master | Rama usada = master         |

`Map` admite la anidación, por ejemplo:

```csharp
app.Map("/level1", level1App => {
    level1App.Map("/level2a", level2AApp => {
        // "/level1/level2a" processing
    });
    level1App.Map("/level2b", level2BApp => {
        // "/level1/level2b" processing
    });
});
```

`Map` puede hacer coincidir varios segmentos a la vez:

[!code-csharp[](index/snapshot/Chain/StartupMultiSeg.cs?highlight=13)]

## <a name="built-in-middleware"></a>Middleware integrado

ASP.NET Core incluye los componentes de software intermedio siguientes. En la columna *Orden* se proporcionan notas sobre la ubicación del middleware en la canalización de procesamiento de solicitudes, así como las condiciones con las que podría finalizar el procesamiento de solicitudes. Cuando un middleware cortocircuita la canalización de procesamiento de solicitudes e impide el procesamiento de una solicitud por parte de middleware descendente adicional, se llama *middleware de terminal*. Para más información sobre cómo cortocircuitar, consulte la sección [Creación de una canalización de middleware con IApplicationBuilder](#create-a-middleware-pipeline-with-iapplicationbuilder).

| Software intermedio | DESCRIPCIÓN | Ordenar |
| ---------- | ----------- | ----- |
| [Autenticación](xref:security/authentication/identity) | Proporciona compatibilidad con autenticación. | Antes de que se necesite `HttpContext.User`. Terminal para devoluciones de llamadas OAuth. |
| [Directiva de cookies](xref:security/gdpr) | Realiza un seguimiento del consentimiento de los usuarios para almacenar información personal y aplica los estándares mínimos para los campos de las cookies, como `secure` y `SameSite`. | Antes del middleware que emite las cookies. Ejemplos: autenticación, sesión y MVC (TempData). |
| [CORS](xref:security/cors) | Configura el uso compartido de recursos entre orígenes. | Antes de los componentes que usan CORS. |
| [Diagnóstico](xref:fundamentals/error-handling) | Varios middleware independientes que proporcionan una página de excepciones para el desarrollador, control de excepciones, páginas de códigos de estado y la página web predeterminada para las nuevas aplicaciones. | Antes de los componentes que generan errores. Terminal para excepciones o con el fin de proporcionar la página web predeterminada para las nuevas aplicaciones. |
| [Encabezados reenviados](xref:host-and-deploy/proxy-load-balancer) | Reenvía encabezados con proxy a la solicitud actual. | Antes de los componentes que consumen los campos actualizados. Ejemplos: esquema, host, IP de cliente y método. |
| [Comprobación de estado](xref:host-and-deploy/health-checks) | Comprueba el estado de una aplicación ASP.NET Core y sus dependencias, como la comprobación de disponibilidad de base de datos. | Terminal si una solicitud coincide con un punto de conexión de comprobación de estado. |
| [Invalidación del método HTTP](xref:Microsoft.AspNetCore.Builder.HttpMethodOverrideExtensions) | Permite que una solicitud POST entrante invalide el método. | Antes de los componentes que consumen el método actualizado. |
| [Redireccionamiento de HTTPS](xref:security/enforcing-ssl#require-https) | Redirija todas las solicitudes HTTP a HTTPS. | Antes de los componentes que consumen la dirección URL. |
| [Seguridad de transporte estricta de HTTP (HSTS)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts) | Middleware de mejora de seguridad que agrega un encabezado de respuesta especial. | Antes de que se envíen las respuestas y después de los componentes que modifican las solicitudes. Ejemplos: encabezados reenviados y reescritura de URL. |
| [MVC](xref:mvc/overview) | Procesa las solicitudes con MVC/Razor Pages. | Si hay una solicitud que coincida con una ruta, será final. |
| [OWIN](xref:fundamentals/owin) | Puede interoperar con aplicaciones, servidores y software intermedio basados en OWIN. | Si el software intermedio de OWIN procesa completamente la solicitud, será final. |
| [Almacenamiento en caché de respuestas](xref:performance/caching/middleware) | Proporciona compatibilidad con la captura de respuestas. | Antes de los componentes que requieren el almacenamiento en caché. |
| [Compresión de respuesta](xref:performance/response-compression) | Proporciona compatibilidad con la compresión de respuestas. | Antes de los componentes que requieren compresión. |
| [Localización de solicitudes](xref:fundamentals/localization) | Proporciona compatibilidad con ubicación. | Antes de los componentes que dependen de la ubicación. |
| [Enrutamiento de punto de conexión](xref:fundamentals/routing) | Define y restringe las rutas de la solicitud. | Terminal para rutas que coincidan. |
| [Sesión](xref:fundamentals/app-state) | Proporciona compatibilidad con la administración de sesiones de usuario. | Antes de los componentes que requieren Session. |
| [Archivos estáticos](xref:fundamentals/static-files) | Proporciona compatibilidad con la proporción de archivos estáticos y la exploración de directorios. | Si hay una solicitud que coincida con un archivo, será final. |
| [Reescritura de direcciones URL](xref:fundamentals/url-rewriting) | Proporciona compatibilidad con la reescritura de direcciones URL y la redirección de solicitudes. | Antes de los componentes que consumen la dirección URL. |
| [WebSockets](xref:fundamentals/websockets) | Habilita el protocolo WebSockets. | Antes de los componentes necesarios para aceptar solicitudes de WebSocket. |

## <a name="additional-resources"></a>Recursos adicionales

* <xref:fundamentals/middleware/write>
* <xref:migration/http-modules>
* <xref:fundamentals/startup>
* <xref:fundamentals/request-features>
* <xref:fundamentals/middleware/extensibility>
* <xref:fundamentals/middleware/extensibility-third-party-container>
