# <a name="aspnet-core-url-rewriting-sample-aspnet-core-2x"></a><span data-ttu-id="756d2-101">Ejemplo de reescritura de la URL de núcleo de ASP.NET (ASP.NET Core 2.x)</span><span class="sxs-lookup"><span data-stu-id="756d2-101">ASP.NET Core URL Rewriting Sample (ASP.NET Core 2.x)</span></span>

<span data-ttu-id="756d2-102">Este ejemplo muestra el uso de ASP.NET Core 2.x Middleware de reescritura de dirección URL.</span><span class="sxs-lookup"><span data-stu-id="756d2-102">This sample illustrates usage of ASP.NET Core 2.x URL Rewriting Middleware.</span></span> <span data-ttu-id="756d2-103">La aplicación muestra el redireccionamiento de la dirección URL y opciones de reescritura de dirección URL.</span><span class="sxs-lookup"><span data-stu-id="756d2-103">The application demonstrates URL redirect and URL rewriting options.</span></span> <span data-ttu-id="756d2-104">Para obtener el ejemplo de 1.x ASP.NET Core, vea [ejemplo de reescritura de URL de ASP.NET Core (ASP.NET Core 1.x)](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/url-rewriting/samples/1.x).</span><span class="sxs-lookup"><span data-stu-id="756d2-104">For the ASP.NET Core 1.x sample, see [ASP.NET Core URL Rewriting Sample (ASP.NET Core 1.x)](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/url-rewriting/samples/1.x).</span></span>

<span data-ttu-id="756d2-105">Al ejecutar el ejemplo, se enviará una respuesta que muestra la dirección URL redirigida o ha vuelto a escribir cuando una de las reglas se aplica a una dirección URL de la solicitud.</span><span class="sxs-lookup"><span data-stu-id="756d2-105">When running the sample, a response will be served that shows the rewritten or redirected URL when one of the rules is applied to a request URL.</span></span>

## <a name="examples-in-this-sample"></a><span data-ttu-id="756d2-106">En los ejemplos de este ejemplo</span><span class="sxs-lookup"><span data-stu-id="756d2-106">Examples in this sample</span></span>

* `AddRedirect("redirect-rule/(.*)", "$1")`
  - <span data-ttu-id="756d2-107">Código de estado correcto: 302 (encontrado)</span><span class="sxs-lookup"><span data-stu-id="756d2-107">Success status code: 302 (Found)</span></span>
  - <span data-ttu-id="756d2-108">Ejemplo (redirección): **/redirect-rule / {capture_group}** a **/redirected/ {capture_group}**</span><span class="sxs-lookup"><span data-stu-id="756d2-108">Example (redirect): **/redirect-rule/{capture_group}** to **/redirected/{capture_group}**</span></span>
* `AddRewrite(@"^rewrite-rule/(\d+)/(\d+)", "rewritten?var1=$1&var2=$2", skipRemainingRules: true)`
  - <span data-ttu-id="756d2-109">Código de estado correcto: 200 (correcto)</span><span class="sxs-lookup"><span data-stu-id="756d2-109">Success status code: 200 (OK)</span></span>
  - <span data-ttu-id="756d2-110">Ejemplo (reescritura): **/rewrite-rule / {capture_group_1} / {capture_group_2}** a **/ reescrito? var1 = {capture_group_1} & var2 = {capture_group_2}**</span><span class="sxs-lookup"><span data-stu-id="756d2-110">Example (rewrite): **/rewrite-rule/{capture_group_1}/{capture_group_2}** to **/rewritten?var1={capture_group_1}&var2={capture_group_2}**</span></span>
* `AddApacheModRewrite(env.ContentRootFileProvider, "ApacheModRewrite.txt")`
  - <span data-ttu-id="756d2-111">Código de estado correcto: 302 (encontrado)</span><span class="sxs-lookup"><span data-stu-id="756d2-111">Success status code: 302 (Found)</span></span>
  - <span data-ttu-id="756d2-112">Ejemplo (redirección): **/apache-mod-rules-redirect / {capture_group}** a **/ redirigido? Id. = {capture_group}**</span><span class="sxs-lookup"><span data-stu-id="756d2-112">Example (redirect): **/apache-mod-rules-redirect/{capture_group}** to **/redirected?id={capture_group}**</span></span>
* `AddIISUrlRewrite(env.ContentRootFileProvider, "IISUrlRewrite.xml")`
  - <span data-ttu-id="756d2-113">Código de estado correcto: 200 (correcto)</span><span class="sxs-lookup"><span data-stu-id="756d2-113">Success status code: 200 (OK)</span></span>
  - <span data-ttu-id="756d2-114">Ejemplo (reescritura): **/iis-rules-rewrite / {capture_group}** a **/ reescrito? Id. = {capture_group}**</span><span class="sxs-lookup"><span data-stu-id="756d2-114">Example (rewrite): **/iis-rules-rewrite/{capture_group}** to **/rewritten?id={capture_group}**</span></span>
