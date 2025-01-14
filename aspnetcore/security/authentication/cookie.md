---
title: Usar la autenticación de cookies sin ASP.NET Core identidad
author: rick-anderson
description: Aprenda a usar la autenticación de cookies sin ASP.NET Core identidad.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 08/20/2019
uid: security/authentication/cookie
ms.openlocfilehash: 76c7fc20c8870668ca7c65d975e2ed59f40f7dc8
ms.sourcegitcommit: 116bfaeab72122fa7d586cdb2e5b8f456a2dc92a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70384826"
---
# <a name="use-cookie-authentication-without-aspnet-core-identity"></a>Usar la autenticación de cookies sin ASP.NET Core identidad

Por [Rick Anderson](https://twitter.com/RickAndMSFT) y [Luke Latham](https://github.com/guardrex)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core Identity es un completo proveedor de autenticación completo para crear y mantener inicios de sesión. Sin embargo, se puede usar un proveedor de autenticación de autenticación basada en cookies sin ASP.NET Core identidad. Para obtener más información, consulta <xref:security/authentication/identity>.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/cookie/samples) ([cómo descargarlo](xref:index#how-to-download-a-sample))

Para fines de demostración en la aplicación de ejemplo, la cuenta de usuario para el usuario hipotético, María Rodriguez, está codificada en la aplicación. Use la dirección `maria.rodriguez@contoso.com` de **correo electrónico** y cualquier contraseña para iniciar sesión en el usuario. El usuario se autentica en el `AuthenticateUser` método del archivo *pages/Account/login. cshtml. CS* . En un ejemplo real, el usuario se autenticaría en una base de datos.

## <a name="configuration"></a>Configuración

Si la aplicación no usa el [metapaquete Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app), cree una referencia de paquete en el archivo de proyecto para el paquete [Microsoft. AspNetCore. Authentication. cookies](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Cookies/) .

En el `Startup.ConfigureServices` método, cree los servicios de middleware de autenticación <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> con <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> los métodos y:

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme>se pasa `AddAuthentication` a establece el esquema de autenticación predeterminado para la aplicación. `AuthenticationScheme`resulta útil cuando hay varias instancias de autenticación de cookies y se desea [autorizar con un esquema específico](xref:security/authorization/limitingidentitybyscheme). Si se establece en [CookieAuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) , se proporciona el valor "cookies" para el esquema. `AuthenticationScheme` Puede proporcionar cualquier valor de cadena que distinga el esquema.

El esquema de autenticación de la aplicación es diferente del esquema de autenticación de cookies de la aplicación. Cuando no se proporciona un esquema de autenticación <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*>de cookies `CookieAuthenticationDefaults.AuthenticationScheme` a, USA ("cookies").

La propiedad de la <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> cookie de autenticación está `true` establecida de forma predeterminada en. Las cookies de autenticación se permiten cuando un visitante del sitio no ha dado su consentimiento a la recopilación de datos. Para obtener más información, consulta <xref:security/gdpr#essential-cookies>.

En `Startup.Configure`, llame `UseAuthentication` a `UseAuthorization` y para establecer `HttpContext.User` la propiedad y ejecutar el middleware de autorización para las solicitudes. Llame a `UseAuthentication` los `UseAuthorization` métodos y antes `UseEndpoints`de llamar a:

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet2)]

La <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> clase se utiliza para configurar las opciones del proveedor de autenticación.

Establezca `CookieAuthenticationOptions` en la configuración del servicio para la autenticación `Startup.ConfigureServices` en el método:

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a>Middleware de directiva de cookies

El [middleware de directiva de cookies](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware) habilita las funcionalidades de directiva de cookies. Agregar el middleware a la canalización de procesamiento de la&mdash;aplicación es dependiente del orden. solo afecta a los componentes de nivel inferior registrados en la canalización.

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

Use <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> proporcionado al middleware de la Directiva de cookies para controlar las características globales del procesamiento de cookies y enlazar los controladores de procesamiento de cookies cuando se anexan o eliminan cookies.

El valor <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> predeterminado es `SameSiteMode.Lax` permitir la autenticación de OAuth2. Para aplicar estrictamente una directiva del mismo sitio de `SameSiteMode.Strict`, `MinimumSameSitePolicy`establezca. Aunque esta configuración interrumpe OAuth2 y otros esquemas de autenticación entre orígenes, eleva el nivel de seguridad de las cookies para otros tipos de aplicaciones que no se basan en el procesamiento de solicitudes entre orígenes.

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

