---
title: Conservar notificaciones y tokens adicionales de proveedores externos en ASP.NET Core
author: guardrex
description: Obtenga información sobre cómo establecer notificaciones y tokens adicionales de proveedores externos.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
uid: security/authentication/social/additional-claims
ms.openlocfilehash: 72710d249d3210208dd9b0356a700ba02a0b727a
ms.sourcegitcommit: dd026eceee79e943bd6b4a37b144803b50617583
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/15/2019
ms.locfileid: "72378882"
---
# <a name="persist-additional-claims-and-tokens-from-external-providers-in-aspnet-core"></a>Conservar notificaciones y tokens adicionales de proveedores externos en ASP.NET Core

Por [Luke Latham](https://github.com/guardrex)

::: moniker range=">= aspnetcore-3.0"

Una aplicación ASP.NET Core puede establecer notificaciones y tokens adicionales de proveedores de autenticación externos, como Facebook, Google, Microsoft y Twitter. Cada proveedor revela información diferente sobre los usuarios en su plataforma, pero el patrón para recibir y transformar los datos de usuario en notificaciones adicionales es el mismo.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/social/additional-claims/samples) ([cómo descargarlo](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Requisitos previos

Decida qué proveedores de autenticación externos admitir en la aplicación. Para cada proveedor, registre la aplicación y obtenga un identificador de cliente y un secreto de cliente. Para obtener más información, vea <xref:security/authentication/social/index>. La aplicación de ejemplo usa el [proveedor de autenticación de Google](xref:security/authentication/google-logins).

## <a name="set-the-client-id-and-client-secret"></a>Establecimiento del identificador de cliente y el secreto de cliente

El proveedor de autenticación OAuth establece una relación de confianza con una aplicación que usa un identificador de cliente y un secreto de cliente. El proveedor de autenticación externo crea los valores de identificador de cliente y secreto de cliente para la aplicación cuando la aplicación se registra con el proveedor. Cada proveedor externo que use la aplicación debe configurarse de forma independiente con el identificador de cliente y el secreto de cliente del proveedor. Para obtener más información, consulte los temas de proveedores de autenticación externos que se aplican a su escenario:

* [Autenticación con Facebook](xref:security/authentication/facebook-logins)
* [Autenticación con Google](xref:security/authentication/google-logins)
* [Autenticación con Microsoft](xref:security/authentication/microsoft-logins)
* [Autenticación con Twitter](xref:security/authentication/twitter-logins)
* [Otros proveedores de autenticación](xref:security/authentication/otherlogins)
* [OpenIdConnect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

La aplicación de ejemplo configura el proveedor de autenticación de Google con un identificador de cliente y un secreto de cliente proporcionado por Google:

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=4,9)]

## <a name="establish-the-authentication-scope"></a>Establecer el ámbito de autenticación

Especifique la lista de permisos que se van a recuperar del proveedor especificando el <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*>. Los ámbitos de autenticación para proveedores externos comunes aparecen en la tabla siguiente.

| Proveedor  | Ámbito                                                            |
| --------- | ---------------------------------------------------------------- |
| Facebook  | `https://www.facebook.com/dialog/oauth`                          |
| Google    | `https://www.googleapis.com/auth/userinfo.profile`               |
| Microsoft | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Twitter   | `https://api.twitter.com/oauth/authenticate`                     |

En la aplicación de ejemplo, el marco de trabajo agrega automáticamente el ámbito `userinfo.profile` de Google cuando se llama a <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle*> en el @no__t 2. Si la aplicación requiere ámbitos adicionales, agréguelos a las opciones. En el ejemplo siguiente, se agrega el ámbito de Google `https://www.googleapis.com/auth/user.birthday.read` para recuperar el cumpleaños de un usuario:

```csharp
options.Scope.Add("https://www.googleapis.com/auth/user.birthday.read");
```

## <a name="map-user-data-keys-and-create-claims"></a>Asignar claves de datos de usuario y crear notificaciones

En las opciones del proveedor, especifique un <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> o <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> para cada clave o subclave en los datos de usuario JSON del proveedor externo para que la identidad de la aplicación Lea el inicio de sesión. Para obtener más información sobre los tipos de notificaciones, vea <xref:System.Security.Claims.ClaimTypes>.

La aplicación de ejemplo crea notificaciones de configuración regional (`urn:google:locale`) e imagen (`urn:google:picture`) de las claves @no__t 2 y `picture` en los datos de usuario de Google:

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=13-14)]