* `Add(RedirectXMLRequests)`
  - <span data-ttu-id="756d2-115">Código de estado correcto: 301 (movida permanentemente)</span><span class="sxs-lookup"><span data-stu-id="756d2-115">Success status code: 301 (Moved Permanently)</span></span>
  - <span data-ttu-id="756d2-116">Ejemplo (redirección): **/file.xml** a **/xmlfiles/file.xml**</span><span class="sxs-lookup"><span data-stu-id="756d2-116">Example (redirect): **/file.xml** to **/xmlfiles/file.xml**</span></span>
* `Add(new RedirectPNGRequests(".png", "/png-images")))`<br>`Add(new RedirectPNGRequests(".jpg", "/jpg-images")))`
  - <span data-ttu-id="756d2-117">Código de estado correcto: 301 (movida permanentemente)</span><span class="sxs-lookup"><span data-stu-id="756d2-117">Success status code: 301 (Moved Permanently)</span></span>
  - <span data-ttu-id="756d2-118">Ejemplo (redirección): **/image.png** a **/png-images/image.png**</span><span class="sxs-lookup"><span data-stu-id="756d2-118">Example (redirect): **/image.png** to **/png-images/image.png**</span></span>
  - <span data-ttu-id="756d2-119">Ejemplo (redirección): **/image.jpg** a **/jpg-images/image.jpg**</span><span class="sxs-lookup"><span data-stu-id="756d2-119">Example (redirect): **/image.jpg** to **/jpg-images/image.jpg**</span></span>

## <a name="using-a-physicalfileprovider"></a><span data-ttu-id="756d2-120">Con un`PhysicalFileProvider`</span><span class="sxs-lookup"><span data-stu-id="756d2-120">Using a `PhysicalFileProvider`</span></span>
<span data-ttu-id="756d2-121">También puede obtener un `IFileProvider` mediante la creación de un `PhysicalFileProvider` que se pasará a la `AddApacheModRewrite()` y `AddIISUrlRewrite()` métodos:</span><span class="sxs-lookup"><span data-stu-id="756d2-121">You can also obtain an `IFileProvider` by creating a `PhysicalFileProvider` to pass into the `AddApacheModRewrite()` and `AddIISUrlRewrite()` methods:</span></span>
```csharp
using Microsoft.Extensions.FileProviders;
PhysicalFileProvider fileProvider = new PhysicalFileProvider(Directory.GetCurrentDirectory());
```
## <a name="secure-redirection-extensions"></a><span data-ttu-id="756d2-122">Extensiones de redirección segura</span><span class="sxs-lookup"><span data-stu-id="756d2-122">Secure redirection extensions</span></span>
<span data-ttu-id="756d2-123">Este ejemplo incluye `WebHostBuilder` configuración de la aplicación usar direcciones URL (**https://localhost:5001**, **https://localhost**) y un certificado de prueba (**testCert.pfx**) para ayudar a explorar estos redirigir métodos.</span><span class="sxs-lookup"><span data-stu-id="756d2-123">This sample includes `WebHostBuilder` configuration for the app to use URLs (**https://localhost:5001**, **https://localhost**) and a test certificate (**testCert.pfx**) to assist you in exploring these redirect methods.</span></span> <span data-ttu-id="756d2-124">Agregar cualquiera de ellos para que la `RewriteOptions()` en **Startup.cs** para estudiar su comportamiento.</span><span class="sxs-lookup"><span data-stu-id="756d2-124">Add any of them to the `RewriteOptions()` in **Startup.cs** to study their behavior.</span></span>

<span data-ttu-id="756d2-125">Método</span><span class="sxs-lookup"><span data-stu-id="756d2-125">Method</span></span> | <span data-ttu-id="756d2-126">Código de estado</span><span class="sxs-lookup"><span data-stu-id="756d2-126">Status Code</span></span> | <span data-ttu-id="756d2-127">Puerto</span><span class="sxs-lookup"><span data-stu-id="756d2-127">Port</span></span>
--- | :---: | :---:
`.AddRedirectToHttpsPermanent()` | <span data-ttu-id="756d2-128">301</span><span class="sxs-lookup"><span data-stu-id="756d2-128">301</span></span> | <span data-ttu-id="756d2-129">null (465)</span><span class="sxs-lookup"><span data-stu-id="756d2-129">null (465)</span></span>
`.AddRedirectToHttps()` | <span data-ttu-id="756d2-130">302</span><span class="sxs-lookup"><span data-stu-id="756d2-130">302</span></span> | <span data-ttu-id="756d2-131">null (465)</span><span class="sxs-lookup"><span data-stu-id="756d2-131">null (465)</span></span>
`.AddRedirectToHttps(301)` | <span data-ttu-id="756d2-132">301</span><span class="sxs-lookup"><span data-stu-id="756d2-132">301</span></span> | <span data-ttu-id="756d2-133">null (465)</span><span class="sxs-lookup"><span data-stu-id="756d2-133">null (465)</span></span>
`.AddRedirectToHttps(301, 5001)` | <span data-ttu-id="756d2-134">301</span><span class="sxs-lookup"><span data-stu-id="756d2-134">301</span></span> | <span data-ttu-id="756d2-135">5001</span><span class="sxs-lookup"><span data-stu-id="756d2-135">5001</span></span>