La configuración de middleware de la `MinimumSameSitePolicy` Directiva de cookies para `Cookie.SameSite` puede `CookieAuthenticationOptions` afectar a la configuración de en la configuración de acuerdo con la matriz siguiente.

| MinimumSameSitePolicy | Cookie. SameSite | Valor de cookie. SameSite resultante |
| --------------------- | --------------- | --------------------------------- |
| SameSiteMode.None     | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict |
| SameSiteMode.Lax      | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.Lax<br>SameSiteMode.Lax<br>SameSiteMode.Strict |
| SameSiteMode.Strict   | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.Strict<br>SameSiteMode.Strict<br>SameSiteMode.Strict |

## <a name="create-an-authentication-cookie"></a>Crear una cookie de autenticación

Para crear una cookie que contiene información de usuario, <xref:System.Security.Claims.ClaimsPrincipal>construya un. La información del usuario se serializa y se almacena en la cookie. 

Cree un <xref:System.Security.Claims.ClaimsIdentity> con los s <xref:System.Security.Claims.Claim>necesarios y llame <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> al usuario para iniciar sesión:

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

`SignInAsync`crea una cookie cifrada y la agrega a la respuesta actual. Si `AuthenticationScheme` no se especifica, se usa el esquema predeterminado.

El sistema de [protección de datos](xref:security/data-protection/using-data-protection) de ASP.net Core se usa para el cifrado. En el caso de una aplicación hospedada en varios equipos, equilibrio de carga entre aplicaciones o uso de una granja de servidores Web, [Configure la protección de datos](xref:security/data-protection/configuration/overview) para que use el mismo anillo de claves e identificador de la aplicación.

## <a name="sign-out"></a>Cerrar sesión

Para cerrar la sesión del usuario actual y eliminar su cookie, <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>llame a:

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

Si `CookieAuthenticationDefaults.AuthenticationScheme` (o "cookies") no se usa como esquema (por ejemplo, "ContosoCookie"), proporcione el esquema que se usa al configurar el proveedor de autenticación. De lo contrario, se usa el esquema predeterminado.

## <a name="react-to-back-end-changes"></a>Reaccionar a los cambios de back-end

Una vez creada una cookie, ésta es la única fuente de identidad. Si una cuenta de usuario está deshabilitada en los sistemas de servidor:

* El sistema de autenticación de cookies de la aplicación continúa procesando solicitudes basadas en la cookie de autenticación.
* El usuario permanecerá conectado en la aplicación siempre y cuando la cookie de autenticación sea válida.

El <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> evento se puede utilizar para interceptar e invalidar la validación de la identidad de la cookie. La validación de la cookie en cada solicitud mitiga el riesgo de que los usuarios revocados accedan a la aplicación.

Un enfoque para la validación de cookies se basa en realizar un seguimiento de cuándo cambia la base de datos de usuario. Si no se ha cambiado la base de datos desde que se emitió la cookie del usuario, no es necesario volver a autenticar al usuario si la cookie sigue siendo válida. En la aplicación de ejemplo, la base de datos `IUserRepository` se implementa en `LastChanged` y almacena un valor. Cuando se actualiza un usuario en la base de datos `LastChanged` , el valor se establece en la hora actual.

Para invalidar una cookie cuando la base de datos cambie según el `LastChanged` valor, cree la cookie con una `LastChanged` solicitud que contenga el `LastChanged` valor actual de la base de datos:

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Email),
    new Claim("LastChanged", {Database Value})
};

var claimsIdentity = new ClaimsIdentity(
    claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme, 
    new ClaimsPrincipal(claimsIdentity));
```

Para implementar una invalidación para `ValidatePrincipal` el evento, escriba un método con la siguiente firma en una clase que derive de <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents>:

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

El siguiente es un ejemplo de implementación `CookieAuthenticationEvents`de:

```csharp
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;

public class CustomCookieAuthenticationEvents : CookieAuthenticationEvents
{
    private readonly IUserRepository _userRepository;

