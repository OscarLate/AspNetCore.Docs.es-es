---
title: Autorización basada en directivas en ASP.NET Core
author: rick-anderson
description: Aprenda a crear y usar controladores de directivas de autorización para aplicar los requisitos de autorización en una aplicación ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 04/05/2019
uid: security/authorization/policies
ms.openlocfilehash: 60625944d4ba31da6b98bdf947991088dc75ed87
ms.sourcegitcommit: 7001657c00358b082734ba4273693b9b3ed35d2a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2019
ms.locfileid: "68669969"
---
# <a name="policy-based-authorization-in-aspnet-core"></a>Autorización basada en directivas en ASP.NET Core

En segundo plano, la autorización basada en [roles](xref:security/authorization/roles) y la [autorización basada](xref:security/authorization/claims) en notificaciones usan un requisito, un controlador de requisitos y una directiva configurada previamente. Estos bloques de creación admiten la expresión de evaluaciones de autorización en el código. El resultado es una estructura de autorización más enriquecida, reutilizable y comprobable.

Una directiva de autorización se compone de uno o varios requisitos. Se registra como parte de la configuración del servicio de autorización, en `Startup.ConfigureServices` el método:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=32-33,48-53,61,66)]

En el ejemplo anterior, se crea una directiva "AtLeast21". Tiene un único requisito&mdash;de una edad mínima, que se proporciona como un parámetro al requisito.

## <a name="iauthorizationservice"></a>IAuthorizationService 

El servicio principal que determina si la autorización se realiza <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService>correctamente es:

[!code-csharp[](policies/samples/stubs/copy_of_IAuthorizationService.cs?highlight=24-25,48-49&name=snippet)]