En `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync`, un <xref:Microsoft.AspNetCore.Identity.IdentityUser> (`ApplicationUser`) se inicia sesión en la aplicación con <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*>. Durante el proceso de inicio de sesión, el <xref:Microsoft.AspNetCore.Identity.UserManager%601> puede almacenar notificaciones `ApplicationUser` para los datos de usuario disponibles en el @no__t 2.

En la aplicación de ejemplo, `OnPostConfirmationAsync` (*account/ExternalLogin. cshtml. CS*) establece las notificaciones de configuración regional (`urn:google:locale`) e imagen (`urn:google:picture`) del `ApplicationUser` con sesión iniciada, incluida una notificación de <xref:System.Security.Claims.ClaimTypes.GivenName>:

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=35-51)]

De forma predeterminada, las notificaciones de un usuario se almacenan en la cookie de autenticación. Si la cookie de autenticación es demasiado grande, puede provocar un error en la aplicación porque:

* El explorador detecta que el encabezado de la cookie es demasiado largo.
* El tamaño total de la solicitud es demasiado grande.

Si se requiere una gran cantidad de datos de usuario para el procesamiento de solicitudes de usuario:

* Limite el número y el tamaño de las notificaciones de usuario para el procesamiento de solicitudes solo a lo que requiere la aplicación.
* Use un @no__t personalizado-0 para el middleware de autenticación de cookies <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> para almacenar la identidad entre las solicitudes. Conservar grandes cantidades de información de identidad en el servidor y enviar solo una pequeña clave de identificador de sesión al cliente.

## <a name="save-the-access-token"></a>Guardar el token de acceso

<xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> define si los tokens de acceso y de actualización deben almacenarse en el <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> después de una autorización correcta. de forma predeterminada, `SaveTokens` se establece en `false` para reducir el tamaño de la cookie de autenticación final.

La aplicación de ejemplo establece el valor de `SaveTokens` en `true` en <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions>:

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=15)]

Cuando se ejecuta `OnPostConfirmationAsync`, almacene el token de acceso ([ExternalLoginInfo. AuthenticationTokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) del proveedor externo en `AuthenticationProperties` de `ApplicationUser`.

La aplicación de ejemplo guarda el token de acceso en `OnPostConfirmationAsync` (nuevo registro de usuario) y `OnGetCallbackAsync` (usuario registrado previamente) en *account/ExternalLogin. cshtml. CS*:

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=54-56)]

## <a name="how-to-add-additional-custom-tokens"></a>Cómo agregar tokens personalizados adicionales

Para mostrar cómo agregar un token personalizado, que se almacena como parte de `SaveTokens`, la aplicación de ejemplo agrega un <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> con el @no__t actual-2 para un [AuthenticationToken.Name](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) de `TicketCreated`:

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=17-30)]

## <a name="creating-and-adding-claims"></a>Creación y adición de notificaciones

El marco de trabajo proporciona acciones comunes y métodos de extensión para crear y agregar notificaciones a la colección. Para obtener más información, vea <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> y <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>.

Los usuarios pueden definir acciones personalizadas derivando de <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> e implementando el método Abstract <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*>.

Para obtener más información, vea <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>.

## <a name="removal-of-claim-actions-and-claims"></a>Eliminación de las notificaciones y las acciones de notificación

[ClaimActionCollection. Remove (String)](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) quita todas las acciones de notificaciones para el @no__t especificado-1 de la colección. [ClaimActionCollectionMapExtensions. DeleteClaim (ClaimActionCollection, String)](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) elimina una demanda del <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> especificado de la identidad. <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> se usa principalmente con [OpenID Connect (OIDC)](/azure/active-directory/develop/v2-protocols-oidc) para quitar notificaciones generadas por el protocolo.

## <a name="sample-app-output"></a>Salida de la aplicación de ejemplo

