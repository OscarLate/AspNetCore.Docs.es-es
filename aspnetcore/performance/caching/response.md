---
title: Almacenamiento en caché de respuestas en ASP.NET Core
author: rick-anderson
description: Obtenga información sobre cómo usar el almacenamiento en caché de respuestas para reducir los requisitos de ancho de banda y aumentar el rendimiento de las aplicaciones de ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 10/15/2019
uid: performance/caching/response
ms.openlocfilehash: 4ebac97689347245d25e0954b33729d78dd1b516
ms.sourcegitcommit: dd026eceee79e943bd6b4a37b144803b50617583
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/15/2019
ms.locfileid: "72378832"
---
# <a name="response-caching-in-aspnet-core"></a>Almacenamiento en caché de respuestas en ASP.NET Core

Por [John Luo](https://github.com/JunTaoLuo), [Rick Anderson](https://twitter.com/RickAndMSFT), [Steve Smith](https://ardalis.com/)y [Luke Latham](https://github.com/guardrex)

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/response/samples) ([cómo descargarlo](xref:index#how-to-download-a-sample))

El almacenamiento en caché de respuestas reduce el número de solicitudes que un cliente o proxy realiza a un servidor Web. El almacenamiento en caché de respuestas también reduce la cantidad de trabajo que realiza el servidor web para generar una respuesta. El almacenamiento en caché de respuestas está controlado por encabezados que especifican cómo desea que el cliente, el proxy y el middleware almacenen en caché las respuestas.

El [atributo ResponseCache](#responsecache-attribute) participa en el establecimiento de encabezados de caché de respuesta, que los clientes pueden respetar al almacenar en caché las respuestas. El [middleware de almacenamiento en](xref:performance/caching/middleware) caché de respuestas se puede usar para almacenar en caché las respuestas en el servidor. El middleware puede usar las propiedades <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> para influir en el comportamiento del almacenamiento en caché del servidor.

## <a name="http-based-response-caching"></a>Almacenamiento en caché de respuesta basada en HTTP

La [especificación HTTP 1,1 Caching](https://tools.ietf.org/html/rfc7234) describe el comportamiento de las memorias caché de Internet. El encabezado HTTP principal usado para el almacenamiento en caché es [Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2), que se usa para especificar *directivas*de caché. Las directivas controlan el comportamiento de almacenamiento en caché a medida que las solicitudes proceden de los clientes a los servidores y, a medida que las respuestas, vuelven desde los servidores a los clientes. Las solicitudes y respuestas se desplazan a través de servidores proxy y los servidores proxy también deben cumplir con la especificación de almacenamiento en caché HTTP 1,1.

En la tabla siguiente se muestran las directivas comunes de `Cache-Control`.

| Directiva                                                       | Acción |
| --------------------------------------------------------------- | ------ |
| [public](https://tools.ietf.org/html/rfc7234#section-5.2.2.5)   | Una memoria caché puede almacenar la respuesta. |
| [private](https://tools.ietf.org/html/rfc7234#section-5.2.2.6)  | La respuesta no debe almacenarse en una memoria caché compartida. Una caché privada puede almacenar y reutilizar la respuesta. |
| [Max-Age](https://tools.ietf.org/html/rfc7234#section-5.2.1.1)  | El cliente no acepta una respuesta cuya antigüedad sea mayor que el número de segundos especificado. Ejemplos: `max-age=60` (60 segundos), `max-age=2592000` (1 mes) |
| [no-cache](https://tools.ietf.org/html/rfc7234#section-5.2.1.4) | **En las solicitudes**: una memoria caché no debe usar una respuesta almacenada para satisfacer la solicitud. El servidor de origen regenera la respuesta para el cliente y el middleware actualiza la respuesta almacenada en su caché.<br><br>**En las respuestas**: la respuesta no se debe utilizar para una solicitud subsiguiente sin validación en el servidor de origen. |
| [sin almacén](https://tools.ietf.org/html/rfc7234#section-5.2.1.5) | **En las solicitudes**: una memoria caché no debe almacenar la solicitud.<br><br>**En las respuestas**: una memoria caché no debe almacenar ninguna parte de la respuesta. |

En la tabla siguiente se muestran otros encabezados de caché que desempeñan un rol en el almacenamiento en caché.

| Header                                                     | Función |
| ---------------------------------------------------------- | -------- |
| [Antig](https://tools.ietf.org/html/rfc7234#section-5.1)     | Una estimación de la cantidad de tiempo en segundos transcurrido desde que se generó la respuesta o se validó correctamente en el servidor de origen. |
| [Expira](https://tools.ietf.org/html/rfc7234#section-5.3) | Hora a partir de la cual la respuesta se considera obsoleta. |
| [Omiti](https://tools.ietf.org/html/rfc7234#section-5.4)  | Existe por compatibilidad con versiones anteriores de caché HTTP/1.0 para establecer el comportamiento de `no-cache`. Si el encabezado `Cache-Control` está presente, se omite el encabezado `Pragma`. |
| [Variaciones](https://tools.ietf.org/html/rfc7231#section-7.1.4)  | Especifica que no se debe enviar una respuesta almacenada en caché a menos que todos los campos de encabezado `Vary` coincidan con la solicitud original de la respuesta almacenada en caché y la nueva solicitud. |

## <a name="http-based-caching-respects-request-cache-control-directives"></a>El almacenamiento en caché basado en HTTP respeta las directivas de control de caché de solicitudes

La [especificación HTTP 1,1 Caching del encabezado Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2) requiere una memoria caché para respetar un encabezado `Cache-Control` válido enviado por el cliente. Un cliente puede realizar solicitudes con un valor de encabezado `no-cache` y obligar al servidor a generar una nueva respuesta para cada solicitud.

Tener en cuenta los encabezados de solicitud de Client `Cache-Control` siempre tiene sentido si tiene en cuenta el objetivo del almacenamiento en caché de HTTP. En la especificación oficial, el almacenamiento en caché está pensado para reducir la latencia y la sobrecarga de red que supone satisfacer las solicitudes a través de una red de clientes, servidores proxy y servidores. No es necesariamente una manera de controlar la carga en un servidor de origen.

No hay ningún control de desarrollador sobre este comportamiento de almacenamiento en caché cuando se usa el [middleware de almacenamiento en caché de respuesta](xref:performance/caching/middleware) , ya que el middleware se adhiere a la especificación oficial de almacenamiento en caché. [Las mejoras planeadas en el middleware](https://github.com/aspnet/AspNetCore/issues/2612) son una oportunidad para configurar el middleware para omitir el encabezado de `Cache-Control` de una solicitud al decidir si se va a dar servicio a una respuesta almacenada en caché. Las mejoras planeadas proporcionan una oportunidad para controlar mejor la carga del servidor.

## <a name="other-caching-technology-in-aspnet-core"></a>Otra tecnología de almacenamiento en caché en ASP.NET Core

### <a name="in-memory-caching"></a>Almacenamiento en caché en memoria

El almacenamiento en caché en memoria utiliza la memoria del servidor para almacenar los datos en caché. Este tipo de almacenamiento en caché es adecuado para un solo servidor o para varios servidores que usan *sesiones permanentes*. Las sesiones permanentes significan que las solicitudes de un cliente siempre se enrutan al mismo servidor para su procesamiento.

Para obtener más información, vea <xref:performance/caching/memory>.

### <a name="distributed-cache"></a>Caché distribuida

Use una caché distribuida para almacenar los datos en memoria cuando la aplicación se hospeda en una granja de servidores o en la nube. La memoria caché se comparte entre los servidores que procesan las solicitudes. Un cliente puede enviar una solicitud controlada por cualquier servidor del grupo si están disponibles los datos almacenados en caché del cliente. ASP.NET Core ofrece memorias caché distribuidas SQL Server y Redis.

Para obtener más información, vea <xref:performance/caching/distributed>.

### <a name="cache-tag-helper"></a>Aplicación auxiliar de etiquetas de caché

Almacene en caché el contenido de una vista de MVC o una página de Razor con la aplicación auxiliar de etiquetas de caché. La aplicación auxiliar de etiquetas de caché usa el almacenamiento en caché en memoria para almacenar los datos.

Para obtener más información, vea <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>.

### <a name="distributed-cache-tag-helper"></a>Asistente de etiquetas de caché distribuida

Almacene en caché el contenido de una vista de MVC o una página de Razor en escenarios de granja de servidores web o nube distribuida con la aplicación auxiliar de etiquetas de caché distribuida. La aplicación auxiliar de etiquetas de caché distribuida usa SQL Server o Redis para almacenar datos.

Para obtener más información, vea <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>.

## <a name="responsecache-attribute"></a>Atributo ResponseCache

El <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> especifica los parámetros necesarios para establecer los encabezados adecuados en el almacenamiento en caché de las respuestas.

> [!WARNING]
> Deshabilite el almacenamiento en caché para el contenido que contiene información de los clientes autenticados. El almacenamiento en caché solo debe estar habilitado para el contenido que no cambia en función de la identidad de un usuario o de si un usuario ha iniciado sesión.

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> varía la respuesta almacenada por los valores de la lista de claves de consulta dada. Cuando se proporciona un valor único de `*`, el middleware varía las respuestas de todos los parámetros de la cadena de consulta de solicitud.

El [middleware de almacenamiento en caché de respuestas](xref:performance/caching/middleware) debe estar habilitado para establecer la propiedad <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys>. De lo contrario, se produce una excepción en tiempo de ejecución. No hay un encabezado HTTP correspondiente para la propiedad <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys>. La propiedad es una característica de HTTP administrada por middleware de almacenamiento en caché de respuestas. Para que el middleware atienda una respuesta almacenada en caché, la cadena de consulta y el valor de la cadena de consulta deben coincidir con una solicitud anterior. Por ejemplo, considere la secuencia de solicitudes y los resultados que se muestran en la tabla siguiente.

| Request                          | Resultado                    |
| -------------------------------- | ------------------------- |
| `http://example.com?key1=value1` | Se devuelve desde el servidor. |
| `http://example.com?key1=value1` | Se devuelve desde middleware. |
| `http://example.com?key1=value2` | Se devuelve desde el servidor. |

El servidor devuelve la primera solicitud y la almacena en caché en middleware. El middleware devuelve la segunda solicitud porque la cadena de consulta coincide con la solicitud anterior. La tercera solicitud no está en la caché de middleware porque el valor de la cadena de consulta no coincide con una solicitud anterior.

El <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> se usa para configurar y crear (a través de <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>) un `Microsoft.AspNetCore.Mvc.Internal.ResponseCacheFilter`. El `ResponseCacheFilter` realiza el trabajo de actualizar los encabezados HTTP y las características correspondientes de la respuesta. El filtro:

* Quita todos los encabezados existentes para `Vary`, `Cache-Control` y `Pragma`.
* Escribe los encabezados adecuados en función de las propiedades establecidas en el <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute>.
* Actualiza la característica HTTP de almacenamiento en caché de respuesta si se establece <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys>.

### <a name="vary"></a>Variaciones

Este encabezado solo se escribe cuando se establece la propiedad <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByHeader>. Propiedad establecida en el valor de la propiedad `Vary`. En el ejemplo siguiente se usa la propiedad <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByHeader>:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache1.cshtml.cs?name=snippet)]

Use la aplicación de ejemplo para ver los encabezados de respuesta con las herramientas de red del explorador. Los siguientes encabezados de respuesta se envían con la respuesta de la página Cache1:

```
Cache-Control: public,max-age=30
Vary: User-Agent
```

### <a name="nostore-and-locationnone"></a>Nostore y Location. None

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> invalida la mayoría de las demás propiedades. Cuando esta propiedad se establece en `true`, el encabezado `Cache-Control` se establece en `no-store`. Si <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> está establecido en `None`:

* El valor de `Cache-Control` está establecido en `no-store,no-cache`.
* El valor de `Pragma` está establecido en `no-cache`.

Si <xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> es `false` y <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> es `None`, `Cache-Control` y `Pragma` se establecen en `no-cache`.

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> normalmente se establece en `true` para las páginas de error. La página Cache2 de la aplicación de ejemplo genera encabezados de respuesta que indican al cliente que no almacene la respuesta.

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache2.cshtml.cs?name=snippet)]

La aplicación de ejemplo devuelve la página Cache2 con los encabezados siguientes:

```
Cache-Control: no-store,no-cache
Pragma: no-cache
```

### <a name="location-and-duration"></a>Ubicación y duración

Para habilitar el almacenamiento en caché, <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Duration> debe establecerse en un valor positivo y <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> debe ser `Any` (valor predeterminado) o `Client`. En este caso, el encabezado `Cache-Control` se establece en el valor de ubicación seguido del `max-age` de la respuesta.

> [!NOTE]
> las opciones de <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> de `Any` y `Client` se traducen en valores de encabezado `Cache-Control` de `public` y `private`, respectivamente. Como se indicó anteriormente, si se establece <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> en `None`, se establecen los encabezados @no__t 2 y `Pragma` en `no-cache`.

En el ejemplo siguiente se muestra el modelo de página Cache3 de la aplicación de ejemplo y los encabezados generados al establecer <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Duration> y se sale del valor predeterminado <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location>:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache3.cshtml.cs?name=snippet)]

La aplicación de ejemplo devuelve la página Cache3 con el siguiente encabezado:

```
Cache-Control: public,max-age=10
```

### <a name="cache-profiles"></a>Perfiles de caché

En lugar de duplicar la configuración de la caché de respuesta en muchos atributos de acción del controlador, los perfiles de caché se pueden configurar como opciones al configurar MVC/Razor Pages en `Startup.ConfigureServices`. Los valores que se encuentran en un perfil de caché al que se hace referencia se utilizan como valores predeterminados por el <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> y se reemplazan por las propiedades especificadas en el atributo.

Configure un perfil de caché. En el ejemplo siguiente se muestra un perfil de caché de 30 segundos en el @no__t de la aplicación de ejemplo:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Startup.cs?name=snippet1)]

El modelo de página Cache4 de la aplicación de ejemplo hace referencia al perfil de caché `Default30`:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache4.cshtml.cs?name=snippet)]

El <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> se puede aplicar a:

* Los controladores de páginas de Razor (clases) &ndash; no se pueden aplicar a los métodos de controlador.
* Controladores MVC (clases).
* Acciones de MVC (métodos) &ndash; los atributos de nivel de método invalidan los valores especificados en los atributos de nivel de clase.

El encabezado resultante aplicado a la respuesta de la página Cache4 por el perfil de caché `Default30`:

```
Cache-Control: public,max-age=30
```

## <a name="additional-resources"></a>Recursos adicionales

* [Almacenar respuestas en cachés](https://tools.ietf.org/html/rfc7234#section-3)
* [Cache-control](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