En el código anterior se resaltan los dos métodos de [IAuthorizationService](https://github.com/aspnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationService.cs).

<xref:Microsoft.AspNetCore.Authorization.IAuthorizationRequirement>es un servicio de marcador sin métodos y el mecanismo para hacer un seguimiento de si la autorización se realiza correctamente.

Cada <xref:Microsoft.AspNetCore.Authorization.IAuthorizationHandler> uno de ellos es responsable de comprobar si se cumplen los requisitos:
<!--The following code is a copy/paste from 
https://github.com/aspnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationHandler.cs -->

```csharp
/// <summary>
/// Classes implementing this interface are able to make a decision if authorization
/// is allowed.
/// </summary>
public interface IAuthorizationHandler
{
    /// <summary>
    /// Makes a decision if authorization is allowed.
    /// </summary>
    /// <param name="context">The authorization information.</param>
    Task HandleAsync(AuthorizationHandlerContext context);
}
```

La <xref:Microsoft.AspNetCore.Authorization.AuthorizationHandlerContext> clase es lo que el controlador usa para marcar si se cumplen los requisitos:

```csharp
 context.Succeed(requirement)
```

En el código siguiente se muestra la implementación predeterminada simplificada (y anotada con comentarios) del servicio de autorización:

```csharp
public async Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user, 
             object resource, IEnumerable<IAuthorizationRequirement> requirements)
{
    // Create a tracking context from the authorization inputs.
    var authContext = _contextFactory.CreateContext(requirements, user, resource);

    // By default this returns an IEnumerable<IAuthorizationHandlers> from DI.
    var handlers = await _handlers.GetHandlersAsync(authContext);

    // Invoke all handlers.
    foreach (var handler in handlers)
    {
        await handler.HandleAsync(authContext);
    }

    // Check the context, by default success is when all requirements have been met.
    return _evaluator.Evaluate(authContext);
}
```

En el código siguiente se muestra `ConfigureServices`un típico:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add all of your handlers to DI.
    services.AddSingleton<IAuthorizationHandler, MyHandler1>();
    // MyHandler2, ...

    services.AddSingleton<IAuthorizationHandler, MyHandlerN>();

    // Configure your policies
    services.AddAuthorization(options =>
          options.AddPolicy("Something",
          policy => policy.RequireClaim("Permission", "CanViewPage", "CanViewAnything")));


    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
}
```

Use <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService> o`[Authorize(Policy = "Something")]` para la autorización.

## <a name="applying-policies-to-mvc-controllers"></a>Aplicar directivas a los controladores de MVC

Si utiliza Razor Pages, consulte [aplicar directivas a Razor pages](#applying-policies-to-razor-pages) en este documento.

Las directivas se aplican a los controladores `[Authorize]` mediante el uso del atributo con el nombre de la Directiva. Por ejemplo:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Controllers/AlcoholPurchaseController.cs?name=snippet_AlcoholPurchaseControllerClass&highlight=4)]

## <a name="applying-policies-to-razor-pages"></a>Aplicar directivas a Razor Pages

Las directivas se aplican a Razor pages mediante `[Authorize]` el uso del atributo con el nombre de la Directiva. Por ejemplo:

[!code-csharp[](policies/samples/PoliciesAuthApp2/Pages/AlcoholPurchase.cshtml.cs?name=snippet_AlcoholPurchaseModelClass&highlight=4)]

También se pueden aplicar directivas a Razor Pages mediante una [Convención de autorización](xref:security/authorization/razor-pages-authorization).

## <a name="requirements"></a>Requisitos

Un requisito de autorización es una colección de parámetros de datos que una Directiva puede usar para evaluar la entidad de seguridad de usuario actual. En la Directiva "AtLeast21", el requisito es un parámetro&mdash;único que es la edad mínima. Un requisito implementa [IAuthorizationRequirement](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement), que es una interfaz de marcador vacía. Un requisito de edad mínima parametrizada podría implementarse de la siguiente manera:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/MinimumAgeRequirement.cs?name=snippet_MinimumAgeRequirementClass)]

Si una directiva de autorización contiene varios requisitos de autorización, deben cumplirse todos los requisitos para que la evaluación de la Directiva se realice correctamente. En otras palabras, los distintos requisitos de autorización que se agregan a una sola directiva de autorización se tratan de manera **y** .

> [!NOTE]
> No es necesario que un requisito tenga datos ni propiedades.

<a name="security-authorization-policies-based-authorization-handler"></a>

## <a name="authorization-handlers"></a>Controladores de autorización

Un controlador de autorización es responsable de la evaluación de las propiedades de un requisito. El controlador de autorización evalúa los requisitos con respecto a un [AuthorizationHandlerContext](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext) proporcionado para determinar si se permite el acceso.

Un requisito puede tener [varios controladores](#security-authorization-policies-based-multiple-handlers). Un controlador puede heredar [\<AuthorizationHandler TRequirement >](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1), `TRequirement` donde es el requisito que se debe controlar. Como alternativa, un controlador puede implementar [IAuthorizationHandler](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationhandler) para administrar más de un tipo de requisito.

### <a name="use-a-handler-for-one-requirement"></a>Usar un controlador para un requisito

<a name="security-authorization-handler-example"></a>

El siguiente es un ejemplo de una relación uno a uno en la que un controlador de edad mínima emplea un único requisito:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/MinimumAgeHandler.cs?name=snippet_MinimumAgeHandlerClass)]

El código anterior determina si la entidad de seguridad de usuario actual tiene una declaración de nacimiento de fecha que ha sido emitida por un emisor conocido y de confianza. No se puede realizar la autorización cuando falta la demanda, en cuyo caso se devuelve una tarea completada. Cuando existe una demanda, se calcula la edad del usuario. Si el usuario cumple la edad mínima definida por el requisito, la autorización se considera correcta. Cuando la autorización se realiza `context.Succeed` correctamente, se invoca con el requisito satisfecho como su único parámetro.

### <a name="use-a-handler-for-multiple-requirements"></a>Usar un controlador para varios requisitos

El siguiente es un ejemplo de una relación de uno a varios en la que un controlador de permisos puede controlar tres tipos diferentes de requisitos:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/PermissionHandler.cs?name=snippet_PermissionHandlerClass)]

El código anterior atraviesa [PendingRequirements](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.pendingrequirements#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_PendingRequirements)&mdash;una propiedad que contiene requisitos no marcados como correctos. Para un `ReadPermission` requisito, el usuario debe ser un propietario o un patrocinador para tener acceso al recurso solicitado. En el caso de un `EditPermission` requisito `DeletePermission` de o, debe ser un propietario para tener acceso al recurso solicitado.

<a name="security-authorization-policies-based-handler-registration"></a>

### <a name="handler-registration"></a>Registro del controlador

Los controladores se registran en la colección de servicios durante la configuración. Por ejemplo:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=32-33,48-53,61,62-63,66)]

El código anterior se registra `MinimumAgeHandler` como singleton mediante la invocación `services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();`de. Los controladores se pueden registrar con cualquiera de las duraciones de [servicio](xref:fundamentals/dependency-injection#service-lifetimes)integradas.

## <a name="what-should-a-handler-return"></a>¿Qué debe devolver un controlador?

Tenga en cuenta `Handle` que el método del [ejemplo de controlador](#security-authorization-handler-example) no devuelve ningún valor. ¿Cómo se indica el estado correcto o incorrecto?

* Un controlador indica que se ha `context.Succeed(IAuthorizationRequirement requirement)`realizado correctamente llamando a, pasando el requisito que se ha validado correctamente.

* Un controlador no necesita controlar los errores por lo general, ya que otros controladores para el mismo requisito pueden realizarse correctamente.

* Para garantizar un error, incluso si se realizan correctamente otros controladores de `context.Fail`requisitos, llame a.

Si un controlador llama `context.Succeed` a `context.Fail`o, todavía se llama a todos los controladores restantes. Esto permite a los requisitos producir efectos secundarios, como el registro, que tiene lugar incluso si otro controlador ha validado o no un requisito correctamente. Cuando se establece `false`en, la propiedad [InvokeHandlersAfterFailure](/dotnet/api/microsoft.aspnetcore.authorization.authorizationoptions.invokehandlersafterfailure#Microsoft_AspNetCore_Authorization_AuthorizationOptions_InvokeHandlersAfterFailure) (disponible en ASP.net Core 1,1 y versiones posteriores) cortocircuita la ejecución de los controladores `context.Fail` cuando se llama a. `InvokeHandlersAfterFailure`tiene como valor predeterminado ,encuyocasosellamaatodosloscontroladores.`true`

> [!NOTE]
> Se llama a los controladores de autorización incluso si se produce un error de autenticación.

<a name="security-authorization-policies-based-multiple-handlers"></a>

## <a name="why-would-i-want-multiple-handlers-for-a-requirement"></a>¿Por qué sería conveniente tener varios controladores para un requisito?

En los casos en los que desea que la evaluación esté en una base de **o** , implemente varios controladores para un único requisito. Por ejemplo, Microsoft tiene puertas que solo se abren con tarjetas clave. Si deja la tarjeta de la llave en casa, el recepcionista imprime una etiqueta temporal y abre la puerta. En este escenario, tendría un único requisito, *BuildingEntry*, pero varios controladores, cada uno examinando un único requisito.

*BuildingEntryRequirement.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/BuildingEntryRequirement.cs?name=snippet_BuildingEntryRequirementClass)]

*BadgeEntryHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/BadgeEntryHandler.cs?name=snippet_BadgeEntryHandlerClass)]

*TemporaryStickerHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/TemporaryStickerHandler.cs?name=snippet_TemporaryStickerHandlerClass)]

Asegúrese de que ambos controladores están [registrados](xref:security/authorization/policies#security-authorization-policies-based-handler-registration). Si alguno de los controladores se realiza correctamente cuando una directiva `BuildingEntryRequirement`evalúa el, la evaluación de la Directiva se realiza correctamente.

## <a name="using-a-func-to-fulfill-a-policy"></a>Usar un FUNC para cumplir una directiva

Puede haber situaciones en las que el cumplimiento de una directiva sea fácil de expresar en el código. Es posible proporcionar un `Func<AuthorizationHandlerContext, bool>` al configurar la Directiva con el `RequireAssertion` generador de directivas.

Por ejemplo, el anterior `BadgeEntryHandler` podría volver a escribirse de la siguiente manera:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=50-51,55-61)]

## <a name="accessing-mvc-request-context-in-handlers"></a>Acceso al contexto de solicitud MVC en los controladores

El `HandleRequirementAsync` método que se implementa en un controlador de autorización tiene dos parámetros `AuthorizationHandlerContext` : un `TRequirement` y el que se está controlando. Los marcos como MVC o Jabbr pueden agregar cualquier objeto a la `Resource` propiedad `AuthorizationHandlerContext` en para pasar información adicional.

Por ejemplo, MVC pasa una instancia de [AuthorizationFilterContext](/dotnet/api/?term=AuthorizationFilterContext) en la `Resource` propiedad. Esta propiedad proporciona acceso a `HttpContext`, `RouteData`y todo lo demás proporcionado por MVC y Razor pages.

El uso de la `Resource` propiedad es específico de Framework. El uso de la `Resource` información en la propiedad limita las directivas de autorización a marcos concretos. Debe convertir la `Resource` propiedad mediante la `is` palabra clave y, a continuación, confirmar que la conversión se ha realizado correctamente para asegurarse de `InvalidCastException` que el código no se bloquea con cuando se ejecuta en otros marcos:

```csharp
// Requires the following import:
//     using Microsoft.AspNetCore.Mvc.Filters;
if (context.Resource is AuthorizationFilterContext mvcContext)
{
    // Examine MVC-specific things like routing data.
}
```