```
User Claims

http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier
    9b342344f-7aab-43c2-1ac1-ba75912ca999
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
    someone@gmail.com
AspNet.Identity.SecurityStamp
    7D4312MOWRYYBFI1KXRPHGOSTBVWSFDE
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
    Judy
urn:google:locale
    en
urn:google:picture
    https://lh4.googleusercontent.com/-XXXXXX/XXXXXX/XXXXXX/XXXXXX/photo.jpg

Authentication Properties

.Token.access_token
    yc23.AlvoZqz56...1lxltXV7D-ZWP9
.Token.token_type
    Bearer
.Token.expires_at
    2019-04-11T22:14:51.0000000+00:00
.Token.TicketCreated
    4/11/2019 9:14:52 PM
.TokenNames
    access_token;token_type;expires_at;TicketCreated
.persistent
.issued
    Thu, 11 Apr 2019 20:51:06 GMT
.expires
    Thu, 25 Apr 2019 20:51:06 GMT

```

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Una aplicación ASP.NET Core puede establecer notificaciones y tokens adicionales de proveedores de autenticación externos, como Facebook, Google, Microsoft y Twitter. Cada proveedor revela información diferente sobre los usuarios en su plataforma, pero el patrón para recibir y transformar los datos de usuario en notificaciones adicionales es el mismo.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/social/additional-claims/samples) ([cómo descargarlo](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Requisitos previos

Decida qué proveedores de autenticación externos admitir en la aplicación. Para cada proveedor, registre la aplicación y obtenga un identificador de cliente y un secreto de cliente. Para obtener más información, vea <xref:security/authentication/social/index>. La aplicación de ejemplo usa el [proveedor de autenticación de Google](xref:security/authentication/google-logins).

## <a name="set-the-client-id-and-client-secret"></a>Establecimiento del identificador de cliente y el secreto de cliente

El proveedor de autenticación OAuth establece una relación de confianza con una aplicación que usa un identificador de cliente y un secreto de cliente. El proveedor de autenticación externo crea los valores de identificador de cliente y secreto de cliente para la aplicación cuando la aplicación se registra con el proveedor. Cada proveedor externo que use la aplicación debe configurarse de forma independiente con el identificador de cliente y el secreto de cliente del proveedor. Para obtener más información, consulte los temas de proveedores de autenticación externos que se aplican a su escenario:

* [Autenticación con Facebook](xref:security/authentication/facebook-logins)
* [Autenticación con Google](xref:security/authentication/google-logins)
* [Autenticación con Microsoft](xref:security/authentication/microsoft-logins)
* [Autenticación con Twitter](xref:security/authentication/twitter-logins)
* [Otros proveedores de autenticación](xref:security/authentication/otherlogins)
* [OpenIdConnect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

La aplicación de ejemplo configura el proveedor de autenticación de Google con un identificador de cliente y un secreto de cliente proporcionado por Google:

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=4,9)]

## <a name="establish-the-authentication-scope"></a>Establecer el ámbito de autenticación

Especifique la lista de permisos que se van a recuperar del proveedor especificando el <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*>. Los ámbitos de autenticación para proveedores externos comunes aparecen en la tabla siguiente.

| Proveedor  | Ámbito                                                            |
| --------- | ---------------------------------------------------------------- |
| Facebook  | `https://www.facebook.com/dialog/oauth`                          |
| Google    | `https://www.googleapis.com/auth/userinfo.profile`               |
| Microsoft | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Twitter   | `https://api.twitter.com/oauth/authenticate`                     |

En la aplicación de ejemplo, el marco de trabajo agrega automáticamente el ámbito `userinfo.profile` de Google cuando se llama a <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle*> en el @no__t 2. Si la aplicación requiere ámbitos adicionales, agréguelos a las opciones. En el ejemplo siguiente, se agrega el ámbito de Google `https://www.googleapis.com/auth/user.birthday.read` para recuperar el cumpleaños de un usuario:

```csharp
options.Scope.Add("https://www.googleapis.com/auth/user.birthday.read");
```

## <a name="map-user-data-keys-and-create-claims"></a>Asignar claves de datos de usuario y crear notificaciones

En las opciones del proveedor, especifique un <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> o <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> para cada clave o subclave en los datos de usuario JSON del proveedor externo para que la identidad de la aplicación Lea el inicio de sesión. Para obtener más información sobre los tipos de notificaciones, vea <xref:System.Security.Claims.ClaimTypes>.

La aplicación de ejemplo crea notificaciones de configuración regional (`urn:google:locale`) e imagen (`urn:google:picture`) de las claves @no__t 2 y `picture` en los datos de usuario de Google:

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=13-14)]

En `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync`, un <xref:Microsoft.AspNetCore.Identity.IdentityUser> (`ApplicationUser`) se inicia sesión en la aplicación con <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*>. Durante el proceso de inicio de sesión, el <xref:Microsoft.AspNetCore.Identity.UserManager%601> puede almacenar notificaciones `ApplicationUser` para los datos de usuario disponibles en el @no__t 2.

En la aplicación de ejemplo, `OnPostConfirmationAsync` (*account/ExternalLogin. cshtml. CS*) establece las notificaciones de configuración regional (`urn:google:locale`) e imagen (`urn:google:picture`) del `ApplicationUser` con sesión iniciada, incluida una notificación de <xref:System.Security.Claims.ClaimTypes.GivenName>:

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=35-51)]

De forma predeterminada, las notificaciones de un usuario se almacenan en la cookie de autenticación. Si la cookie de autenticación es demasiado grande, puede provocar un error en la aplicación porque:

