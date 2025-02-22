---
title: Almacenamiento en caché distribuido en ASP.NET Core
author: guardrex
description: Aprenda a usar una ASP.NET Core caché distribuida para mejorar el rendimiento y la escalabilidad de las aplicaciones, especialmente en un entorno de granja de servidores o en la nube.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/27/2019
uid: performance/caching/distributed
ms.openlocfilehash: dbcdfcd07877fabfe6d18cd4d840b5597afa1afd
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/18/2019
ms.locfileid: "71081546"
---
# <a name="distributed-caching-in-aspnet-core"></a>Almacenamiento en caché distribuido en ASP.NET Core

Por [Luke Latham](https://github.com/guardrex) y [Steve Smith](https://ardalis.com/)

Una caché distribuida es una memoria caché compartida por varios servidores de aplicaciones, que normalmente se mantiene como un servicio externo a los servidores de aplicaciones que tienen acceso a ella. Una caché distribuida puede mejorar el rendimiento y la escalabilidad de una aplicación ASP.NET Core, especialmente cuando la aplicación está hospedada en un servicio en la nube o en una granja de servidores.

Una caché distribuida tiene varias ventajas con respecto a otros escenarios de almacenamiento en caché en los que los datos almacenados en caché se almacenan en servidores de aplicaciones individuales.

Cuando se distribuyen los datos almacenados en caché, los datos:

* Es *coherente (coherente)* entre las solicitudes a varios servidores.
* Sobrevive los reinicios del servidor y las implementaciones de aplicaciones.
* No usa la memoria local.

La configuración de caché distribuida es específica de la implementación. En este artículo se describe cómo configurar SQL Server y las cachés distribuidas de Redis. También hay implementaciones de terceros disponibles, como [NCache](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache en github](https://github.com/Alachisoft/NCache)). Independientemente de la implementación seleccionada, la aplicación interactúa con la memoria caché mediante la <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaz.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/distributed/samples/) ([cómo descargarlo](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Requisitos previos

::: moniker range=">= aspnetcore-3.0"

Para usar una SQL Server caché distribuida, agregue una referencia de paquete al paquete [Microsoft. Extensions. Caching. SqlServer](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer) .

Para usar una caché distribuida de Redis, agregue una referencia de paquete al paquete [Microsoft. Extensions. Caching. StackExchangeRedis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis) .

::: moniker-end

::: moniker range="= aspnetcore-2.2"

Para usar una SQL Server caché distribuida, haga referencia al [metapaquete Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) o agregue una referencia de paquete al paquete [Microsoft. Extensions. Caching. SqlServer](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer) .

Para usar una caché distribuida de Redis, haga referencia al [metapaquete Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) y agregue una referencia de paquete al paquete [Microsoft. Extensions. Caching. StackExchangeRedis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis) . El paquete de Redis no está incluido `Microsoft.AspNetCore.App` en el paquete, por lo que debe hacer referencia al paquete de Redis por separado en el archivo de proyecto.

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Para usar una SQL Server caché distribuida, haga referencia al [metapaquete Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) o agregue una referencia de paquete al paquete [Microsoft. Extensions. Caching. SqlServer](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer) .

Para usar una caché distribuida de Redis, haga referencia al [metapaquete Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) y agregue una referencia de paquete al paquete [Microsoft. Extensions. Caching. Redis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Redis) . El paquete de Redis no está incluido `Microsoft.AspNetCore.App` en el paquete, por lo que debe hacer referencia al paquete de Redis por separado en el archivo de proyecto.

::: moniker-end

## <a name="idistributedcache-interface"></a>Interfaz IDistributedCache

La <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaz proporciona los métodos siguientes para manipular los elementos de la implementación de caché distribuida:

* <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Get*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.GetAsync*> `byte[]` Acepta una clave de cadena y recupera un elemento almacenado en caché como una matriz, si se encuentra en la memoria caché. &ndash;
* <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Set*>`byte[]` , <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.SetAsync*> Agrega&ndash; un elemento (como matriz) a la memoria caché utilizando una clave de cadena.
* <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Refresh*><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RefreshAsync*> ,&ndash; Actualiza un elemento en la memoria caché en función de su clave, restableciendo el tiempo de espera de expiración variable (si existe).
* <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Remove*><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RemoveAsync*> ,&ndash; Quita un elemento de la memoria caché en función de su clave de cadena.

## <a name="establish-distributed-caching-services"></a>Establecer servicios de almacenamiento en caché distribuidos

Registre una implementación de <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> en `Startup.ConfigureServices`. Las implementaciones proporcionadas por el marco de trabajo que se describen en este tema incluyen:

* [Caché de memoria distribuida](#distributed-memory-cache)
* [Caché de SQL Server distribuida](#distributed-sql-server-cache)
* [Caché en Redis distribuida](#distributed-redis-cache)

### <a name="distributed-memory-cache"></a>Caché de memoria distribuida

La memoria caché de memoria<xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddDistributedMemoryCache*>distribuida () es una implementación proporcionada <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> por el marco de que almacena elementos en la memoria. La caché de memoria distribuida no es una caché distribuida real. Los elementos almacenados en caché se almacenan en la instancia de la aplicación en el servidor donde se ejecuta la aplicación.

La caché de memoria distribuida es una implementación útil:

* En escenarios de desarrollo y pruebas.
* Cuando se usa un solo servidor en producción y el consumo de memoria no es un problema. La implementación de la caché de memoria distribuida abstrae el almacenamiento de datos en caché. Permite implementar una solución de almacenamiento en caché distribuida verdadera en el futuro si se necesitan varios nodos o tolerancia a errores.

La aplicación de ejemplo hace uso de la caché de memoria distribuida cuando la aplicación se ejecuta en el `Startup.ConfigureServices`entorno de desarrollo en:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedMemoryCache)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedMemoryCache)]

::: moniker-end

### <a name="distributed-sql-server-cache"></a>Caché de SQL Server distribuida

La implementación de la memoria caché<xref:Microsoft.Extensions.DependencyInjection.SqlServerCachingServicesExtensions.AddDistributedSqlServerCache*>de SQL Server distribuida () permite que la memoria caché distribuida use una base de datos SQL Server como su memoria auxiliar. Para crear una SQL Server tabla de elementos almacenados en caché en una instancia de SQL Server, puede `sql-cache` usar la herramienta. La herramienta crea una tabla con el nombre y el esquema que especifique.

Cree una tabla en SQL Server ejecutando el `sql-cache create` comando. Proporcione la SQL Server instancia (`Data Source`), la base`Initial Catalog`de datos (), el esquema `dbo`(por ejemplo,) y el nombre de `TestCache`la tabla (por ejemplo,):

```dotnetcli
dotnet sql-cache create "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=DistCache;Integrated Security=True;" dbo TestCache
```

Se registra un mensaje para indicar que la herramienta se ha realizado correctamente:

```console
Table and index were created successfully.
```

La tabla creada por la `sql-cache` herramienta tiene el siguiente esquema:

![Tabla de caché SqlServer](distributed/_static/SqlServerCacheTable.png)

> [!NOTE]
> Una aplicación debe manipular los valores de la memoria caché mediante <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>una instancia de <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache>, no un.

La aplicación <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> de ejemplo implementa en un entorno que no es de `Startup.ConfigureServices`desarrollo en:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedSqlServerCache)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedSqlServerCache)]

