---
title: Cliente .NET de Signalr ASP.NET Core
author: bradygaster
description: Información sobre el cliente .NET de Signalr ASP.NET Core
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 09/13/2019
uid: signalr/dotnet-client
ms.openlocfilehash: 4419799ef11469413f813843a9d02ac0223d30c6
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/18/2019
ms.locfileid: "71081292"
---
# <a name="aspnet-core-signalr-net-client"></a>Cliente .NET de Signalr ASP.NET Core

La biblioteca de cliente de ASP.NET Core Signalr .NET le permite comunicarse con los concentradores de Signalr desde aplicaciones .NET.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/dotnet-client/sample) ([cómo descargarlo](xref:index#how-to-download-a-sample))

El ejemplo de código de este artículo es una aplicación de WPF que usa el cliente .NET de ASP.NET Core Signalr.

## <a name="install-the-signalr-net-client-package"></a>Instalar el paquete de cliente de Signalr .NET

El paquete [Microsoft. AspNetCore. signalr. Client](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) es necesario para que los clientes de .net se conecten a los concentradores de signalr.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Para instalar la biblioteca de cliente de, ejecute el siguiente comando en la ventana de la **consola del administrador de paquetes** :

```powershell
Install-Package Microsoft.AspNetCore.SignalR.Client
```

# <a name="net-core-clitabnetcore-cli"></a>[CLI de .NET Core](#tab/netcore-cli)

Para instalar la biblioteca de cliente de, ejecute el siguiente comando en un shell de comandos:

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

---

## <a name="connect-to-a-hub"></a>Conexión a un concentrador

Para establecer una conexión, cree un `HubConnectionBuilder` y llame `Build`a. La dirección URL del concentrador, el protocolo, el tipo de transporte, el nivel de registro, los encabezados y otras opciones se pueden configurar al compilar una conexión. Configure las opciones necesarias insertando cualquiera de los `HubConnectionBuilder` métodos en `Build`. Inicie la conexión con `StartAsync`.

[!code-csharp[Build hub connection](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_MainWindowClass&highlight=15-17,39)]

## <a name="handle-lost-connection"></a>Controlar la conexión perdida

::: moniker range=">= aspnetcore-3.0"

### <a name="automatically-reconnect"></a>Volver a conectar automáticamente

Se puede configurar para que se vuelva a conectar automáticamente `WithAutomaticReconnect` mediante el método <xref:Microsoft.AspNetCore.SignalR.Client.HubConnectionBuilder>en. <xref:Microsoft.AspNetCore.SignalR.Client.HubConnection> No se volverá a conectar automáticamente de forma predeterminada.

```csharp
HubConnection connection= new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chatHub"))
    .WithAutomaticReconnect()
    .Build();
```

Sin parámetros, `WithAutomaticReconnect()` configura el cliente para que espere 0, 2, 10 y 30 segundos, respectivamente, antes de intentar cada intento de reconexión, deteniéndose después de cuatro intentos erróneos.

Antes de iniciar cualquier intento de reconexión `HubConnection` , cambiará `HubConnectionState.Reconnecting` al estado y desencadenará `Reconnecting` el evento.  Esto proporciona una oportunidad para advertir a los usuarios de que se ha perdido la conexión y deshabilitar los elementos de la interfaz de usuario. Las aplicaciones no interactivas pueden iniciar la puesta en cola o la eliminación de mensajes.

```csharp
connection.Reconnecting += error =>
{
    Debug.Assert(connection.State == HubConnectionState.Reconnecting);

    // Notify users the connection was lost and the client is reconnecting.
    // Start queuing or dropping messages.

    return Task.CompletedTask;
};
```

Si el cliente `HubConnection` se vuelve `Connected` a conectar correctamente dentro de los cuatro primeros intentos, volverá a pasar al estado y desencadenará `Reconnected` el evento. Esto proporciona una oportunidad para informar a los usuarios de que la conexión se ha restablecido y quitado de la cola todos los mensajes en cola.

Dado que la conexión es completamente nueva en el servidor, se `ConnectionId` proporcionará un nuevo a `Reconnected` los controladores de eventos.

> [!WARNING]
> El `Reconnected` parámetro del controlador `connectionId` de eventos será NULL si se `HubConnection` configuró para [omitir la negociación](xref:signalr/configuration#configure-client-options).

```csharp
connection.Reconnected += connectionId =>
{
    Debug.Assert(connection.State == HubConnectionState.Connected);

    // Notify users the connection was reestablished.
    // Start dequeuing messages queued while reconnecting if any.

    return Task.CompletedTask;
};
```

`WithAutomaticReconnect()`no configurará el para reintentar errores de inicio inicial, por lo que los `HubConnection` errores de inicio deben controlarse manualmente:

```csharp
public static async Task<bool> ConnectWithRetryAsync(HubConnection connection, CancellationToken token)
{
    // Keep trying to until we can start or the token is canceled.
    while (true)
    {
        try
        {
            await connection.StartAsync(token);
            Debug.Assert(connection.State == HubConnectionState.Connected);
            return true;
        }
        catch when (token.IsCancellationRequested)
        {
            return false;
        }
        catch
        {
            // Failed to connect, trying again in 5000 ms.
            Debug.Assert(connection.State == HubConnectionState.Disconnected);
            await Task.Delay(5000);
        }
    }
}
```

Si el cliente no `HubConnection` se vuelve `Disconnected` a conectar correctamente en los cuatro primeros intentos, cambiará al estado y desencadenará <xref:Microsoft.AspNetCore.SignalR.Client.HubConnection.Closed> el evento. Esto proporciona una oportunidad para intentar reiniciar la conexión manualmente o informar a los usuarios de que la conexión se ha perdido permanentemente.

```csharp
connection.Closed += error =>
{
    Debug.Assert(connection.State == HubConnectionState.Disconnected);

    // Notify users the connection has been closed or manually try to restart the connection.

    return Task.CompletedTask;
};
```

Con el fin de configurar un número personalizado de intentos de reconexión antes de desconectar o cambiar el tiempo de reconexión, `WithAutomaticReconnect` acepta una matriz de números que representan el retraso en milisegundos que se debe esperar antes de iniciar cada intento de reconexión.

```csharp
HubConnection connection= new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chatHub"))
    .WithAutomaticReconnect(new[] { TimeSpan.Zero, TimeSpan.Zero, TimeSpan.FromSeconds(10) })
    .Build();

    // .WithAutomaticReconnect(new[] { TimeSpan.Zero, TimeSpan.FromSeconds(2), TimeSpan.FromSeconds(10), TimeSpan.FromSeconds(30) }) yields the default behavior.
```

En el ejemplo anterior se configura para `HubConnection` iniciar el intento de reconexión inmediatamente después de la pérdida de la conexión. Esto también se aplica a la configuración predeterminada.

Si se produce un error en el primer intento de reconexión, el segundo intento de reconexión también se iniciará inmediatamente en lugar de esperar 2 segundos como lo haría en la configuración predeterminada.

Si se produce un error en el segundo intento de reconexión, el tercer intento de reconexión se iniciará en 10 segundos, lo que volverá a ser como la configuración predeterminada.

Después, el comportamiento personalizado difiere de nuevo del comportamiento predeterminado al detenerse después del tercer error de intento de reconexión. En la configuración predeterminada, se producirá un intento de reconexión más en otros 30 segundos.

Si desea tener un mayor control sobre la temporización y el número de intentos de reconexión `WithAutomaticReconnect` automática, acepta un objeto `IRetryPolicy` que implementa la interfaz, que tiene un `NextRetryDelay`único método denominado.

`NextRetryDelay`toma un único argumento con el tipo `RetryContext`. `PreviousRetryCount` `ElapsedTime` Tienetres`RetryReason` propiedades: `TimeSpan` , y que son ,y,respectivamente.`Exception` `long` `RetryContext` Antes del primer intento de reconexión, `PreviousRetryCount` y `ElapsedTime` serán cero `RetryReason` y será la excepción que provocó la pérdida de la conexión. Después de cada intento de reintento erróneo, `PreviousRetryCount` se incrementará en uno, `ElapsedTime` se actualizará para reflejar la cantidad de tiempo empleado en la reconexión `RetryReason` hasta el momento, y será la excepción que provocó el último error en el intento de reconexión.

`NextRetryDelay`debe devolver un valor TimeSpan que representa el tiempo de espera antes del siguiente intento de reconexión `null` o `HubConnection` si debe dejar de volver a conectarse.

```csharp
public class RandomRetryPolicy : IRetryPolicy
{
    private readonly Random _random = new Random();

    public TimeSpan? NextRetryDelay(RetryContext retryContext)
    {
        // If we've been reconnecting for less than 60 seconds so far,
        // wait between 0 and 10 seconds before the next reconnect attempt.
        if (retryContext.ElapsedTime < TimeSpan.FromSeconds(60))
        {
            return TimeSpan.FromSeconds(_random.Next() * 10);
        }
        else
        {
            // If we've been reconnecting for more than 60 seconds so far, stop reconnecting.
            return null;
        }
    }
}
```

```csharp
HubConnection connection = new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chatHub"))
    .WithAutomaticReconnect(new RandomRetryPolicy())
    .Build();
```

Como alternativa, puede escribir código que volverá a conectar el cliente manualmente, tal como se muestra en [reconexión manual](#manually-reconnect).

::: moniker-end

### <a name="manually-reconnect"></a>Volver a conectar manualmente

::: moniker range="< aspnetcore-3.0"

> [!WARNING]
> Antes de 3,0, el cliente de .NET para Signalr no se vuelve a conectar automáticamente. Debe escribir código que volverá a conectar el cliente manualmente.

::: moniker-end

Utilice el <xref:Microsoft.AspNetCore.SignalR.Client.HubConnection.Closed> evento para responder a una conexión perdida. Por ejemplo, puede que desee automatizar la reconexión.

El `Closed` evento requiere un delegado que `Task`devuelve, que permite que el código asincrónico se ejecute sin `async void`usar. Para satisfacer la firma del delegado en `Closed` un controlador de eventos que se ejecuta `Task.CompletedTask`sincrónicamente, devuelva:

```csharp
connection.Closed += (error) => {
    // Do your close logic.
    return Task.CompletedTask;
};
```

La razón principal de la compatibilidad con Async es para que pueda reiniciar la conexión. Iniciar una conexión es una acción asincrónica.

En un `Closed` controlador que reinicie la conexión, considere la posibilidad de esperar algún retraso aleatorio para evitar la sobrecarga del servidor, como se muestra en el ejemplo siguiente:

[!code-csharp[Use Closed event handler to automate reconnection](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ClosedRestart)]

## <a name="call-hub-methods-from-client"></a>Llamar a métodos de Hub desde el cliente

`InvokeAsync`llama a los métodos en el concentrador. Pase el nombre del método de concentrador y los argumentos definidos en el `InvokeAsync`método de concentrador a. Signalr es asincrónico, por lo que `async` debe `await` usar y al realizar las llamadas.

[!code-csharp[InvokeAsync method](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_InvokeAsync)]

El `InvokeAsync` método devuelve un `Task` que se completa cuando el método de servidor devuelve. El valor devuelto, si existe, se proporciona como resultado de `Task`. Cualquier excepción producida por el método en el servidor produce un error `Task`. Use `await` la sintaxis para esperar a que se complete el método `try...catch` de servidor y la sintaxis para controlar los errores.

El `SendAsync` método devuelve un `Task` que se completa cuando se envía el mensaje al servidor. No se proporciona ningún valor devuelto `Task` , ya que esto no espera hasta que se complete el método del servidor. Las excepciones producidas en el cliente mientras se envía el mensaje producen un `Task`error. Use `await` la `try...catch` sintaxis y para controlar los errores de envío.

> [!NOTE]
> Si usa Azure Signalr Service en modo sin *servidor*, no puede llamar a métodos de concentrador desde un cliente. Para obtener más información, consulte la [documentación del servicio signalr](/azure/azure-signalr/signalr-concept-serverless-development-config).

## <a name="call-client-methods-from-hub"></a>Llamar a métodos de cliente desde el concentrador

Defina los métodos que el concentrador llama mediante `connection.On` después de compilar, pero antes de iniciar la conexión.

[!code-csharp[Define client methods](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ConnectionOn)]

El código anterior en `connection.On` se ejecuta cuando el código del lado servidor lo llama `SendAsync` mediante el método.

[!code-csharp[Call client method](dotnet-client/sample/signalrchat/hubs/chathub.cs?name=snippet_SendMessage)]

## <a name="error-handling-and-logging"></a>Control de errores y registro

Controlar los errores con una instrucción try-catch. Inspeccione `Exception` el objeto para determinar la acción adecuada que se realizará después de que se produzca un error.

[!code-csharp[Logging](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ErrorHandling)]

## <a name="additional-resources"></a>Recursos adicionales

* [Concentradores](xref:signalr/hubs)
* [Cliente de JavaScript](xref:signalr/javascript-client)
* [Publicar en Azure](xref:signalr/publish-to-azure-web-app)
* [Documentación sin servidor del servicio Azure Signalr](/azure/azure-signalr/signalr-concept-serverless-development-config)
