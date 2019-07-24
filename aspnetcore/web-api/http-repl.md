---
title: Prueba de las API web HTTP REPL
author: scottaddie
description: Obtenga información sobre cómo usar la herramienta global HTTP REPL de .NET Core para examinar y probar una API web de ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 07/12/2019
uid: web-api/http-repl
ms.openlocfilehash: 1774382305cc3d479291700390807d277a24bfa7
ms.sourcegitcommit: b40613c603d6f0cc71f3232c16df61550907f550
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/18/2019
ms.locfileid: "68308341"
---
# <a name="test-web-apis-with-the-http-repl"></a>Prueba de las API web HTTP REPL

Por [Scott Addie](https://twitter.com/Scott_Addie)

El bucle HTTP read-eval-print (REPL):

* Es una herramienta de línea de comandos ligera y multiplataforma con la misma compatibilidad que .NET Core.
* Se usa para realizar solicitudes HTTP con el fin de probar API web de ASP.NET Core y ver los resultados.

Se admiten los siguientes [verbos HTTP](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods):

* [DELETE](#test-http-delete-requests)
* [GET](#test-http-get-requests)
* [HEAD](#test-http-head-requests)
* [OPTIONS](#test-http-options-requests)
* [PATCH](#test-http-patch-requests)
* [POST](#test-http-post-requests)
* [PUT](#test-http-put-requests)

Para continuar, [vea o descargue la API web de muestra de ASP.NET Core ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/http-repl/samples) ([cómo descargar](xref:index#how-to-download-a-sample)).

## <a name="prerequisites"></a>Requisitos previos

* [!INCLUDE [2.1-SDK](~/includes/2.1-SDK.md)]

## <a name="installation"></a>Instalación

Ejecute el siguiente comando para instalar HTTP REPL:

```console
dotnet tool install -g dotnet-httprepl
    --version 2.2.0-*
    --add-source https://dotnet.myget.org/F/dotnet-core/api/v3/index.json
```

Se instala una [herramienta global .NET Core](/dotnet/core/tools/global-tools#install-a-global-tool) desde el paquete NuGet [dotnet-httprepl](https://dotnet.myget.org/feed/dotnet-core/package/nuget/dotnet-httprepl
) hospedado en MyGet.

## <a name="usage"></a>Uso

Tras la correcta instalación de la herramienta, ejecute el siguiente comando para iniciar HTTP REPL:

```console
dotnet httprepl
```

Para ver los comandos de HTTP REPL disponibles, ejecute uno de los siguientes comandos:

```console
dotnet httprepl -h
```

```console
dotnet httprepl --help
```

Se muestra el siguiente resultado:

```console
Usage: dotnet httprepl [<BASE_ADDRESS>] [options]

Arguments:
  <BASE_ADDRESS> - The initial base address for the REPL.

Options:
  --help - Show help information.

Once the REPL starts, these commands are valid:

HTTP Commands:
Use these commands to execute requests against your application.

GET            Issues a GET request.
POST           Issues a POST request.
PUT            Issues a PUT request.
DELETE         Issues a DELETE request.
PATCH          Issues a PATCH request.
HEAD           Issues a HEAD request.
OPTIONS        Issues an OPTIONS request.

set header     Sets or clears a header for all requests. e.g. `set header content-type application/json`


Navigation Commands:
The REPL allows you to navigate your URL space and focus on specific APIs that you are working on.

set base       Set the base URI. e.g. `set base http://locahost:5000`
set swagger    Set the URI, relative to your base if set, of the Swagger document for this API. e.g. `set swagger /swagger/v1/swagger.json`
ls             Show all endpoints for the current path.
cd             Append the given directory to the currently selected path, or move up a path when using `cd ..`.

Shell Commands:
Use these commands to interact with the REPL shell.

clear          Removes all text from the shell.
echo [on/off]  Turns request echoing on or off, show the request that was made when using request commands.
exit           Exit the shell.

REPL Customization Commands:
Use these commands to customize the REPL behavior.

pref [get/set] Allows viewing or changing preferences, e.g. 'pref set editor.command.default 'C:\Program Files\Microsoft VS Code\Code.exe'`
run            Runs the script at the given path. A script is a set of commands that can be typed with one command per line.
ui             Displays the Swagger UI page, if available, in the default browser.

Use help <COMMAND> to learn more details about individual commands. e.g. `help get`
```

HTTP REPL ofrece la finalización del comando. Al presionar la tecla <kbd>Tabulador</kbd>, se procesa una iteración en la lista de comandos que completa los caracteres o el punto de conexión de la API que ha escrito. En las secciones siguientes se describen los comandos disponibles de la CLI.

## <a name="connect-to-the-web-api"></a>Conexión a la API web

Conéctese a la API web ejecutando el comando siguiente:

```console
dotnet httprepl <BASE URI>
```

`<BASE URI>` es el URI base de la API web. Por ejemplo:

```console
dotnet httprepl https://localhost:5001
```

También puede ejecutar el comando siguiente en cualquier momento mientras se ejecuta HTTP REPL:

```console
set base <BASE URI>
```

Por ejemplo:

```console
(Disconnected)~ set base https://localhost:5001
```

## <a name="point-to-the-swagger-document-for-the-web-api"></a>Establecimiento como destino del documento de Swagger de la API web

Para inspeccionar correctamente la API web, establezca el URI relativo en el documento de Swagger para la API web. Ejecute el siguiente comando:

```console
set swagger <RELATIVE URI>
```

Por ejemplo:

```console
https://localhost:5001/~ set swagger /swagger/v1/swagger.json
```

## <a name="navigate-the-web-api"></a>Navegación de la API web

### <a name="view-available-endpoints"></a>Visualización de los puntos de conexión disponibles

Para enumerar los diferentes puntos de conexión (controladores) en la ruta de acceso actual de la dirección de la API web, ejecute el comando `ls` o `dir`:

```console
https://localhot:5001/~ ls
```

Se muestra el siguiente formato de salida:

```console
.        []
Fruits   [get|post]
People   [get|post]

https://localhost:5001/~
```

La salida anterior indica que hay dos controladores disponibles: `Fruits` y `People`. Ambos controladores admiten operaciones HTTP GET y POST sin parámetros.

Al navegar a un controlador específico, se revela más información. Por ejemplo, la salida del comando siguiente muestra que el controlador `Fruits` también admite las operaciones HTTP GET, PUT y DELETE. Cada una de estas operaciones esperan un parámetro `id` en la ruta:

```console
https://localhost:5001/fruits~ ls
.      [get|post]
..     []
{id}   [get|put|delete]

https://localhost:5001/fruits~
```

También puede ejecutar el comando `ui` para abrir la página de la interfaz de usuario de Swagger de la API web en un explorador. Por ejemplo:

```console
https://localhost:5001/~ ui
```

### <a name="navigate-to-an-endpoint"></a>Navegación a un punto de conexión

Para navegar a un punto de conexión distinto en la API web, ejecute el comando `cd`:

```console
https://localhost:5001/~ cd people
```

La ruta de acceso que sigue al comando `cd` no distingue mayúsculas de minúsculas. Se muestra el siguiente formato de salida:

```console
/people    [get|post]

https://localhost:5001/people~
```

## <a name="customize-the-http-repl"></a>Personalización de HTTP REPL

Los [colores](#set-color-preferences) predeterminados de HTTP REPL se pueden personalizar. Además, se puede definir un [editor de texto predeterminado](#set-the-default-text-editor). Las preferencias de HTTP REPL se conservan tanto en la sesión actual como en futuras sesiones. Una vez modificadas, se almacenan en el archivo siguiente:

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

*%HOME%/.httpreplprefs*

# <a name="macostabmacos"></a>[macOS](#tab/macos)

*%HOME%/.httpreplprefs*

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

*%USERPROFILE%\\.httpreplprefs*

---

El archivo *.httpreplprefs* se carga al inicio; los cambios no se supervisan durante el tiempo en ejecución. Las modificaciones manuales que se hagan en el archivo solo se aplicarán después de reiniciar la herramienta.

### <a name="view-the-settings"></a>Visualización de la configuración

Para ver la configuración disponible, ejecute el comando `pref get`. Por ejemplo:

```console
https://localhost:5001/~ pref get
```

El comando anterior muestra los pares clave-valor disponibles:

```console
colors.json=Green
colors.json.arrayBrace=BoldCyan
colors.json.comma=BoldYellow
colors.json.name=BoldMagenta
colors.json.nameSeparator=BoldWhite
colors.json.objectBrace=Cyan
colors.protocol=BoldGreen
colors.status=BoldYellow
```

### <a name="set-color-preferences"></a>Establecimiento de las preferencias de color

Actualmente solo se permite colorear la respuesta para JSON. Para personalizar el color predeterminado de la herramienta HTTP REPL, busque la clave correspondiente al color que se va a cambiar. Para obtener instrucciones sobre cómo buscar las claves, consulte la sección [Visualización de la configuración](#view-the-settings). Por ejemplo, cambie el valor de clave `colors.json` de `Green` a `White`, tal como se indica a continuación:

```console
https://localhost:5001/people~ pref set colors.json White
```

Solo se pueden usar los [colores permitidos](https://github.com/aspnet/HttpRepl/blob/01d5c3c3373e98fe566ff5ef8a17c571de880293/src/Microsoft.Repl/ConsoleHandling/AllowedColors.cs). Las solicitudes HTTP posteriores muestran la salida con el nuevo color.

Si no se establecen claves de color específicas, se consideran las más genéricas. Para mostrar este comportamiento de reserva, veamos el ejemplo siguiente:

* Si `colors.json.name` no tiene ningún valor, se usa `colors.json.string`.
* Si `colors.json.string` no tiene ningún valor, se usa `colors.json.literal`.
* Si `colors.json.literal` no tiene ningún valor, se usa `colors.json`. 
* Si `colors.json` no tiene ningún valor, se usa el color de texto predeterminado del shell del comando (`AllowedColors.None`).

### <a name="set-indentation-size"></a>Establecimiento del tamaño de sangría

Actualmente, la personalización del tamaño de sangría de respuesta solo se admite para JSON. El tamaño predeterminado es de dos espacios. Por ejemplo:

```json
[
  {
    "id": 1,
    "name": "Apple"
  },
  {
    "id": 2,
    "name": "Orange"
  },
  {
    "id": 3,
    "name": "Strawberry"
  }
]
```

Para cambiar el tamaño predeterminado, establezca la clave `formatting.json.indentSize`. Por ejemplo, para usar siempre cuatro espacios:

```console
pref set formatting.json.indentSize 4
```

Las respuestas posteriores respetan el valor de cuatro espacios:

```json
[
    {
        "id": 1,
        "name": "Apple"
    },
    {
        "id": 2,
        "name": "Orange"
    },
    {
        "id": 3,
        "name": "Strawberry"
    }
]
```

### <a name="set-indentation-size"></a>Establecimiento del tamaño de sangría

Actualmente, la personalización del tamaño de sangría de respuesta solo se admite para JSON. El tamaño predeterminado es de dos espacios. Por ejemplo:

```json
[
  {
    "id": 1,
    "name": "Apple"
  },
  {
    "id": 2,
    "name": "Orange"
  },
  {
    "id": 3,
    "name": "Strawberry"
  }
]
```

Para cambiar el tamaño predeterminado, establezca la clave `formatting.json.indentSize`. Por ejemplo, para usar siempre cuatro espacios:

```console
pref set formatting.json.indentSize 4
```

Las respuestas posteriores respetan el valor de cuatro espacios:

```json
[
    {
        "id": 1,
        "name": "Apple"
    },
    {
        "id": 2,
        "name": "Orange"
    },
    {
        "id": 3,
        "name": "Strawberry"
    }
]
```

### <a name="set-the-default-text-editor"></a>Establecimiento del editor de texto predeterminado

De manera predeterminada, HTTP REPL no tiene ningún editor de texto configurado para su uso. Para probar los métodos de la API web que requieren un cuerpo de la solicitud HTTP, se debe establecer un editor de texto predeterminado. La herramienta HTTP REPL inicia el editor de texto configurado con el único fin de redactar el cuerpo de la solicitud. Ejecute el comando siguiente para establecer el editor de texto preferido como predeterminado:

```console
pref set editor.command.default "<EXECUTABLE>"
```

En el comando anterior, `<EXECUTABLE>` es la ruta de acceso completa al archivo ejecutable del editor de texto. Por ejemplo, ejecute el comando siguiente para establecer Visual Studio Code como editor de texto predeterminado:

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

```console
pref set editor.command.default "/usr/bin/code"
```

# <a name="macostabmacos"></a>[macOS](#tab/macos)

```console
pref set editor.command.default "/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code"
```

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

```console
pref set editor.command.default "C:\Program Files\Microsoft VS Code\Code.exe"
```

---

Para iniciar el editor de texto predeterminado con argumentos específicos de la CLI, establezca la clave `editor.command.default.arguments`. Por ejemplo, supongamos que Visual Studio Code es el editor de texto predeterminado y que siempre quiere que HTTP REPL abra Visual Studio Code en una nueva sesión con las extensiones deshabilitadas. Ejecute el siguiente comando:

```console
pref set editor.command.default.arguments "--disable-extensions --new-window"
```

## <a name="test-http-get-requests"></a>Prueba de solicitudes HTTP GET

### <a name="synopsis"></a>Sinopsis

```console
get <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumentos

`PARAMETER`

Se trata del parámetro de ruta, si existe, que espera el método de acción del controlador asociado.

### <a name="options"></a>Opciones

Las siguientes opciones están disponibles para el comando `get`:

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

### <a name="example"></a>Ejemplo

Para emitir una solicitud HTTP GET realice lo siguiente:

1. Ejecute el comando `get` en un punto de conexión que lo admita:

    ```console
    https://localhost:5001/people~ get
    ```

    El comando anterior muestra el siguiente formato de salida:

    ```console
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 21 Jun 2019 03:38:45 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "name": "Scott Hunter"
      },
      {
        "id": 2,
        "name": "Scott Hanselman"
      },
      {
        "id": 3,
        "name": "Scott Guthrie"
      }
    ]


    https://localhost:5001/people~
    ```

1. Recupere un registro específico pasando un parámetro al comando `get`:

    ```console
    https://localhost:5001/people~ get 2
    ```

    El comando anterior muestra el siguiente formato de salida:

    ```console
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 21 Jun 2019 06:17:57 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 2,
        "name": "Scott Hanselman"
      }
    ]


    https://localhost:5001/people~
    ```

## <a name="test-http-post-requests"></a>Prueba de solicitudes HTTP POST

### <a name="synopsis"></a>Sinopsis

```console
post <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumentos

`PARAMETER`

Se trata del parámetro de ruta, si existe, que espera el método de acción del controlador asociado.

### <a name="options"></a>Opciones

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

### <a name="example"></a>Ejemplo

Para emitir una solicitud HTTP POST, realice lo siguiente:

1. Ejecute el comando `post` en un punto de conexión que lo admita:

    ```console
    https://localhost:5001/people~ post -h Content-Type=application/json
    ```

    En el comando anterior, el encabezado de solicitud HTTP `Content-Type` se establece para indicar un tipo de medios de cuerpo de la solicitud de JSON. El editor de texto predeterminado abre un archivo *.tmp* con una plantilla JSON que representa el cuerpo de la solicitud HTTP. Por ejemplo:

    ```json
    {
      "id": 0,
      "name": ""
    }
    ```

    > [!TIP]
    > Para establecer el editor de texto predeterminado, consulte la sección [Establecimiento del editor de texto predeterminado](#set-the-default-text-editor).

1. Modifique la plantilla JSON para cumplir los requisitos de validación de modelos:

  ```json
  {
    "id": 0,
    "name": "Scott Addie"
  }
  ```

1. Guarde el archivo *.tmp* y cierre el editor de texto. En el shell del comando aparecerá la salida siguiente:

    ```console
    HTTP/1.1 201 Created
    Content-Type: application/json; charset=utf-8
    Date: Thu, 27 Jun 2019 21:24:18 GMT
    Location: https://localhost:5001/people/4
    Server: Kestrel
    Transfer-Encoding: chunked

    {
      "id": 4,
      "name": "Scott Addie"
    }


    https://localhost:5001/people~
    ```

## <a name="test-http-put-requests"></a>Prueba de solicitudes HTTP PUT

### <a name="synopsis"></a>Sinopsis

```console
put <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumentos

`PARAMETER`

Se trata del parámetro de ruta, si existe, que espera el método de acción del controlador asociado.

### <a name="options"></a>Opciones

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

### <a name="example"></a>Ejemplo

Para emitir una solicitud HTTP PUT, realice lo siguiente:

1. *Opcional*: Ejecute el comando `get` para ver los datos antes de modificarlos:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:07:32 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Orange"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]

1. Run the `put` command on an endpoint that supports it:

    ```console
    https://localhost:5001/fruits~ put 2 -h Content-Type=application/json
    ```

    En el comando anterior, el encabezado de solicitud HTTP `Content-Type` se establece para indicar un tipo de medios de cuerpo de la solicitud de JSON. El editor de texto predeterminado abre un archivo *.tmp* con una plantilla JSON que representa el cuerpo de la solicitud HTTP. Por ejemplo:

    ```json
    {
      "id": 0,
      "name": ""
    }
    ```

    > [!TIP]
    > Para establecer el editor de texto predeterminado, consulte la sección [Establecimiento del editor de texto predeterminado](#set-the-default-text-editor).

1. Modifique la plantilla JSON para cumplir los requisitos de validación de modelos:

    ```json
    {
      "id": 2,
      "name": "Cherry"
    }
    ```

1. Guarde el archivo *.tmp* y cierre el editor de texto. En el shell del comando aparecerá la salida siguiente:

    ```console
    [main 2019-06-28T17:27:01.805Z] update#setState idle
    HTTP/1.1 204 No Content
    Date: Fri, 28 Jun 2019 17:28:21 GMT
    Server: Kestrel
    ```

1. *Opcional*: Emita un comando `get` para ver las modificaciones. Por ejemplo, si ha escrito "Cherry" en el editor de texto, un elemento `get` devuelve lo siguiente:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:08:20 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Cherry"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]


    https://localhost:5001/fruits~
    ```

## <a name="test-http-delete-requests"></a>Prueba de solicitudes HTTP DELETE

### <a name="synopsis"></a>Sinopsis

```console
delete <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumentos

`PARAMETER`

Se trata del parámetro de ruta, si existe, que espera el método de acción del controlador asociado.

### <a name="options"></a>Opciones

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

### <a name="example"></a>Ejemplo

Para emitir una solicitud HTTP DELETE, realice lo siguiente:

1. *Opcional*: Ejecute el comando `get` para ver los datos antes de modificarlos:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:07:32 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Orange"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]

1. Run the `delete` command on an endpoint that supports it:

    ```console
    https://localhost:5001/fruits~ delete 2
    ```

    El comando anterior muestra el siguiente formato de salida:

    ```console
    HTTP/1.1 204 No Content
    Date: Fri, 28 Jun 2019 17:36:42 GMT
    Server: Kestrel
    ```

1. *Opcional*: Emita un comando `get` para ver las modificaciones. En este ejemplo, un elemento `get` devuelve lo siguiente:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:16:30 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]


    https://localhost:5001/fruits~
    ```

## <a name="test-http-patch-requests"></a>Prueba de solicitudes HTTP PATCH

### <a name="synopsis"></a>Sinopsis

```console
patch <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumentos

`PARAMETER`

Se trata del parámetro de ruta, si existe, que espera el método de acción del controlador asociado.

### <a name="options"></a>Opciones

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

## <a name="test-http-head-requests"></a>Prueba de solicitudes HTTP HEAD

### <a name="synopsis"></a>Sinopsis

```console
head <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumentos

`PARAMETER`

Se trata del parámetro de ruta, si existe, que espera el método de acción del controlador asociado.

### <a name="options"></a>Opciones

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

## <a name="test-http-options-requests"></a>Prueba de solicitudes HTTP OPTIONS

### <a name="synopsis"></a>Sinopsis

```console
options <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Argumentos

`PARAMETER`

Se trata del parámetro de ruta, si existe, que espera el método de acción del controlador asociado.

### <a name="options"></a>Opciones

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

## <a name="set-http-request-headers"></a>Establecimiento de encabezados de solicitudes HTTP

Para establecer un encabezado de solicitud HTTP, use uno de los métodos siguientes:

1. Establézcalo insertado con la solicitud HTTP. Por ejemplo:

  ```console
  https://localhost:5001/people~ post -h Content-Type=application/json
  ```

  Con el método anterior, cada encabezado de solicitud HTTP distinto requiere su propia opción `-h`.

1. Establézcalo antes de enviar la solicitud HTTP. Por ejemplo:

  ```console
  https://localhost:5001/people~ set header Content-Type application/json
  ```

  Al establecer el encabezado antes de enviar una solicitud, este permanece establecido mientras dure la sesión de shell de comandos. Para borrar el encabezado, proporcione un valor vacío. Por ejemplo:

  ```console
  https://localhost:5001/people~ set header Content-Type
  ```

## <a name="toggle-http-request-display"></a>Alternancia de la pantalla de solicitud HTTP

De manera predeterminada, se suprime la pantalla de solicitud HTTP que se está enviando. Es posible cambiar la configuración correspondiente mientras dure la sesión de shell de comandos.

### <a name="enable-request-display"></a>Habilitación de la pantalla de solicitudes

Vea la solicitud HTTP que se envía mediante la ejecución del comando `echo on`. Por ejemplo:

```console
https://localhost:5001/people~ echo on
Request echoing is on
```

Las solicitudes HTTP posteriores de la sesión actual muestran los encabezados de la solicitud. Por ejemplo:

```console
https://localhost:5001/people~ post

[main 2019-06-28T18:50:11.930Z] update#setState idle
Request to https://localhost:5001...

POST /people HTTP/1.1
Content-Length: 41
Content-Type: application/json
User-Agent: HTTP-REPL

{
  "id": 0,
  "name": "Scott Addie"
}

Response from https://localhost:5001...

HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Date: Fri, 28 Jun 2019 18:50:21 GMT
Location: https://localhost:5001/people/4
Server: Kestrel
Transfer-Encoding: chunked

{
  "id": 4,
  "name": "Scott Addie"
}


https://localhost:5001/people~
```

### <a name="disable-request-display"></a>Deshabilitación de la pantalla de solicitudes

Suprima la pantalla de solicitud HTTP que se envía mediante la ejecución del comando `echo off`. Por ejemplo:

```console
https://localhost:5001/people~ echo off
Request echoing is off
```

## <a name="run-a-script"></a>Ejecución de un script

Si ejecuta con frecuencia el mismo conjunto de comandos de HTTP REPL, considere la posibilidad de almacenarlos en un archivo de texto. Los comandos del archivo toman el mismo formulario que los ejecutados manualmente en la línea de comandos. Los comandos se pueden ejecutar en un modo por lotes mediante el comando `run`. Por ejemplo:

1. Cree un archivo de texto que contenga un conjunto de comandos delimitados por línea nueva. Para ilustrarlo, considere la posibilidad de usar un archivo *people-script.txt* que contenga los comandos siguientes:

    ```text
    set base https://localhost:5001
    ls
    cd People
    ls
    get 1
    ```

1. Ejecute el comando `run`, pasando la ruta de acceso del archivo de texto. Por ejemplo:

    ```console
    https://localhost:5001/~ run C:\http-repl-scripts\people-script.txt
    ```

    Se mostrará lo siguiente:

    ```console
    https://localhost:5001/~ set base https://localhost:5001
    Using swagger metadata from https://localhost:5001/swagger/v1/swagger.json
    
    https://localhost:5001/~ ls
    .        []
    Fruits   [get|post]
    People   [get|post]
    
    https://localhost:5001/~ cd People
    /People    [get|post]
    
    https://localhost:5001/People~ ls
    .      [get|post]
    ..     []
    {id}   [get|put|delete]
    
    https://localhost:5001/People~ get 1
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 12 Jul 2019 19:20:10 GMT
    Server: Kestrel
    Transfer-Encoding: chunked
    
    {
      "id": 1,
      "name": "Scott Hunter"
    }
    
    
    https://localhost:5001/People~
    ```

## <a name="clear-the-output"></a>Borrado de la salida

Para quitar todas las salidas escritas en el shell de comandos mediante la herramienta REPL HTTP, ejecute el comando `clear` o `cls`. Para ilustrarlo, imagine que el shell de comandos contiene la salida siguiente:

```console
dotnet httprepl https://localhost:5001
(Disconnected)~ set base "https://localhost:5001"
Using swagger metadata from https://localhost:5001/swagger/v1/swagger.json

https://localhost:5001/~ ls
.        []
Fruits   [get|post]
People   [get|post]

https://localhost:5001/~
```

Ejecute el comando siguiente para borrar la salida:

```console
https://localhost:5001/~ clear
```

Después de ejecutar el comando anterior, el shell de comandos solo contiene la salida siguiente:

```console
https://localhost:5001/~
```

## <a name="additional-resources"></a>Recursos adicionales

* [Solicitudes de API REST](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods)
* [Repositorio GitHub de HTTP REPL](https://github.com/aspnet/HttpRepl)