    public CustomCookieAuthenticationEvents(IUserRepository userRepository)
    {
        // Get the database from registered DI services.
        _userRepository = userRepository;
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        // Look for the LastChanged claim.
        var lastChanged = (from c in userPrincipal.Claims
                           where c.Type == "LastChanged"
                           select c.Value).FirstOrDefault();

        if (string.IsNullOrEmpty(lastChanged) ||
            !_userRepository.ValidateLastChanged(lastChanged))
        {
            context.RejectPrincipal();

            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

Registre la instancia de eventos durante el registro del servicio `Startup.ConfigureServices` de cookies en el método. Proporcione un [registro de servicio de ámbito](xref:fundamentals/dependency-injection#service-lifetimes) para `CustomCookieAuthenticationEvents` la clase:

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

Considere una situación en la que el nombre del usuario se&mdash;actualice una decisión que no afecte a la seguridad de ningún modo. Si desea actualizar de la entidad de seguridad de usuario de una no destructiva `context.ReplacePrincipal` , llame a `context.ShouldRenew` y establezca `true`la propiedad en.

> [!WARNING]
> El enfoque descrito aquí se desencadena en cada solicitud. La validación de las cookies de autenticación para todos los usuarios de cada solicitud puede producir una gran penalización del rendimiento de la aplicación.

## <a name="persistent-cookies"></a>Cookies persistentes

Puede que desee que la cookie se conserve entre sesiones del explorador. Esta persistencia solo se debe habilitar con el consentimiento explícito del usuario con una casilla "Recordarme" al iniciar sesión o un mecanismo similar. 

El siguiente fragmento de código crea una identidad y la cookie correspondiente que sobrevive a través de los cierres del explorador. Se respetan los valores de expiración que se hayan configurado previamente. Si la cookie expira mientras se cierra el explorador, el explorador borra la cookie una vez que se reinicia.

Se <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties>establece enen:`true`

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true
    });
```

## <a name="absolute-cookie-expiration"></a>Expiración absoluta de cookies

Se puede establecer una hora de expiración <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc>absoluta con. Para crear una cookie persistente, `IsPersistent` también se debe establecer. De lo contrario, la cookie se crea con una duración basada en sesión y puede expirar antes o después del vale de autenticación que contiene. Cuando `ExpiresUtc` se establece, reemplaza el valor de la <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.ExpireTimeSpan> opción de <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions>, si se establece.

El siguiente fragmento de código crea una identidad y la cookie correspondiente que dura 20 minutos. Esto omite cualquier configuración de expiración variable previamente configurada.

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
    });
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core Identity es un completo proveedor de autenticación completo para crear y mantener inicios de sesión. Sin embargo, se puede usar un proveedor de autenticación de autenticación basada en cookies sin ASP.NET Core identidad. Para obtener más información, consulta <xref:security/authentication/identity>.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/cookie/samples) ([cómo descargarlo](xref:index#how-to-download-a-sample))

Para fines de demostración en la aplicación de ejemplo, la cuenta de usuario para el usuario hipotético, María Rodriguez, está codificada en la aplicación. Use la dirección `maria.rodriguez@contoso.com` de **correo electrónico** y cualquier contraseña para iniciar sesión en el usuario. El usuario se autentica en el `AuthenticateUser` método del archivo *pages/Account/login. cshtml. CS* . En un ejemplo real, el usuario se autenticaría en una base de datos.

## <a name="configuration"></a>Configuración

Si la aplicación no usa el [metapaquete Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app), cree una referencia de paquete en el archivo de proyecto para el paquete [Microsoft. AspNetCore. Authentication. cookies](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Cookies/) .

En el `Startup.ConfigureServices` método, cree el servicio de middleware de autenticación <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> con <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> los métodos y:

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme>se pasa `AddAuthentication` a establece el esquema de autenticación predeterminado para la aplicación. `AuthenticationScheme`resulta útil cuando hay varias instancias de autenticación de cookies y se desea [autorizar con un esquema específico](xref:security/authorization/limitingidentitybyscheme). Si se establece en [CookieAuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) , se proporciona el valor "cookies" para el esquema. `AuthenticationScheme` Puede proporcionar cualquier valor de cadena que distinga el esquema.

El esquema de autenticación de la aplicación es diferente del esquema de autenticación de cookies de la aplicación. Cuando no se proporciona un esquema de autenticación <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*>de cookies `CookieAuthenticationDefaults.AuthenticationScheme` a, USA ("cookies").

La propiedad de la <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> cookie de autenticación está `true` establecida de forma predeterminada en. Las cookies de autenticación se permiten cuando un visitante del sitio no ha dado su consentimiento a la recopilación de datos. Para obtener más información, consulta <xref:security/gdpr#essential-cookies>.

En el `Startup.Configure` método, `UseAuthentication` llame al método para invocar el middleware de autenticación que `HttpContext.User` establece la propiedad. Llame al `UseAuthentication` método antes de `UseMvcWithDefaultRoute` llamar `UseMvc`a o a:

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet2)]

La <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> clase se utiliza para configurar las opciones del proveedor de autenticación.

Establezca `CookieAuthenticationOptions` en la configuración del servicio para la autenticación `Startup.ConfigureServices` en el método:

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a>Middleware de directiva de cookies

El [middleware de directiva de cookies](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware) habilita las funcionalidades de directiva de cookies. Agregar el middleware a la canalización de procesamiento de la&mdash;aplicación es dependiente del orden. solo afecta a los componentes de nivel inferior registrados en la canalización.

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

Use <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> proporcionado al middleware de la Directiva de cookies para controlar las características globales del procesamiento de cookies y enlazar los controladores de procesamiento de cookies cuando se anexan o eliminan cookies.

El valor <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> predeterminado es `SameSiteMode.Lax` permitir la autenticación de OAuth2. Para aplicar estrictamente una directiva del mismo sitio de `SameSiteMode.Strict`, `MinimumSameSitePolicy`establezca. Aunque esta configuración interrumpe OAuth2 y otros esquemas de autenticación entre orígenes, eleva el nivel de seguridad de las cookies para otros tipos de aplicaciones que no se basan en el procesamiento de solicitudes entre orígenes.

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

La configuración de middleware de la `MinimumSameSitePolicy` Directiva de cookies para `Cookie.SameSite` puede `CookieAuthenticationOptions` afectar a la configuración de en la configuración de acuerdo con la matriz siguiente.

| MinimumSameSitePolicy | Cookie. SameSite | Valor de cookie. SameSite resultante |
| --------------------- | --------------- | --------------------------------- |
| SameSiteMode.None     | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict |
| SameSiteMode.Lax      | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.Lax<br>SameSiteMode.Lax<br>SameSiteMode.Strict |
| SameSiteMode.Strict   | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.Strict<br>SameSiteMode.Strict<br>SameSiteMode.Strict |

## <a name="create-an-authentication-cookie"></a>Crear una cookie de autenticación

Para crear una cookie que contiene información de usuario, <xref:System.Security.Claims.ClaimsPrincipal>construya un. La información del usuario se serializa y se almacena en la cookie. 

Cree un <xref:System.Security.Claims.ClaimsIdentity> con los s <xref:System.Security.Claims.Claim>necesarios y llame <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> al usuario para iniciar sesión:

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

`SignInAsync`crea una cookie cifrada y la agrega a la respuesta actual. Si `AuthenticationScheme` no se especifica, se usa el esquema predeterminado.

El sistema de [protección de datos](xref:security/data-protection/using-data-protection) de ASP.net Core se usa para el cifrado. En el caso de una aplicación hospedada en varios equipos, equilibrio de carga entre aplicaciones o uso de una granja de servidores Web, [Configure la protección de datos](xref:security/data-protection/configuration/overview) para que use el mismo anillo de claves e identificador de la aplicación.

## <a name="sign-out"></a>Cerrar sesión

Para cerrar la sesión del usuario actual y eliminar su cookie, <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>llame a:

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

Si `CookieAuthenticationDefaults.AuthenticationScheme` (o "cookies") no se usa como esquema (por ejemplo, "ContosoCookie"), proporcione el esquema que se usa al configurar el proveedor de autenticación. De lo contrario, se usa el esquema predeterminado.

## <a name="react-to-back-end-changes"></a>Reaccionar a los cambios de back-end

Una vez creada una cookie, ésta es la única fuente de identidad. Si una cuenta de usuario está deshabilitada en los sistemas de servidor:

* El sistema de autenticación de cookies de la aplicación continúa procesando solicitudes basadas en la cookie de autenticación.
* El usuario permanecerá conectado en la aplicación siempre y cuando la cookie de autenticación sea válida.

El <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> evento se puede utilizar para interceptar e invalidar la validación de la identidad de la cookie. La validación de la cookie en cada solicitud mitiga el riesgo de que los usuarios revocados accedan a la aplicación.

Un enfoque para la validación de cookies se basa en realizar un seguimiento de cuándo cambia la base de datos de usuario. Si no se ha cambiado la base de datos desde que se emitió la cookie del usuario, no es necesario volver a autenticar al usuario si la cookie sigue siendo válida. En la aplicación de ejemplo, la base de datos `IUserRepository` se implementa en `LastChanged` y almacena un valor. Cuando se actualiza un usuario en la base de datos `LastChanged` , el valor se establece en la hora actual.

Para invalidar una cookie cuando la base de datos cambie según el `LastChanged` valor, cree la cookie con una `LastChanged` solicitud que contenga el `LastChanged` valor actual de la base de datos:

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Email),
    new Claim("LastChanged", {Database Value})
};

var claimsIdentity = new ClaimsIdentity(
    claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme, 
    new ClaimsPrincipal(claimsIdentity));
```

Para implementar una invalidación para `ValidatePrincipal` el evento, escriba un método con la siguiente firma en una clase que derive de <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents>:

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

El siguiente es un ejemplo de implementación `CookieAuthenticationEvents`de:

```csharp
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;

public class CustomCookieAuthenticationEvents : CookieAuthenticationEvents
{
    private readonly IUserRepository _userRepository;

    public CustomCookieAuthenticationEvents(IUserRepository userRepository)
    {
        // Get the database from registered DI services.
        _userRepository = userRepository;
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        // Look for the LastChanged claim.
        var lastChanged = (from c in userPrincipal.Claims
                           where c.Type == "LastChanged"
                           select c.Value).FirstOrDefault();

        if (string.IsNullOrEmpty(lastChanged) ||
            !_userRepository.ValidateLastChanged(lastChanged))
        {
            context.RejectPrincipal();

            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

Registre la instancia de eventos durante el registro del servicio `Startup.ConfigureServices` de cookies en el método. Proporcione un [registro de servicio de ámbito](xref:fundamentals/dependency-injection#service-lifetimes) para `CustomCookieAuthenticationEvents` la clase:

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

Considere una situación en la que el nombre del usuario se&mdash;actualice una decisión que no afecte a la seguridad de ningún modo. Si desea actualizar de la entidad de seguridad de usuario de una no destructiva `context.ReplacePrincipal` , llame a `context.ShouldRenew` y establezca `true`la propiedad en.

> [!WARNING]
> El enfoque descrito aquí se desencadena en cada solicitud. La validación de las cookies de autenticación para todos los usuarios de cada solicitud puede producir una gran penalización del rendimiento de la aplicación.

## <a name="persistent-cookies"></a>Cookies persistentes

Puede que desee que la cookie se conserve entre sesiones del explorador. Esta persistencia solo se debe habilitar con el consentimiento explícito del usuario con una casilla "Recordarme" al iniciar sesión o un mecanismo similar. 

El siguiente fragmento de código crea una identidad y la cookie correspondiente que sobrevive a través de los cierres del explorador. Se respetan los valores de expiración que se hayan configurado previamente. Si la cookie expira mientras se cierra el explorador, el explorador borra la cookie una vez que se reinicia.

Se <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties>establece enen:`true`

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true
    });
```

## <a name="absolute-cookie-expiration"></a>Expiración absoluta de cookies

Se puede establecer una hora de expiración <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc>absoluta con. Para crear una cookie persistente, `IsPersistent` también se debe establecer. De lo contrario, la cookie se crea con una duración basada en sesión y puede expirar antes o después del vale de autenticación que contiene. Cuando `ExpiresUtc` se establece, reemplaza el valor de la <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.ExpireTimeSpan> opción de <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions>, si se establece.

El siguiente fragmento de código crea una identidad y la cookie correspondiente que dura 20 minutos. Esto omite cualquier configuración de expiración variable previamente configurada.

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
    });
```

::: moniker-end

## <a name="additional-resources"></a>Recursos adicionales

* <xref:security/authorization/limitingidentitybyscheme>
* <xref:security/authorization/claims>
* [Comprobaciones de roles basadas en directivas](xref:security/authorization/roles#policy-based-role-checks)
* <xref:host-and-deploy/web-farm>