* El explorador detecta que el encabezado de la cookie es demasiado largo.
* El tamaño total de la solicitud es demasiado grande.

Si se requiere una gran cantidad de datos de usuario para el procesamiento de solicitudes de usuario:

* Limite el número y el tamaño de las notificaciones de usuario para el procesamiento de solicitudes solo a lo que requiere la aplicación.
* Use un @no__t personalizado-0 para el middleware de autenticación de cookies <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> para almacenar la identidad entre las solicitudes. Conservar grandes cantidades de información de identidad en el servidor y enviar solo una pequeña clave de identificador de sesión al cliente.

## <a name="save-the-access-token"></a>Guardar el token de acceso

<xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> define si los tokens de acceso y de actualización deben almacenarse en el <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> después de una autorización correcta. de forma predeterminada, `SaveTokens` se establece en `false` para reducir el tamaño de la cookie de autenticación final.

La aplicación de ejemplo establece el valor de `SaveTokens` en `true` en <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions>:

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=15)]

Cuando se ejecuta `OnPostConfirmationAsync`, almacene el token de acceso ([ExternalLoginInfo. AuthenticationTokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) del proveedor externo en `AuthenticationProperties` de `ApplicationUser`.

La aplicación de ejemplo guarda el token de acceso en `OnPostConfirmationAsync` (nuevo registro de usuario) y `OnGetCallbackAsync` (usuario registrado previamente) en *account/ExternalLogin. cshtml. CS*:

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=54-56)]

## <a name="how-to-add-additional-custom-tokens"></a>Cómo agregar tokens personalizados adicionales

Para mostrar cómo agregar un token personalizado, que se almacena como parte de `SaveTokens`, la aplicación de ejemplo agrega un <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> con el @no__t actual-2 para un [AuthenticationToken.Name](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) de `TicketCreated`:

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=17-30)]

## <a name="creating-and-adding-claims"></a>Creación y adición de notificaciones

El marco de trabajo proporciona acciones comunes y métodos de extensión para crear y agregar notificaciones a la colección. Para obtener más información, vea <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> y <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>.

Los usuarios pueden definir acciones personalizadas derivando de <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> e implementando el método Abstract <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*>.

Para obtener más información, vea <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>.

## <a name="removal-of-claim-actions-and-claims"></a>Eliminación de las notificaciones y las acciones de notificación

[ClaimActionCollection. Remove (String)](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) quita todas las acciones de notificaciones para el @no__t especificado-1 de la colección. [ClaimActionCollectionMapExtensions. DeleteClaim (ClaimActionCollection, String)](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) elimina una demanda del <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> especificado de la identidad. <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> se usa principalmente con [OpenID Connect (OIDC)](/azure/active-directory/develop/v2-protocols-oidc) para quitar notificaciones generadas por el protocolo.

## <a name="sample-app-output"></a>Salida de la aplicación de ejemplo

```
User Claims

http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier
    9b342344f-7aab-43c2-1ac1-ba75912ca999
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
    someone@gmail.com
AspNet.Identity.SecurityStamp
    7D4312MOWRYYBFI1KXRPHGOSTBVWSFDE
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
    Judy
urn:google:locale
    en
urn:google:picture
    https://lh4.googleusercontent.com/-XXXXXX/XXXXXX/XXXXXX/XXXXXX/photo.jpg

Authentication Properties

.Token.access_token
    yc23.AlvoZqz56...1lxltXV7D-ZWP9
.Token.token_type
    Bearer
.Token.expires_at
    2019-04-11T22:14:51.0000000+00:00
.Token.TicketCreated
    4/11/2019 9:14:52 PM
.TokenNames
    access_token;token_type;expires_at;TicketCreated
.persistent
.issued
    Thu, 11 Apr 2019 20:51:06 GMT
.expires
    Thu, 25 Apr 2019 20:51:06 GMT

```

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

::: moniker-end

## <a name="additional-resources"></a>Recursos adicionales

* [ASPNET/AspNetCore Engineering SocialSample app](https://github.com/aspnet/AspNetCore/tree/master/src/Security/Authentication/samples/SocialSample) &ndash; la aplicación de ejemplo vinculada se encuentra en la rama de ingeniería de @no__t 3 [del repositorio de github de ASPNET/AspNetCore](https://github.com/aspnet/AspNetCore) . La rama `master` contiene código bajo desarrollo activo para la próxima versión de ASP.NET Core. Para ver una versión de la aplicación de ejemplo de una versión de lanzamiento de ASP.NET Core, use la lista desplegable **rama** para seleccionar una rama de versión (por ejemplo `release/{X.Y}`).
