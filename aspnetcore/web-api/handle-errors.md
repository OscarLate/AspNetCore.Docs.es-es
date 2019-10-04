---
title: Control de errores en API web de ASP.NET Core
author: pranavkm
description: Obtenga información sobre el control de errores con API web de ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: prkrishn
ms.custom: mvc
ms.date: 09/27/2019
uid: web-api/handle-errors
ms.openlocfilehash: dc21d4b2cf096b8d38b0a24d739e6874186004e7
ms.sourcegitcommit: 5d25a7f22c50ca6fdd0f8ecd8e525822e1b35b7a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2019
ms.locfileid: "71551743"
---
# <a name="handle-errors-in-aspnet-core-web-apis"></a><span data-ttu-id="41a60-103">Control de errores en API web de ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="41a60-103">Handle errors in ASP.NET Core web APIs</span></span>

<span data-ttu-id="41a60-104">En este artículo se describe cómo administrar y personalizar el control de errores con API web de ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="41a60-104">This article describes how to handle and customize error handling with ASP.NET Core web APIs.</span></span>

<span data-ttu-id="41a60-105">[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/handle-errors/samples) ([cómo descargarlo](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="41a60-105">[View or download sample code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/handle-errors/samples) ([How to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="developer-exception-page"></a><span data-ttu-id="41a60-106">Página de excepciones para el desarrollador</span><span class="sxs-lookup"><span data-stu-id="41a60-106">Developer Exception Page</span></span>

<span data-ttu-id="41a60-107">La [Página de excepciones para el desarrollador](xref:fundamentals/error-handling) es una herramienta útil para obtener seguimientos de pila detallados de los errores del servidor.</span><span class="sxs-lookup"><span data-stu-id="41a60-107">The [Developer Exception Page](xref:fundamentals/error-handling) is a useful tool to get detailed stack traces for server errors.</span></span> <span data-ttu-id="41a60-108">Usa <xref:Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware> para capturar excepciones sincrónicas y asincrónicas de la canalización HTTP y para generar respuestas de error.</span><span class="sxs-lookup"><span data-stu-id="41a60-108">It uses <xref:Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware> to capture synchronous and asynchronous exceptions from the HTTP pipeline and to generate error responses.</span></span> <span data-ttu-id="41a60-109">A modo de ejemplo, observe la siguiente acción del controlador:</span><span class="sxs-lookup"><span data-stu-id="41a60-109">To illustrate, consider the following controller action:</span></span>

[!code-csharp[](handle-errors/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_GetByCity)]

<span data-ttu-id="41a60-110">Ejecute el siguiente comando `curl` para probar la acción anterior:</span><span class="sxs-lookup"><span data-stu-id="41a60-110">Run the following `curl` command to test the preceding action:</span></span>

```bash
curl -i https://localhost:5001/weatherforecast/chicago
```

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="41a60-111">En ASP.NET Core 3.0 y versiones posteriores, en la Página de excepciones para el desarrollador se muestra una respuesta de texto sin formato si el cliente no solicita la salida con formato HTML.</span><span class="sxs-lookup"><span data-stu-id="41a60-111">In ASP.NET Core 3.0 and later, the Developer Exception Page displays a plain-text response if the client doesn't request HTML-formatted output.</span></span> <span data-ttu-id="41a60-112">Aparece el siguiente resultado:</span><span class="sxs-lookup"><span data-stu-id="41a60-112">The following output appears:</span></span>

```console
HTTP/1.1 500 Internal Server Error
Transfer-Encoding: chunked
Content-Type: text/plain
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 27 Sep 2019 16:13:16 GMT

System.ArgumentException: We don't offer a weather forecast for chicago. (Parameter 'city')
   at WebApiSample.Controllers.WeatherForecastController.Get(String city) in C:\working_folder\aspnet\AspNetCore.Docs\aspnetcore\web-api\handle-errors\samples\3.x\Controllers\WeatherForecastController.cs:line 34
   at lambda_method(Closure , Object , Object[] )
   at Microsoft.Extensions.Internal.ObjectMethodExecutor.Execute(Object target, Object[] parameters)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ActionMethodExecutor.SyncObjectResultExecutor.Execute(IActionResultTypeMapper mapper, ObjectMethodExecutor executor, Object controller, Object[] arguments)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeActionMethodAsync>g__Logged|12_1(ControllerActionInvoker invoker)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeNextActionFilterAsync>g__Awaited|10_0(ControllerActionInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Rethrow(ActionExecutedContextSealed context)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Next(State& next, Scope& scope, Object& state, Boolean& isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.InvokeInnerFilterAsync()
--- End of stack trace from previous location where exception was thrown ---
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeFilterPipelineAsync>g__Awaited|19_0(ResourceInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Logged|17_1(ResourceInvoker invoker)
   at Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|6_0(Endpoint endpoint, Task requestTask, ILogger logger)
   at Microsoft.AspNetCore.Authorization.AuthorizationMiddleware.Invoke(HttpContext context)
   at Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware.Invoke(HttpContext context)

HEADERS
=======
Accept: */*
Host: localhost:44312
User-Agent: curl/7.55.1
```

<span data-ttu-id="41a60-113">Para mostrar una respuesta con formato HTML, establezca el encabezado de solicitud HTTP `Accept` en el tipo de medio `text/html`.</span><span class="sxs-lookup"><span data-stu-id="41a60-113">To display an HTML-formatted response instead, set the `Accept` HTTP request header to the `text/html` media type.</span></span> <span data-ttu-id="41a60-114">Por ejemplo:</span><span class="sxs-lookup"><span data-stu-id="41a60-114">For example:</span></span>

```bash
curl -i -H "Accept: text/html" https://localhost:5001/weatherforecast/chicago
```

<span data-ttu-id="41a60-115">El siguiente fragmento de la respuesta HTTP es importante:</span><span class="sxs-lookup"><span data-stu-id="41a60-115">Consider the following excerpt from the HTTP response:</span></span>

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

<span data-ttu-id="41a60-116">En ASP.NET Core 2.2 y versiones anteriores, en la Página de excepciones para el desarrollador se muestra una respuesta con formato HTML.</span><span class="sxs-lookup"><span data-stu-id="41a60-116">In ASP.NET Core 2.2 and earlier, the Developer Exception Page displays an HTML-formatted response.</span></span> <span data-ttu-id="41a60-117">Por ejemplo, el siguiente fragmento de la respuesta HTTP es importante:</span><span class="sxs-lookup"><span data-stu-id="41a60-117">For example, consider the following excerpt from the HTTP response:</span></span>

::: moniker-end

```console
HTTP/1.1 500 Internal Server Error
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 27 Sep 2019 16:55:37 GMT

<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta charset="utf-8" />
        <title>Internal Server Error</title>
        <style>
            body {
    font-family: 'Segoe UI', Tahoma, Arial, Helvetica, sans-serif;
    font-size: .813em;
    color: #222;
    background-color: #fff;
}
```

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="41a60-118">La respuesta con formato HTML resulta útil al realizar pruebas mediante herramientas como Postman.</span><span class="sxs-lookup"><span data-stu-id="41a60-118">The HTML-formatted response becomes useful when testing via tools like Postman.</span></span> <span data-ttu-id="41a60-119">En la siguiente captura de pantalla se muestra tanto el texto sin formato como las respuestas con formato HTML en Postman:</span><span class="sxs-lookup"><span data-stu-id="41a60-119">The following screen capture shows both the plain-text and the HTML-formatted responses in Postman:</span></span>

![Prueba de la Página de excepciones para el desarrollador en Postman](handle-errors/_static/developer-exception-page-postman.gif)

::: moniker-end

> [!WARNING]
> <span data-ttu-id="41a60-121">Habilite la página de excepciones para el desarrollador **solo cuando la aplicación se ejecute en el entorno de desarrollo**.</span><span class="sxs-lookup"><span data-stu-id="41a60-121">Enable the Developer Exception Page **only when the app is running in the Development environment**.</span></span> <span data-ttu-id="41a60-122">No le interesa compartir públicamente información detallada sobre las excepciones cuando la aplicación se ejecute en producción.</span><span class="sxs-lookup"><span data-stu-id="41a60-122">You don't want to share detailed exception information publicly when the app runs in production.</span></span> <span data-ttu-id="41a60-123">Para más información sobre la configuración de entornos, consulte <xref:fundamentals/environments>.</span><span class="sxs-lookup"><span data-stu-id="41a60-123">For more information on configuring environments, see <xref:fundamentals/environments>.</span></span>

## <a name="exception-handler"></a><span data-ttu-id="41a60-124">Controlador de excepciones</span><span class="sxs-lookup"><span data-stu-id="41a60-124">Exception handler</span></span>

<span data-ttu-id="41a60-125">En entornos que no son de desarrollo, se puede usar [middleware de control de excepciones](xref:fundamentals/error-handling) para producir una carga de error:</span><span class="sxs-lookup"><span data-stu-id="41a60-125">In non-development environments, [Exception Handling Middleware](xref:fundamentals/error-handling) can be used to produce an error payload:</span></span>

1. <span data-ttu-id="41a60-126">En `Startup.Configure`, invoque <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler*> para usar el middleware:</span><span class="sxs-lookup"><span data-stu-id="41a60-126">In `Startup.Configure`, invoke <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler*> to use the middleware:</span></span>

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

1. <span data-ttu-id="41a60-127">Configure una acción de controlador para responder a la ruta `/error`:</span><span class="sxs-lookup"><span data-stu-id="41a60-127">Configure a controller action to respond to the `/error` route:</span></span>

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

<span data-ttu-id="41a60-128">La acción `Error` anterior envía una carga compatible con [RFC7807](https://tools.ietf.org/html/rfc7807) al cliente.</span><span class="sxs-lookup"><span data-stu-id="41a60-128">The preceding `Error` action sends an [RFC7807](https://tools.ietf.org/html/rfc7807)-compliant payload to the client.</span></span>

<span data-ttu-id="41a60-129">El middleware de control de excepciones también puede proporcionar una salida negociada de contenido más detallada en el entorno de desarrollo local.</span><span class="sxs-lookup"><span data-stu-id="41a60-129">Exception Handling Middleware can also provide more detailed content-negotiated output in the local development environment.</span></span> <span data-ttu-id="41a60-130">Use los pasos siguientes para generar un formato de carga coherente en los entornos de desarrollo y producción:</span><span class="sxs-lookup"><span data-stu-id="41a60-130">Use the following steps to produce a consistent payload format across development and production environments:</span></span>

1. <span data-ttu-id="41a60-131">En `Startup.Configure`, registre las instancias de middleware de control de excepciones específicas del entorno:</span><span class="sxs-lookup"><span data-stu-id="41a60-131">In `Startup.Configure`, register environment-specific Exception Handling Middleware instances:</span></span>

    ::: moniker range=">= aspnetcore-3.0"

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseExceptionHandler("/error-local-development");
        }
        else
        {
            app.UseExceptionHandler("/error");
        }
    }
    ```

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseExceptionHandler("/error-local-development");
        }
        else
        {
            app.UseExceptionHandler("/error");
        }
    }
    ```

    ::: moniker-end

    <span data-ttu-id="41a60-132">En el código anterior, el middleware se ha registrado con:</span><span class="sxs-lookup"><span data-stu-id="41a60-132">In the preceding code, the middleware is registered with:</span></span>

    * <span data-ttu-id="41a60-133">Una ruta de `/error-local-development` en el entorno de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="41a60-133">A route of `/error-local-development` in the Development environment.</span></span>
    * <span data-ttu-id="41a60-134">Una ruta de `/error` en entornos que no son de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="41a60-134">A route of `/error` in environments that aren't Development.</span></span>
    
1. <span data-ttu-id="41a60-135">Aplique el enrutamiento de atributos a acciones del controlador:</span><span class="sxs-lookup"><span data-stu-id="41a60-135">Apply attribute routing to controller actions:</span></span>

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

## <a name="use-exceptions-to-modify-the-response"></a><span data-ttu-id="41a60-136">Uso de excepciones para modificar la respuesta</span><span class="sxs-lookup"><span data-stu-id="41a60-136">Use exceptions to modify the response</span></span>

<span data-ttu-id="41a60-137">El contenido de la respuesta se puede modificar desde fuera del controlador.</span><span class="sxs-lookup"><span data-stu-id="41a60-137">The contents of the response can be modified from outside of the controller.</span></span> <span data-ttu-id="41a60-138">En la API web de ASP.NET 4.x, una manera de hacerlo era usar el tipo <xref:System.Web.Http.HttpResponseException>.</span><span class="sxs-lookup"><span data-stu-id="41a60-138">In ASP.NET 4.x Web API, one way to do this was using the <xref:System.Web.Http.HttpResponseException> type.</span></span> <span data-ttu-id="41a60-139">ASP.NET Core no incluye un tipo equivalente.</span><span class="sxs-lookup"><span data-stu-id="41a60-139">ASP.NET Core doesn't include an equivalent type.</span></span> <span data-ttu-id="41a60-140">Se pueden agregar compatibilidad para `HttpResponseException` con los pasos siguientes:</span><span class="sxs-lookup"><span data-stu-id="41a60-140">Support for `HttpResponseException` can be added with the following steps:</span></span>

1. <span data-ttu-id="41a60-141">Cree un tipo de excepción conocido denominado `HttpResponseException`:</span><span class="sxs-lookup"><span data-stu-id="41a60-141">Create a well-known exception type named `HttpResponseException`:</span></span>

    [!code-csharp[](handle-errors/samples/3.x/Exceptions/HttpResponseException.cs?name=snippet_HttpResponseException)]

1. <span data-ttu-id="41a60-142">Cree un filtro de acción denominado `HttpResponseExceptionFilter`:</span><span class="sxs-lookup"><span data-stu-id="41a60-142">Create an action filter named `HttpResponseExceptionFilter`:</span></span>

    [!code-csharp[](handle-errors/samples/3.x/Filters/HttpResponseExceptionFilter.cs?name=snippet_HttpResponseExceptionFilter)]

1. <span data-ttu-id="41a60-143">En `Startup.ConfigureServices`, agregue el filtro de acción a la colección de filtros:</span><span class="sxs-lookup"><span data-stu-id="41a60-143">In `Startup.ConfigureServices`, add the action filter to the filters collection:</span></span>

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.2"
    
    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.1"

    [!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

## <a name="validation-failure-error-response"></a><span data-ttu-id="41a60-144">Respuesta de error ante errores de validación</span><span class="sxs-lookup"><span data-stu-id="41a60-144">Validation failure error response</span></span>

<span data-ttu-id="41a60-145">En el caso de los controladores de API web, MVC responde con un tipo de respuesta <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> cuando se produce un error en la validación del modelo.</span><span class="sxs-lookup"><span data-stu-id="41a60-145">For web API controllers, MVC responds with a <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> response type when model validation fails.</span></span> <span data-ttu-id="41a60-146">MVC usa los resultados de <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.InvalidModelStateResponseFactory> para construir la respuesta de error para un error de validación.</span><span class="sxs-lookup"><span data-stu-id="41a60-146">MVC uses the results of <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.InvalidModelStateResponseFactory> to construct the error response for a validation failure.</span></span> <span data-ttu-id="41a60-147">En el ejemplo siguiente se usa la fábrica para cambiar el tipo de respuesta predeterminado a <xref:Microsoft.AspNetCore.Mvc.SerializableError> en `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="41a60-147">The following example uses the factory to change the default response type to <xref:Microsoft.AspNetCore.Mvc.SerializableError> in `Startup.ConfigureServices`:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=4-13)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=5-14)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=6-15)]

::: moniker-end

## <a name="client-error-response"></a><span data-ttu-id="41a60-148">Respuesta de error del cliente</span><span class="sxs-lookup"><span data-stu-id="41a60-148">Client error response</span></span>

<span data-ttu-id="41a60-149">Se define un *resultado error* como un resultado con un código de estado HTTP de 400 o superior.</span><span class="sxs-lookup"><span data-stu-id="41a60-149">An *error result* is defined as a result with an HTTP status code of 400 or higher.</span></span> <span data-ttu-id="41a60-150">En el caso de los controladores de API web, MVC transforma un resultado de un error en un resultado con <xref:Microsoft.AspNetCore.Mvc.ProblemDetails>.</span><span class="sxs-lookup"><span data-stu-id="41a60-150">For web API controllers, MVC transforms an error result to a result with <xref:Microsoft.AspNetCore.Mvc.ProblemDetails>.</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="41a60-151">La respuesta de error se puede configurar de una de las siguientes maneras:</span><span class="sxs-lookup"><span data-stu-id="41a60-151">The error response can be configured in one of the following ways:</span></span>

1. [<span data-ttu-id="41a60-152">Implementar ProblemDetailsFactory</span><span class="sxs-lookup"><span data-stu-id="41a60-152">Implement ProblemDetailsFactory</span></span>](#implement-problemdetailsfactory)
1. [<span data-ttu-id="41a60-153">Usar ApiBehaviorOptions.ClientErrorMapping</span><span class="sxs-lookup"><span data-stu-id="41a60-153">Use ApiBehaviorOptions.ClientErrorMapping</span></span>](#use-apibehavioroptionsclienterrormapping)

### <a name="implement-problemdetailsfactory"></a><span data-ttu-id="41a60-154">Implementación de ProblemDetailsFactory</span><span class="sxs-lookup"><span data-stu-id="41a60-154">Implement ProblemDetailsFactory</span></span>

<span data-ttu-id="41a60-155">MVC usa `Microsoft.AspNetCore.Mvc.ProblemDetailsFactory` para generar todas las instancias de <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> y <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails>.</span><span class="sxs-lookup"><span data-stu-id="41a60-155">MVC uses `Microsoft.AspNetCore.Mvc.ProblemDetailsFactory` to produce all instances of <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> and <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails>.</span></span> <span data-ttu-id="41a60-156">Esto incluye las respuestas de error del cliente, las respuestas de error ante errores de validación y los métodos auxiliares `Microsoft.AspNetCore.Mvc.ControllerBase.Problem` y <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem>.</span><span class="sxs-lookup"><span data-stu-id="41a60-156">This includes client error responses, validation failure error responses, and the `Microsoft.AspNetCore.Mvc.ControllerBase.Problem` and <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem> helper methods.</span></span>

<span data-ttu-id="41a60-157">Para personalizar la respuesta de detalles del problema, registre una implementación personalizada de `ProblemDetailsFactory` en `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="41a60-157">To customize the problem details response, register a custom implementation of `ProblemDetailsFactory` in `Startup.ConfigureServices`:</span></span>

```csharp
public void ConfigureServices(IServiceCollection serviceCollection)
{
    services.AddControllers();
    services.AddTransient<ProblemDetailsFactory, CustomProblemDetailsFactory>();
}
```

::: moniker-end

::: moniker range="= aspnetcore-2.2"

<span data-ttu-id="41a60-158">La respuesta de error se puede configurar como se describe en la sección [Uso de ApiBehaviorOptions.ClientErrorMapping](#use-apibehavioroptionsclienterrormapping).</span><span class="sxs-lookup"><span data-stu-id="41a60-158">The error response can be configured as outlined in the [Use ApiBehaviorOptions.ClientErrorMapping](#use-apibehavioroptionsclienterrormapping) section.</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="use-apibehavioroptionsclienterrormapping"></a><span data-ttu-id="41a60-159">Uso de ApiBehaviorOptions.ClientErrorMapping</span><span class="sxs-lookup"><span data-stu-id="41a60-159">Use ApiBehaviorOptions.ClientErrorMapping</span></span>

<span data-ttu-id="41a60-160">Use la propiedad <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.ClientErrorMapping*> para configurar el contenido de la respuesta `ProblemDetails`.</span><span class="sxs-lookup"><span data-stu-id="41a60-160">Use the <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.ClientErrorMapping*> property to configure the contents of the `ProblemDetails` response.</span></span> <span data-ttu-id="41a60-161">Por ejemplo, el código siguiente de `Startup.ConfigureServices` permite actualizar la propiedad `type` para respuestas 404:</span><span class="sxs-lookup"><span data-stu-id="41a60-161">For example, the following code in `Startup.ConfigureServices` updates the `type` property for 404 responses:</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=8-9)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=9-10)]

::: moniker-end