::: moniker-end

> [!NOTE]
> Un <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.ConnectionString*> (y, opcionalmente <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.SchemaName*> , <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.TableName*>y) se almacena normalmente fuera del control de código fuente (por ejemplo, lo almacena el [Administrador secreto](xref:security/app-secrets) o en *appSettings. JSON*/*appSettings. { ENVIRONMENT}. JSON* files). La cadena de conexión puede contener credenciales que deben mantenerse fuera de los sistemas de control de código fuente.

### <a name="distributed-redis-cache"></a>Redis Cache distribuido

[Redis](https://redis.io/) es un almacén de datos en memoria de código abierto, que a menudo se usa como caché distribuida. Puede usar Redis localmente, y puede configurar un [Azure Redis cache](https://azure.microsoft.com/services/cache/) para una aplicación de ASP.net Core hospedada en Azure.

::: moniker range=">= aspnetcore-3.0"

Una aplicación configura la implementación de la memoria caché mediante <xref:Microsoft.Extensions.Caching.StackExchangeRedis.RedisCache> una instancia<xref:Microsoft.Extensions.DependencyInjection.StackExchangeRedisCacheServiceCollectionExtensions.AddStackExchangeRedisCache*>de () en un entorno que no `Startup.ConfigureServices`es de desarrollo en:

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_AddStackExchangeRedisCache)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

Una aplicación configura la implementación de la memoria caché mediante <xref:Microsoft.Extensions.Caching.StackExchangeRedis.RedisCache> una instancia<xref:Microsoft.Extensions.DependencyInjection.StackExchangeRedisCacheServiceCollectionExtensions.AddStackExchangeRedisCache*>de () en un entorno que no `Startup.ConfigureServices`es de desarrollo en:

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddStackExchangeRedisCache)]

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Una aplicación configura la implementación de la memoria caché mediante <xref:Microsoft.Extensions.Caching.Redis.RedisCache> una instancia<xref:Microsoft.Extensions.DependencyInjection.RedisCacheServiceCollectionExtensions.AddDistributedRedisCache*>de ():

```csharp
services.AddDistributedRedisCache(options =>
{
    options.Configuration = "localhost";
    options.InstanceName = "SampleInstance";
});
```

::: moniker-end

Para instalar Redis en el equipo local:

* Instale el [paquete chocolatey Redis](https://chocolatey.org/packages/redis-64/).
* Ejecute `redis-server` desde un símbolo del sistema.

## <a name="use-the-distributed-cache"></a>Usar la memoria caché distribuida

Para usar la <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaz, solicite una instancia de <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> desde cualquier constructor de la aplicación. La instancia se proporciona mediante la [inserción de dependencias (di)](xref:fundamentals/dependency-injection).

::: moniker range=">= aspnetcore-3.0"

Cuando se inicia la aplicación de <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> ejemplo, se inserta `Startup.Configure`en. La hora actual se almacena en la <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> memoria caché mediante (para obtener [más información, consulte host genérico: IHostApplicationLifetime](xref:fundamentals/host/generic-host#ihostapplicationlifetime)):

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_Configure&highlight=10)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Cuando se inicia la aplicación de <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> ejemplo, se inserta `Startup.Configure`en. La hora actual se almacena en la <xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime> memoria caché mediante (para obtener [más información, vea host de Web: Interfaz](xref:fundamentals/host/web-host#iapplicationlifetime-interface)IApplicationLifetime):

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_Configure&highlight=10)]

::: moniker-end

La aplicación de ejemplo inyecta <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> en el `IndexModel` para su uso en la página de índice.

Cada vez que se carga la página de índice, se comprueba la hora almacenada en caché `OnGetAsync`en. Si la hora almacenada en caché no ha expirado, se muestra la hora. Si han transcurrido 20 segundos desde la última vez que se tuvo acceso a la hora almacenada en caché (la última vez que se cargó esta página), la página muestra la *fecha de expiración de la memoria caché*.

Actualice inmediatamente la hora almacenada en caché a la hora actual; para ello, seleccione el botón **restablecer hora de almacenamiento en caché** . El botón desencadena el método `OnPostResetCachedTime` de control.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Pages/Index.cshtml.cs?name=snippet_IndexModel&highlight=7,14-20,25-29)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Pages/Index.cshtml.cs?name=snippet_IndexModel&highlight=7,14-20,25-29)]

::: moniker-end

> [!NOTE]
> No es necesario usar un singleton o una duración de ámbito para <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> las instancias (al menos para las implementaciones integradas).
>
> También puede crear una <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> instancia siempre que necesite una en lugar de usar di, pero la creación de una instancia de en el código puede hacer que el código sea más difícil de probar e infringe el principio de [dependencias explícitas](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).

## <a name="recommendations"></a>Recomendaciones

A la hora de decidir <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> qué implementación de es la más adecuada para su aplicación, tenga en cuenta lo siguiente:

* Infraestructura existente
* Requisitos de rendimiento
* Coste
* Experiencia del equipo

Las soluciones de almacenamiento en caché suelen basarse en el almacenamiento en memoria para proporcionar una recuperación rápida de los datos almacenados en memoria caché, pero la memoria es un recurso limitado y se amplía de forma costosa. Almacene solo los datos usados habitualmente en una memoria caché.

Por lo general, una caché en Redis proporciona un mayor rendimiento y una menor latencia que una memoria caché de SQL Server. Sin embargo, normalmente es necesario realizar pruebas comparativas para determinar las características de rendimiento de las estrategias de almacenamiento en caché.

Cuando SQL Server se usa como un almacén de respaldo de caché distribuida, el uso de la misma base de datos para la memoria caché y el almacenamiento y la recuperación de datos normales de la aplicación puede afectar negativamente al rendimiento de ambos. Se recomienda usar una instancia de SQL Server dedicada para el almacén de respaldo de caché distribuida.

## <a name="additional-resources"></a>Recursos adicionales

* [Redis Cache en Azure](/azure/azure-cache-for-redis/)
* [SQL Database en Azure](/azure/sql-database/)
* [ASP.net Core proveedor de IDistributedCache para NCache en granjas de servidores web](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache en github](https://github.com/Alachisoft/NCache))
* <xref:performance/caching/memory>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
* <xref:host-and-deploy/web-farm>
