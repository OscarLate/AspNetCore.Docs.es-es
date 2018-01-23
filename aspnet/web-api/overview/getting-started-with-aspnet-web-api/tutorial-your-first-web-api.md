---
uid: web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api
title: "Introducción a ASP.NET Web API 2 (C#)"
author: MikeWasson
description: "HTTP no es solo para servir las páginas web. También es una herramienta eficaz para la creación de las API que exponen datos y servicios. HTTP es sencilla y flexible y ubiq..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 11/28/2017
ms.topic: article
ms.technology: dotnet-webapi
ms.prod: .net-framework
msc.legacyurl: /web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api
msc.type: authoredcontent
ms.openlocfilehash: fa1cd068a7466e0b6b6fe7716090c8a7afd2a4d5
ms.sourcegitcommit: ec9371e2fbfcb8d62e7e7cae69e7752f3f205385
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/23/2017
---
<a name="getting-started-with-aspnet-web-api-2-c"></a><span data-ttu-id="8702c-105">Introducción a ASP.NET Web API 2 (C#)</span><span class="sxs-lookup"><span data-stu-id="8702c-105">Getting Started with ASP.NET Web API 2 (C#)</span></span>
====================
<span data-ttu-id="8702c-106">por [Mike Wasson](https://github.com/MikeWasson)</span><span class="sxs-lookup"><span data-stu-id="8702c-106">by [Mike Wasson](https://github.com/MikeWasson)</span></span>

[<span data-ttu-id="8702c-107">Descargar el proyecto completado</span><span class="sxs-lookup"><span data-stu-id="8702c-107">Download Completed Project</span></span>](https://code.msdn.microsoft.com/Sample-code-of-Getting-c56ccb28)

<span data-ttu-id="8702c-108">HTTP no es solo para servir las páginas web.</span><span class="sxs-lookup"><span data-stu-id="8702c-108">HTTP is not just for serving up web pages.</span></span> <span data-ttu-id="8702c-109">HTTP es también una plataforma eficaz para la creación de las API que exponen datos y servicios.</span><span class="sxs-lookup"><span data-stu-id="8702c-109">HTTP is also a powerful platform for building APIs that expose services and data.</span></span> <span data-ttu-id="8702c-110">HTTP es sencillo, flexible y ubicua.</span><span class="sxs-lookup"><span data-stu-id="8702c-110">HTTP is simple, flexible, and ubiquitous.</span></span> <span data-ttu-id="8702c-111">Casi cualquier plataforma que se puede considerar tiene una biblioteca de HTTP, por lo que los servicios HTTP pueden llegar a una amplia gama de clientes, incluidos los exploradores, dispositivos móviles y aplicaciones de escritorio tradicionales.</span><span class="sxs-lookup"><span data-stu-id="8702c-111">Almost any platform that you can think of has an HTTP library, so HTTP services can reach a broad range of clients, including browsers, mobile devices, and traditional desktop applications.</span></span>
 
<span data-ttu-id="8702c-112">ASP.NET Web API es un marco para la creación de web API sobre .NET Framework.</span><span class="sxs-lookup"><span data-stu-id="8702c-112">ASP.NET Web API is a framework for building web APIs on top of the .NET Framework.</span></span> <span data-ttu-id="8702c-113">En este tutorial, utilizará ASP.NET Web API para crear un sitio web de API que devuelve una lista de productos.</span><span class="sxs-lookup"><span data-stu-id="8702c-113">In this tutorial, you will use ASP.NET Web API to create a web API that returns a list of products.</span></span>

 
 ## <a name="software-versions-used-in-the-tutorial"></a><span data-ttu-id="8702c-114">Versiones de software que se usa en el tutorial</span><span class="sxs-lookup"><span data-stu-id="8702c-114">Software versions used in the tutorial</span></span>
  
 - [<span data-ttu-id="8702c-115">Visual Studio 2017</span><span class="sxs-lookup"><span data-stu-id="8702c-115">Visual Studio 2017</span></span>](https://www.visualstudio.com/downloads/)
 - <span data-ttu-id="8702c-116">Web API 2</span><span class="sxs-lookup"><span data-stu-id="8702c-116">Web API 2</span></span>

<span data-ttu-id="8702c-117">Vea [crear una API web con ASP.NET Core y Visual Studio para Windows](https://docs.microsoft.com/aspnet/core/tutorials/first-web-api) para una versión más reciente de este tutorial.</span><span class="sxs-lookup"><span data-stu-id="8702c-117">See [Create a web API with ASP.NET Core and Visual Studio for Windows](https://docs.microsoft.com/aspnet/core/tutorials/first-web-api) for a newer version of this tutorial.</span></span>

## <a name="create-a-web-api-project"></a><span data-ttu-id="8702c-118">Cree un proyecto de API Web</span><span class="sxs-lookup"><span data-stu-id="8702c-118">Create a Web API Project</span></span>

<span data-ttu-id="8702c-119">En este tutorial, utilizará ASP.NET Web API para crear un sitio web de API que devuelve una lista de productos.</span><span class="sxs-lookup"><span data-stu-id="8702c-119">In this tutorial, you will use ASP.NET Web API to create a web API that returns a list of products.</span></span> <span data-ttu-id="8702c-120">La página de web front-end utiliza jQuery para mostrar los resultados.</span><span class="sxs-lookup"><span data-stu-id="8702c-120">The front-end web page uses jQuery to display the results.</span></span>

![](tutorial-your-first-web-api/_static/image1.png)

<span data-ttu-id="8702c-121">Inicie Visual Studio y seleccione **nuevo proyecto** desde el **iniciar** página.</span><span class="sxs-lookup"><span data-stu-id="8702c-121">Start Visual Studio and select **New Project** from the **Start** page.</span></span> <span data-ttu-id="8702c-122">O bien, en la **archivo** menú, seleccione **New** y, a continuación, **proyecto**.</span><span class="sxs-lookup"><span data-stu-id="8702c-122">Or, from the **File** menu, select **New** and then **Project**.</span></span>

<span data-ttu-id="8702c-123">En el **plantillas** panel, seleccione **plantillas instaladas** y expanda el **Visual C#** nodo.</span><span class="sxs-lookup"><span data-stu-id="8702c-123">In the **Templates** pane, select **Installed Templates** and expand the **Visual C#** node.</span></span> <span data-ttu-id="8702c-124">En **Visual C#**, seleccione **Web**.</span><span class="sxs-lookup"><span data-stu-id="8702c-124">Under **Visual C#**, select **Web**.</span></span> <span data-ttu-id="8702c-125">En la lista de plantillas de proyecto, seleccione **aplicación Web ASP.NET**.</span><span class="sxs-lookup"><span data-stu-id="8702c-125">In the list of project templates, select **ASP.NET Web Application**.</span></span> <span data-ttu-id="8702c-126">Denomine el proyecto "ProductsApp" y haga clic en **Aceptar**.</span><span class="sxs-lookup"><span data-stu-id="8702c-126">Name the project "ProductsApp" and click **OK**.</span></span>

![](tutorial-your-first-web-api/_static/image2.png)

<span data-ttu-id="8702c-127">En el **nuevo proyecto ASP.NET** cuadro de diálogo, seleccione la **vacía** plantilla.</span><span class="sxs-lookup"><span data-stu-id="8702c-127">In the **New ASP.NET Project** dialog, select the **Empty** template.</span></span> <span data-ttu-id="8702c-128">En &quot;agregar carpetas y principales referencias para&quot;, comprobar **API Web**.</span><span class="sxs-lookup"><span data-stu-id="8702c-128">Under &quot;Add folders and core references for&quot;, check **Web API**.</span></span> <span data-ttu-id="8702c-129">Haga clic en **Aceptar**.</span><span class="sxs-lookup"><span data-stu-id="8702c-129">Click **OK**.</span></span>

![](tutorial-your-first-web-api/_static/image3.png)

> [!NOTE]
> <span data-ttu-id="8702c-130">También puede crear un proyecto de API Web mediante la &quot;API Web&quot; plantilla.</span><span class="sxs-lookup"><span data-stu-id="8702c-130">You can also create a Web API project using the &quot;Web API&quot; template.</span></span> <span data-ttu-id="8702c-131">La plantilla API Web usa ASP.NET MVC para proporcionar páginas de Ayuda de API.</span><span class="sxs-lookup"><span data-stu-id="8702c-131">The Web API template uses ASP.NET MVC to provide API help pages.</span></span> <span data-ttu-id="8702c-132">Estoy usando la plantilla vacía para este tutorial porque desea mostrar API Web sin MVC.</span><span class="sxs-lookup"><span data-stu-id="8702c-132">I'm using the Empty template for this tutorial because I want to show Web API without MVC.</span></span> <span data-ttu-id="8702c-133">En general, no es necesario conocer ASP.NET MVC se usa API Web.</span><span class="sxs-lookup"><span data-stu-id="8702c-133">In general, you don't need to know ASP.NET MVC to use Web API.</span></span>


## <a name="adding-a-model"></a><span data-ttu-id="8702c-134">Agregar un modelo</span><span class="sxs-lookup"><span data-stu-id="8702c-134">Adding a Model</span></span>

<span data-ttu-id="8702c-135">Un *modelo* es un objeto que representa los datos de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="8702c-135">A *model* is an object that represents the data in your application.</span></span> <span data-ttu-id="8702c-136">ASP.NET Web API puede serializar automáticamente el modelo a algún otro formato, JSON o XML y, a continuación, escribir los datos serializados en el cuerpo del mensaje de respuesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="8702c-136">ASP.NET Web API can automatically serialize your model to JSON, XML, or some other format, and then write the serialized data into the body of the HTTP response message.</span></span> <span data-ttu-id="8702c-137">Siempre que un cliente puede leer el formato de serialización, que se puede deserializar el objeto.</span><span class="sxs-lookup"><span data-stu-id="8702c-137">As long as a client can read the serialization format, it can deserialize the object.</span></span> <span data-ttu-id="8702c-138">La mayoría de los clientes pueden analizar XML o JSON.</span><span class="sxs-lookup"><span data-stu-id="8702c-138">Most clients can parse either XML or JSON.</span></span> <span data-ttu-id="8702c-139">Además, el cliente puede indicar el formato que desee estableciendo el encabezado Accept del mensaje de solicitud HTTP.</span><span class="sxs-lookup"><span data-stu-id="8702c-139">Moreover, the client can indicate which format it wants by setting the Accept header in the HTTP request message.</span></span>

<span data-ttu-id="8702c-140">Empecemos por crear un modelo simple que representa un producto.</span><span class="sxs-lookup"><span data-stu-id="8702c-140">Let's start by creating a simple model that represents a product.</span></span>

<span data-ttu-id="8702c-141">Si el Explorador de soluciones no está visible, haga clic en el **vista** menú y seleccione **el Explorador de soluciones**.</span><span class="sxs-lookup"><span data-stu-id="8702c-141">If Solution Explorer is not already visible, click the **View** menu and select **Solution Explorer**.</span></span> <span data-ttu-id="8702c-142">En el Explorador de soluciones, haga clic en la carpeta Models.</span><span class="sxs-lookup"><span data-stu-id="8702c-142">In Solution Explorer, right-click the Models folder.</span></span> <span data-ttu-id="8702c-143">En el menú contextual, seleccione **agregar** , a continuación, seleccione **clase**.</span><span class="sxs-lookup"><span data-stu-id="8702c-143">From the context menu, select **Add** then select **Class**.</span></span>

![](tutorial-your-first-web-api/_static/image4.png)

<span data-ttu-id="8702c-144">Nombre de la clase &quot;producto&quot;.</span><span class="sxs-lookup"><span data-stu-id="8702c-144">Name the class &quot;Product&quot;.</span></span> <span data-ttu-id="8702c-145">Agregue las siguientes propiedades para el `Product` clase.</span><span class="sxs-lookup"><span data-stu-id="8702c-145">Add the following properties to the `Product` class.</span></span>

[!code-csharp[Main](tutorial-your-first-web-api/samples/sample1.cs)]

## <a name="adding-a-controller"></a><span data-ttu-id="8702c-146">Agregar un controlador</span><span class="sxs-lookup"><span data-stu-id="8702c-146">Adding a Controller</span></span>

<span data-ttu-id="8702c-147">En la API de Web, un *controlador* es un objeto que controla las solicitudes HTTP.</span><span class="sxs-lookup"><span data-stu-id="8702c-147">In Web API, a *controller* is an object that handles HTTP requests.</span></span> <span data-ttu-id="8702c-148">Vamos a agregar un controlador que puede devolver una lista de productos o un único producto especificado por identificador.</span><span class="sxs-lookup"><span data-stu-id="8702c-148">We'll add a controller that can return either a list of products or a single product specified by ID.</span></span>

> [!NOTE]
> <span data-ttu-id="8702c-149">Si ha usado ASP.NET MVC, ya está familiarizado con los controladores.</span><span class="sxs-lookup"><span data-stu-id="8702c-149">If you have used ASP.NET MVC, you are already familiar with controllers.</span></span> <span data-ttu-id="8702c-150">Controladores de API Web son similares a los controladores MVC, pero heredan la **ApiController** clase en lugar de la **controlador** clase.</span><span class="sxs-lookup"><span data-stu-id="8702c-150">Web API controllers are similar to MVC controllers, but inherit the **ApiController** class instead of the **Controller** class.</span></span>

<span data-ttu-id="8702c-151">En **el Explorador de soluciones**, haga clic en la carpeta de controladores.</span><span class="sxs-lookup"><span data-stu-id="8702c-151">In **Solution Explorer**, right-click the Controllers folder.</span></span> <span data-ttu-id="8702c-152">Seleccione **agregar** y, a continuación, seleccione **controlador**.</span><span class="sxs-lookup"><span data-stu-id="8702c-152">Select **Add** and then select **Controller**.</span></span>

![](tutorial-your-first-web-api/_static/image5.png)

<span data-ttu-id="8702c-153">En el **agregar scaffolding** cuadro de diálogo, seleccione **controlador de API de Web - vacío**.</span><span class="sxs-lookup"><span data-stu-id="8702c-153">In the **Add Scaffold** dialog, select **Web API Controller - Empty**.</span></span> <span data-ttu-id="8702c-154">Haga clic en **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="8702c-154">Click **Add**.</span></span>

![](tutorial-your-first-web-api/_static/image6.png)

<span data-ttu-id="8702c-155">En el **Agregar controlador** cuadro de diálogo, el nombre del controlador &quot;ProductsController&quot;.</span><span class="sxs-lookup"><span data-stu-id="8702c-155">In the **Add Controller** dialog, name the controller &quot;ProductsController&quot;.</span></span> <span data-ttu-id="8702c-156">Haga clic en **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="8702c-156">Click **Add**.</span></span>

![](tutorial-your-first-web-api/_static/image7.png)

<span data-ttu-id="8702c-157">La técnica scaffolding crea un archivo denominado ProductsController.cs en la carpeta de controladores.</span><span class="sxs-lookup"><span data-stu-id="8702c-157">The scaffolding creates a file named ProductsController.cs in the Controllers folder.</span></span>

![](tutorial-your-first-web-api/_static/image8.png)

> [!NOTE]
> <span data-ttu-id="8702c-158">No es necesario poner los controladores en una carpeta denominada controladores.</span><span class="sxs-lookup"><span data-stu-id="8702c-158">You don't need to put your controllers into a folder named Controllers.</span></span> <span data-ttu-id="8702c-159">El nombre de la carpeta es una manera cómoda de organizar los archivos de origen.</span><span class="sxs-lookup"><span data-stu-id="8702c-159">The folder name is just a convenient way to organize your source files.</span></span>


<span data-ttu-id="8702c-160">Si este archivo ya no está abierto, haga doble clic en el archivo para abrirlo.</span><span class="sxs-lookup"><span data-stu-id="8702c-160">If this file is not open already, double-click the file to open it.</span></span> <span data-ttu-id="8702c-161">Reemplace el código de este archivo con lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="8702c-161">Replace the code in this file with the following:</span></span>

[!code-csharp[Main](tutorial-your-first-web-api/samples/sample2.cs)]

<span data-ttu-id="8702c-162">Para simplificar el ejemplo, los productos se almacenan en una matriz fija dentro de la clase de controlador.</span><span class="sxs-lookup"><span data-stu-id="8702c-162">To keep the example simple, products are stored in a fixed array inside the controller class.</span></span> <span data-ttu-id="8702c-163">Por supuesto, en una aplicación real, podría consultar una base de datos o utilizar algún otro origen de datos externo.</span><span class="sxs-lookup"><span data-stu-id="8702c-163">Of course, in a real application, you would query a database or use some other external data source.</span></span>

<span data-ttu-id="8702c-164">El controlador define dos métodos que devuelven productos:</span><span class="sxs-lookup"><span data-stu-id="8702c-164">The controller defines two methods that return products:</span></span>

- <span data-ttu-id="8702c-165">El `GetAllProducts` método devuelve la lista completa de productos como un **IEnumerable&lt;producto&gt;**  tipo.</span><span class="sxs-lookup"><span data-stu-id="8702c-165">The `GetAllProducts` method returns the entire list of products as an **IEnumerable&lt;Product&gt;** type.</span></span>
- <span data-ttu-id="8702c-166">El `GetProduct` método busca un único producto por su identificador.</span><span class="sxs-lookup"><span data-stu-id="8702c-166">The `GetProduct` method looks up a single product by its ID.</span></span>

<span data-ttu-id="8702c-167">Ya está.</span><span class="sxs-lookup"><span data-stu-id="8702c-167">That's it!</span></span> <span data-ttu-id="8702c-168">Tiene una API web de trabajo.</span><span class="sxs-lookup"><span data-stu-id="8702c-168">You have a working web API.</span></span> <span data-ttu-id="8702c-169">Cada método en el controlador corresponde a uno o varios URI:</span><span class="sxs-lookup"><span data-stu-id="8702c-169">Each method on the controller corresponds to one or more URIs:</span></span>

| <span data-ttu-id="8702c-170">Método de controlador</span><span class="sxs-lookup"><span data-stu-id="8702c-170">Controller Method</span></span> | <span data-ttu-id="8702c-171">Identificador URI</span><span class="sxs-lookup"><span data-stu-id="8702c-171">URI</span></span> |
| --- | --- |
| <span data-ttu-id="8702c-172">GetAllProducts</span><span class="sxs-lookup"><span data-stu-id="8702c-172">GetAllProducts</span></span> | <span data-ttu-id="8702c-173">productos/api /</span><span class="sxs-lookup"><span data-stu-id="8702c-173">/api/products</span></span> |
| <span data-ttu-id="8702c-174">GetProduct</span><span class="sxs-lookup"><span data-stu-id="8702c-174">GetProduct</span></span> | <span data-ttu-id="8702c-175">/API/products/*Id.*</span><span class="sxs-lookup"><span data-stu-id="8702c-175">/api/products/*id*</span></span> |

<span data-ttu-id="8702c-176">Para el `GetProduct` método, el *identificador* en el URI es un marcador de posición.</span><span class="sxs-lookup"><span data-stu-id="8702c-176">For the `GetProduct` method, the *id* in the URI is a placeholder.</span></span> <span data-ttu-id="8702c-177">Por ejemplo, para obtener el producto con el identificador de 5, el URI es `api/products/5`.</span><span class="sxs-lookup"><span data-stu-id="8702c-177">For example, to get the product with ID of 5, the URI is `api/products/5`.</span></span>

<span data-ttu-id="8702c-178">Para obtener más información acerca de cómo Web API enruta las solicitudes HTTP a los métodos de controlador, consulte [enrutamiento de ASP.NET Web API](../web-api-routing-and-actions/routing-in-aspnet-web-api.md).</span><span class="sxs-lookup"><span data-stu-id="8702c-178">For more information about how Web API routes HTTP requests to controller methods, see [Routing in ASP.NET Web API](../web-api-routing-and-actions/routing-in-aspnet-web-api.md).</span></span>

## <a name="calling-the-web-api-with-javascript-and-jquery"></a><span data-ttu-id="8702c-179">Llamar a la API Web con Javascript y jQuery</span><span class="sxs-lookup"><span data-stu-id="8702c-179">Calling the Web API with Javascript and jQuery</span></span>

<span data-ttu-id="8702c-180">En esta sección, vamos a agregar una página HTML que se usa AJAX para llamar a la API web.</span><span class="sxs-lookup"><span data-stu-id="8702c-180">In this section, we'll add an HTML page that uses AJAX to call the web API.</span></span> <span data-ttu-id="8702c-181">Usaremos jQuery para realizar las llamadas de AJAX y también para actualizar la página con los resultados.</span><span class="sxs-lookup"><span data-stu-id="8702c-181">We'll use jQuery to make the AJAX calls and also to update the page with the results.</span></span>

<span data-ttu-id="8702c-182">En el Explorador de soluciones, haga clic en el proyecto y seleccione **agregar**, a continuación, seleccione **nuevo elemento**.</span><span class="sxs-lookup"><span data-stu-id="8702c-182">In Solution Explorer, right-click the project and select **Add**, then select **New Item**.</span></span>

![](tutorial-your-first-web-api/_static/image9.png)

<span data-ttu-id="8702c-183">En el **Agregar nuevo elemento** cuadro de diálogo, seleccione la **Web** nodo bajo **Visual C#**y, a continuación, seleccione la **página HTML** elemento.</span><span class="sxs-lookup"><span data-stu-id="8702c-183">In the **Add New Item** dialog, select the **Web** node under **Visual C#**, and then select the **HTML Page** item.</span></span> <span data-ttu-id="8702c-184">Nombre de la página &quot;index.html&quot;.</span><span class="sxs-lookup"><span data-stu-id="8702c-184">Name the page &quot;index.html&quot;.</span></span>

![](tutorial-your-first-web-api/_static/image10.png)

<span data-ttu-id="8702c-185">Reemplace todo el contenido de este archivo con lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="8702c-185">Replace everything in this file with the following:</span></span>

[!code-html[Main](tutorial-your-first-web-api/samples/sample3.html)]

<span data-ttu-id="8702c-186">Hay varias maneras de obtener jQuery.</span><span class="sxs-lookup"><span data-stu-id="8702c-186">There are several ways to get jQuery.</span></span> <span data-ttu-id="8702c-187">En este ejemplo, usa el [CDN de Microsoft Ajax](../../../ajax/cdn/overview.md).</span><span class="sxs-lookup"><span data-stu-id="8702c-187">In this example, I used the [Microsoft Ajax CDN](../../../ajax/cdn/overview.md).</span></span> <span data-ttu-id="8702c-188">También puede descargarlo desde [http://jquery.com/](http://jquery.com/)y ASP.NET "Web API" plantilla de proyecto incluye también jQuery.</span><span class="sxs-lookup"><span data-stu-id="8702c-188">You can also download it from [http://jquery.com/](http://jquery.com/), and the ASP.NET "Web API" project template includes jQuery as well.</span></span>

### <a name="getting-a-list-of-products"></a><span data-ttu-id="8702c-189">Obtener una lista de productos</span><span class="sxs-lookup"><span data-stu-id="8702c-189">Getting a List of Products</span></span>

<span data-ttu-id="8702c-190">Para obtener una lista de productos, envíe una solicitud HTTP GET a &quot;productos/api/&quot;.</span><span class="sxs-lookup"><span data-stu-id="8702c-190">To get a list of products, send an HTTP GET request to &quot;/api/products&quot;.</span></span>

<span data-ttu-id="8702c-191">JQuery [getJSON](http://api.jquery.com/jQuery.getJSON/) función envía una solicitud AJAX.</span><span class="sxs-lookup"><span data-stu-id="8702c-191">The jQuery [getJSON](http://api.jquery.com/jQuery.getJSON/) function sends an AJAX request.</span></span> <span data-ttu-id="8702c-192">Para la respuesta contiene la matriz de objetos JSON.</span><span class="sxs-lookup"><span data-stu-id="8702c-192">For response contains array of JSON objects.</span></span> <span data-ttu-id="8702c-193">El `done` función especifica una devolución de llamada que se llama si la solicitud se realiza correctamente.</span><span class="sxs-lookup"><span data-stu-id="8702c-193">The `done` function specifies a callback that is called if the request succeeds.</span></span> <span data-ttu-id="8702c-194">En la devolución de llamada, actualizamos el DOM con la información del producto.</span><span class="sxs-lookup"><span data-stu-id="8702c-194">In the callback, we update the DOM with the product information.</span></span>

[!code-html[Main](tutorial-your-first-web-api/samples/sample4.html)]

### <a name="getting-a-product-by-id"></a><span data-ttu-id="8702c-195">Obtención de un producto por Id.</span><span class="sxs-lookup"><span data-stu-id="8702c-195">Getting a Product By ID</span></span>

<span data-ttu-id="8702c-196">Para obtener un producto por el identificador, envíe una solicitud HTTP GET a &quot;/API/products/*identificador*&quot;, donde *identificador* es el identificador de producto.</span><span class="sxs-lookup"><span data-stu-id="8702c-196">To get a product by ID, send an HTTP GET request to &quot;/api/products/*id*&quot;, where *id* is the product ID.</span></span>

[!code-javascript[Main](tutorial-your-first-web-api/samples/sample5.js)]

<span data-ttu-id="8702c-197">Lo llamamos todavía `getJSON` para enviar la solicitud de AJAX, pero esta vez colocamos el identificador en el URI de solicitud.</span><span class="sxs-lookup"><span data-stu-id="8702c-197">We still call `getJSON` to send the AJAX request, but this time we put the ID in the request URI.</span></span> <span data-ttu-id="8702c-198">La respuesta de esta solicitud es una representación JSON de un único producto.</span><span class="sxs-lookup"><span data-stu-id="8702c-198">The response from this request is a JSON representation of a single product.</span></span>

## <a name="running-the-application"></a><span data-ttu-id="8702c-199">Ejecutar la aplicación</span><span class="sxs-lookup"><span data-stu-id="8702c-199">Running the Application</span></span>

<span data-ttu-id="8702c-200">Presione F5 para iniciar la depuración de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="8702c-200">Press F5 to start debugging the application.</span></span> <span data-ttu-id="8702c-201">La página web debe ser similar al siguiente:</span><span class="sxs-lookup"><span data-stu-id="8702c-201">The web page should look like the following:</span></span>

![](tutorial-your-first-web-api/_static/image11.png)

<span data-ttu-id="8702c-202">Para obtener un producto por el identificador, escriba el identificador y haga clic en Buscar:</span><span class="sxs-lookup"><span data-stu-id="8702c-202">To get a product by ID, enter the ID and click Search:</span></span>

![](tutorial-your-first-web-api/_static/image12.png)

<span data-ttu-id="8702c-203">Si escribe un identificador no válido, el servidor devuelve un error HTTP:</span><span class="sxs-lookup"><span data-stu-id="8702c-203">If you enter an invalid ID, the server returns an HTTP error:</span></span>

![](tutorial-your-first-web-api/_static/image13.png)

## <a name="using-f12-to-view-the-http-request-and-response"></a><span data-ttu-id="8702c-204">Si se usa F12 para ver la solicitud y respuesta HTTP</span><span class="sxs-lookup"><span data-stu-id="8702c-204">Using F12 to View the HTTP Request and Response</span></span>

<span data-ttu-id="8702c-205">Cuando se trabaja con un servicio HTTP, puede ser muy útil ver la solicitud HTTP y mensajes de solicitud.</span><span class="sxs-lookup"><span data-stu-id="8702c-205">When you are working with an HTTP service, it can be very useful to see the HTTP request and request messages.</span></span> <span data-ttu-id="8702c-206">Puede hacerlo mediante el uso de las herramientas de desarrollo F12 de Internet Explorer 9.</span><span class="sxs-lookup"><span data-stu-id="8702c-206">You can do this by using the F12 developer tools in Internet Explorer 9.</span></span> <span data-ttu-id="8702c-207">En Internet Explorer 9, presione **F12** para abrir las herramientas.</span><span class="sxs-lookup"><span data-stu-id="8702c-207">From Internet Explorer 9, press **F12** to open the tools.</span></span> <span data-ttu-id="8702c-208">Haga clic en el **red** pestaña y presione **Iniciar captura**.</span><span class="sxs-lookup"><span data-stu-id="8702c-208">Click the **Network** tab and press **Start Capturing**.</span></span> <span data-ttu-id="8702c-209">Ahora, vaya a la página web y presione **F5** para volver a cargar la página web.</span><span class="sxs-lookup"><span data-stu-id="8702c-209">Now go back to the web page and press **F5** to reload the web page.</span></span> <span data-ttu-id="8702c-210">Internet Explorer capturará el tráfico HTTP entre el explorador y el servidor web.</span><span class="sxs-lookup"><span data-stu-id="8702c-210">Internet Explorer will capture the HTTP traffic between the browser and the web server.</span></span> <span data-ttu-id="8702c-211">La vista de resumen muestra todo el tráfico de red para una página:</span><span class="sxs-lookup"><span data-stu-id="8702c-211">The summary view shows all the network traffic for a page:</span></span>

![](tutorial-your-first-web-api/_static/image14.png)

<span data-ttu-id="8702c-212">Busque la entrada para el URI relativo "api/productos /".</span><span class="sxs-lookup"><span data-stu-id="8702c-212">Locate the entry for the relative URI "api/products/".</span></span> <span data-ttu-id="8702c-213">Seleccione esta entrada y haga clic en **vaya a la vista detallada**.</span><span class="sxs-lookup"><span data-stu-id="8702c-213">Select this entry and click **Go to detailed view**.</span></span> <span data-ttu-id="8702c-214">En la vista de detalle, hay pestañas para ver los encabezados de solicitud y respuesta y cuerpos.</span><span class="sxs-lookup"><span data-stu-id="8702c-214">In the detail view, there are tabs to view the request and response headers and bodies.</span></span> <span data-ttu-id="8702c-215">Por ejemplo, si hace clic en el **encabezados de solicitud** ficha, puede ver que el cliente solicitó &quot;application/json&quot; en el encabezado Accept.</span><span class="sxs-lookup"><span data-stu-id="8702c-215">For example, if you click the **Request headers** tab, you can see that the client requested &quot;application/json&quot; in the Accept header.</span></span>

![](tutorial-your-first-web-api/_static/image15.png)

<span data-ttu-id="8702c-216">Si hace clic en la ficha cuerpo de respuesta, puede ver cómo la lista de productos se serializa a JSON.</span><span class="sxs-lookup"><span data-stu-id="8702c-216">If you click the Response body tab, you can see how the product list was serialized to JSON.</span></span> <span data-ttu-id="8702c-217">Otros exploradores tienen una funcionalidad similar.</span><span class="sxs-lookup"><span data-stu-id="8702c-217">Other browsers have similar functionality.</span></span> <span data-ttu-id="8702c-218">Otra herramienta útil es [Fiddler](http://www.fiddler2.com/fiddler2/), un sitio web proxy de depuración.</span><span class="sxs-lookup"><span data-stu-id="8702c-218">Another useful tool is [Fiddler](http://www.fiddler2.com/fiddler2/), a web debugging proxy.</span></span> <span data-ttu-id="8702c-219">Se puede utilizar Fiddler para ver el tráfico HTTP y también para crear solicitudes HTTP, que proporciona control total sobre los encabezados HTTP en la solicitud.</span><span class="sxs-lookup"><span data-stu-id="8702c-219">You can use Fiddler to view your HTTP traffic, and also to compose HTTP requests, which gives you full control over the HTTP headers in the request.</span></span>

## <a name="see-this-app-running-on-azure"></a><span data-ttu-id="8702c-220">Vea esta aplicación se ejecuta en Azure</span><span class="sxs-lookup"><span data-stu-id="8702c-220">See this App Running on Azure</span></span>

<span data-ttu-id="8702c-221">¿Desea ver el sitio terminado que se ejecuta como una aplicación web en directo?</span><span class="sxs-lookup"><span data-stu-id="8702c-221">Would you like to see the finished site running as a live web app?</span></span> <span data-ttu-id="8702c-222">Puede implementar una versión completa de la aplicación en su cuenta de Azure, simplemente haga clic en el botón siguiente.</span><span class="sxs-lookup"><span data-stu-id="8702c-222">You can deploy a complete version of the app to your Azure account by simply clicking the following button.</span></span>

[![](http://azuredeploy.net/deploybutton.png)](https://deploy.azure.com/?WT.mc_id=deploy_azure_aspnet&repository=https://github.com/tfitzmac/WebAPI-ProductsApp#/form/setup)

<span data-ttu-id="8702c-223">Necesita una cuenta de Azure para implementar esta solución en Azure.</span><span class="sxs-lookup"><span data-stu-id="8702c-223">You need an Azure account to deploy this solution to Azure.</span></span> <span data-ttu-id="8702c-224">Si no dispone de una cuenta, tiene las siguientes opciones:</span><span class="sxs-lookup"><span data-stu-id="8702c-224">If you do not already have an account, you have the following options:</span></span>

- <span data-ttu-id="8702c-225">[Abrir una cuenta de Azure de forma gratuita](https://azure.microsoft.com/en-us/pricing/free-trial/?WT.mc_id=A443DD604) -obtendrá créditos puede usar para probar los servicios de Azure de pagados e incluso después de que se utilizan hasta puede mantener la cuenta y libre de usar los servicios de Azure.</span><span class="sxs-lookup"><span data-stu-id="8702c-225">[Open an Azure account for free](https://azure.microsoft.com/en-us/pricing/free-trial/?WT.mc_id=A443DD604) - You get credits you can use to try out paid Azure services, and even after they're used up you can keep the account and use free Azure services.</span></span>
- <span data-ttu-id="8702c-226">[Activar las ventajas de suscriptor MSDN](https://azure.microsoft.com/en-us/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A443DD604) -su suscripción a MSDN ofrece créditos cada mes que puede usar para los servicios de Azure de pagados.</span><span class="sxs-lookup"><span data-stu-id="8702c-226">[Activate MSDN subscriber benefits](https://azure.microsoft.com/en-us/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A443DD604) - Your MSDN subscription gives you credits every month that you can use for paid Azure services.</span></span>

## <a name="next-steps"></a><span data-ttu-id="8702c-227">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="8702c-227">Next Steps</span></span>

- <span data-ttu-id="8702c-228">Para obtener un ejemplo más completo de un servicio HTTP que admite acciones de POST, PUT y DELETE y escribe en una base de datos, vea [usar Web API 2 con Entity Framework 6](../data/using-web-api-with-entity-framework/part-1.md).</span><span class="sxs-lookup"><span data-stu-id="8702c-228">For a more complete example of an HTTP service that supports POST, PUT, and DELETE actions and writes to a database, see [Using Web API 2 with Entity Framework 6](../data/using-web-api-with-entity-framework/part-1.md).</span></span>
- <span data-ttu-id="8702c-229">Para obtener más información sobre cómo crear aplicaciones sensibles y fluido web encima de un servicio HTTP, consulte [aplicación de una única página ASP.NET](../../../single-page-application/index.md).</span><span class="sxs-lookup"><span data-stu-id="8702c-229">For more about creating fluid and responsive web applications on top of an HTTP service, see [ASP.NET Single Page Application](../../../single-page-application/index.md).</span></span>
- <span data-ttu-id="8702c-230">Para obtener información sobre cómo implementar un proyecto web de Visual Studio al servicio de aplicaciones de Azure, consulte [crear una aplicación web ASP.NET en el servicio de aplicación de Azure](https://azure.microsoft.com/en-us/documentation/articles/web-sites-dotnet-get-started/).</span><span class="sxs-lookup"><span data-stu-id="8702c-230">For information about how to deploy a Visual Studio web project to Azure App Service, see [Create an ASP.NET web app in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/web-sites-dotnet-get-started/).</span></span>