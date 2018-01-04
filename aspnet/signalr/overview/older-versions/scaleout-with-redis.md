---
uid: signalr/overview/older-versions/scaleout-with-redis
title: "Ampliación de SignalR con Redis (SignalR 1.x) | Documentos de Microsoft"
author: MikeWasson
description: 
ms.author: aspnetcontent
manager: wpickett
ms.date: 05/01/2013
ms.topic: article
ms.assetid: 6abecf80-8ffa-41ba-b0d9-1d9edbe7687b
ms.technology: dotnet-signalr
ms.prod: .net-framework
msc.legacyurl: /signalr/overview/older-versions/scaleout-with-redis
msc.type: authoredcontent
ms.openlocfilehash: c0d6fd421dad02298326d1975ae68d1e7cc78d8c
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/10/2017
---
<a name="signalr-scaleout-with-redis-signalr-1x"></a><span data-ttu-id="bf59e-102">Ampliación de SignalR con Redis (SignalR 1.x)</span><span class="sxs-lookup"><span data-stu-id="bf59e-102">SignalR Scaleout with Redis (SignalR 1.x)</span></span>
====================
<span data-ttu-id="bf59e-103">por [Mike Wasson](https://github.com/MikeWasson), [Patrick Fletcher](https://github.com/pfletcher)</span><span class="sxs-lookup"><span data-stu-id="bf59e-103">by [Mike Wasson](https://github.com/MikeWasson), [Patrick Fletcher](https://github.com/pfletcher)</span></span>

<span data-ttu-id="bf59e-104">En este tutorial, va a usar [Redis](http://redis.io/) para distribuir mensajes en una aplicación de SignalR que se implementa en dos instancias diferentes de IIS.</span><span class="sxs-lookup"><span data-stu-id="bf59e-104">In this tutorial, you will use [Redis](http://redis.io/) to distribute messages across a SignalR application that is deployed on two separate IIS instances.</span></span>

<span data-ttu-id="bf59e-105">Redis es un almacén de clave y valor en memoria.</span><span class="sxs-lookup"><span data-stu-id="bf59e-105">Redis is an in-memory key-value store.</span></span> <span data-ttu-id="bf59e-106">También admite un sistema de mensajería con un modelo de publicación/suscripción.</span><span class="sxs-lookup"><span data-stu-id="bf59e-106">It also supports a messaging system with a publish/subscribe model.</span></span> <span data-ttu-id="bf59e-107">El backplane SignalR Redis usa la característica de pub/sub para reenviar los mensajes a otros servidores.</span><span class="sxs-lookup"><span data-stu-id="bf59e-107">The SignalR Redis backplane uses the pub/sub feature to forward messages to other servers.</span></span>

![](scaleout-with-redis/_static/image1.png)

<span data-ttu-id="bf59e-108">Para este tutorial, utilizará los tres servidores:</span><span class="sxs-lookup"><span data-stu-id="bf59e-108">For this tutorial, you will use three servers:</span></span>

- <span data-ttu-id="bf59e-109">Dos servidores que ejecutan Windows, que se usará para implementar una aplicación de SignalR.</span><span class="sxs-lookup"><span data-stu-id="bf59e-109">Two servers running Windows, which you will use to deploy a SignalR application.</span></span>
- <span data-ttu-id="bf59e-110">Un servidor que ejecuta Linux, que se usará para ejecutar Redis.</span><span class="sxs-lookup"><span data-stu-id="bf59e-110">One server running Linux, which you will use to run Redis.</span></span> <span data-ttu-id="bf59e-111">Para las capturas de pantalla en este tutorial, he usado Ubuntu 12.04 TLS.</span><span class="sxs-lookup"><span data-stu-id="bf59e-111">For the screenshots in this tutorial, I used Ubuntu 12.04 TLS.</span></span>

<span data-ttu-id="bf59e-112">Si no tiene tres servidores físicos para usar, puede crear máquinas virtuales en Hyper-V.</span><span class="sxs-lookup"><span data-stu-id="bf59e-112">If you don't have three physical servers to use, you can create VMs on Hyper-V.</span></span> <span data-ttu-id="bf59e-113">Otra opción es crear máquinas virtuales en Azure.</span><span class="sxs-lookup"><span data-stu-id="bf59e-113">Another option is to create VMs on Azure.</span></span>

<span data-ttu-id="bf59e-114">Aunque este tutorial usa la implementación de Redis oficial, hay también un [Windows puerto de Redis](https://github.com/MSOpenTech/redis) desde MSOpenTech.</span><span class="sxs-lookup"><span data-stu-id="bf59e-114">Although this tutorial uses the official Redis implementation, there is also a [Windows port of Redis](https://github.com/MSOpenTech/redis) from MSOpenTech.</span></span> <span data-ttu-id="bf59e-115">El programa de instalación y configuración son diferentes, pero en caso contrario, los pasos son los mismos.</span><span class="sxs-lookup"><span data-stu-id="bf59e-115">Setup and configuration are different, but otherwise the steps are the same.</span></span>

> [!NOTE] 
> 
> <span data-ttu-id="bf59e-116">Ampliación de SignalR con Redis no es compatible con clústeres de Redis.</span><span class="sxs-lookup"><span data-stu-id="bf59e-116">SignalR scaleout with Redis does not support Redis clusters.</span></span>


## <a name="overview"></a><span data-ttu-id="bf59e-117">Información general</span><span class="sxs-lookup"><span data-stu-id="bf59e-117">Overview</span></span>

<span data-ttu-id="bf59e-118">Antes de entrar en el tutorial detallado, presentamos una introducción rápida de lo va a hacer.</span><span class="sxs-lookup"><span data-stu-id="bf59e-118">Before we get to the detailed tutorial, here is a quick overview of what you will do.</span></span>

1. <span data-ttu-id="bf59e-119">Instale Redis e inicie el servidor de Redis.</span><span class="sxs-lookup"><span data-stu-id="bf59e-119">Install Redis and start the Redis server.</span></span>
2. <span data-ttu-id="bf59e-120">Agregue estos paquetes de NuGet para la aplicación:</span><span class="sxs-lookup"><span data-stu-id="bf59e-120">Add these NuGet packages to your application:</span></span> 

    - [<span data-ttu-id="bf59e-121">Microsoft.AspNet.SignalR</span><span class="sxs-lookup"><span data-stu-id="bf59e-121">Microsoft.AspNet.SignalR</span></span>](http://nuget.org/packages/Microsoft.AspNet.SignalR)
    - [<span data-ttu-id="bf59e-122">Microsoft.AspNet.SignalR.Redis</span><span class="sxs-lookup"><span data-stu-id="bf59e-122">Microsoft.AspNet.SignalR.Redis</span></span>](http://nuget.org/packages/Microsoft.AspNet.SignalR.Redis)
3. <span data-ttu-id="bf59e-123">Cree una aplicación de SignalR.</span><span class="sxs-lookup"><span data-stu-id="bf59e-123">Create a SignalR application.</span></span>
4. <span data-ttu-id="bf59e-124">Agregue el código siguiente a Global.asax para configurar el plano posterior:</span><span class="sxs-lookup"><span data-stu-id="bf59e-124">Add the following code to Global.asax to configure the backplane:</span></span> 

    [!code-csharp[Main](scaleout-with-redis/samples/sample1.cs)]

## <a name="ubuntu-on-hyper-v"></a><span data-ttu-id="bf59e-125">Ubuntu en Hyper-V</span><span class="sxs-lookup"><span data-stu-id="bf59e-125">Ubuntu on Hyper-V</span></span>

<span data-ttu-id="bf59e-126">Con Hyper-V de Windows, puede crear fácilmente una VM Ubuntu en Windows Server.</span><span class="sxs-lookup"><span data-stu-id="bf59e-126">Using Windows Hyper-V, you can easily create an Ubuntu VM on Windows Server.</span></span>

<span data-ttu-id="bf59e-127">Descargar la imagen ISO de Ubuntu de [http://www.ubuntu.com](http://www.ubuntu.com/).</span><span class="sxs-lookup"><span data-stu-id="bf59e-127">Download the Ubuntu ISO from [http://www.ubuntu.com](http://www.ubuntu.com/).</span></span>

<span data-ttu-id="bf59e-128">En Hyper-V, agregue una nueva máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="bf59e-128">In Hyper-V, add a new VM.</span></span> <span data-ttu-id="bf59e-129">En el **conectar disco duro Virtual** paso, seleccione **crear un disco duro virtual**.</span><span class="sxs-lookup"><span data-stu-id="bf59e-129">In the **Connect Virtual Hard Disk** step, select **Create a virtual hard disk**.</span></span>

![](scaleout-with-redis/_static/image2.png)

<span data-ttu-id="bf59e-130">En el **opciones de instalación** paso, seleccione **archivo de imagen (.iso)**, haga clic en **examinar**y vaya a la instalación de Ubuntu ISO.</span><span class="sxs-lookup"><span data-stu-id="bf59e-130">In the **Installation Options** step, select **Image file (.iso)**, click **Browse**, and browse to the Ubuntu installation ISO.</span></span>

![](scaleout-with-redis/_static/image3.png)

## <a name="install-redis"></a><span data-ttu-id="bf59e-131">Instalar Redis</span><span class="sxs-lookup"><span data-stu-id="bf59e-131">Install Redis</span></span>

<span data-ttu-id="bf59e-132">Siga los pasos de [http://redis.io/download](http://redis.io/download) para descargar y generar Redis.</span><span class="sxs-lookup"><span data-stu-id="bf59e-132">Follow the steps at [http://redis.io/download](http://redis.io/download) to download and build Redis.</span></span>

[!code-console[Main](scaleout-with-redis/samples/sample2.cmd)]

<span data-ttu-id="bf59e-133">Esto compila los archivos binarios de Redis el `src` directory.</span><span class="sxs-lookup"><span data-stu-id="bf59e-133">This builds the Redis binaries in the `src` directory.</span></span>

<span data-ttu-id="bf59e-134">De forma predeterminada, Redis no requiere una contraseña.</span><span class="sxs-lookup"><span data-stu-id="bf59e-134">By default, Redis does not require a password.</span></span> <span data-ttu-id="bf59e-135">Para establecer una contraseña, edite el `redis.conf` archivo, que se encuentra en el directorio raíz del código fuente.</span><span class="sxs-lookup"><span data-stu-id="bf59e-135">To set a password, edit the `redis.conf` file, which is located in the root directory of the source code.</span></span> <span data-ttu-id="bf59e-136">(Hacer una copia de seguridad del archivo antes de modificarlo!) Agregue la siguiente directiva a `redis.conf`:</span><span class="sxs-lookup"><span data-stu-id="bf59e-136">(Make a backup copy of the file before you edit it!) Add the following directive to `redis.conf`:</span></span>

[!code-powershell[Main](scaleout-with-redis/samples/sample3.ps1)]

<span data-ttu-id="bf59e-137">Ahora, inicie el servidor de Redis:</span><span class="sxs-lookup"><span data-stu-id="bf59e-137">Now start the Redis server:</span></span>

[!code-css[Main](scaleout-with-redis/samples/sample4.css)]

![](scaleout-with-redis/_static/image4.png)

<span data-ttu-id="bf59e-138">Abrir el puerto 6379, que es el puerto predeterminado que Redis escucha en.</span><span class="sxs-lookup"><span data-stu-id="bf59e-138">Open port 6379, which is the default port that Redis listens on.</span></span> <span data-ttu-id="bf59e-139">(Puede cambiar el número de puerto en el archivo de configuración).</span><span class="sxs-lookup"><span data-stu-id="bf59e-139">(You can change the port number in the configuration file.)</span></span>

## <a name="create-the-signalr-application"></a><span data-ttu-id="bf59e-140">Crear la aplicación de SignalR</span><span class="sxs-lookup"><span data-stu-id="bf59e-140">Create the SignalR Application</span></span>

<span data-ttu-id="bf59e-141">Crear una aplicación de SignalR, siga cualquiera de estos tutoriales:</span><span class="sxs-lookup"><span data-stu-id="bf59e-141">Create a SignalR application by following either of these tutorials:</span></span>

- [<span data-ttu-id="bf59e-142">Introducción a SignalR</span><span class="sxs-lookup"><span data-stu-id="bf59e-142">Getting Started with SignalR</span></span>](../getting-started/tutorial-getting-started-with-signalr.md)
- [<span data-ttu-id="bf59e-143">Introducción a SignalR y MVC 4</span><span class="sxs-lookup"><span data-stu-id="bf59e-143">Getting Started with SignalR and MVC 4</span></span>](tutorial-getting-started-with-signalr-and-mvc-4.md)

<span data-ttu-id="bf59e-144">A continuación, modificaremos la aplicación de chat para admitir la ampliación con Redis.</span><span class="sxs-lookup"><span data-stu-id="bf59e-144">Next, we'll modify the chat application to support scaleout with Redis.</span></span> <span data-ttu-id="bf59e-145">En primer lugar, agregue el paquete de SignalR.Redis NuGet al proyecto.</span><span class="sxs-lookup"><span data-stu-id="bf59e-145">First, add the SignalR.Redis NuGet package to your project.</span></span> <span data-ttu-id="bf59e-146">En Visual Studio, desde la **herramientas** menú, seleccione **Administrador de paquetes de biblioteca**, a continuación, seleccione **Package Manager Console**.</span><span class="sxs-lookup"><span data-stu-id="bf59e-146">In Visual Studio, from the **Tools** menu, select **Library Package Manager**, then select **Package Manager Console**.</span></span> <span data-ttu-id="bf59e-147">En la ventana de la consola de administrador de paquetes, escriba el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="bf59e-147">In the Package Manager Console window, enter the following command:</span></span>

[!code-powershell[Main](scaleout-with-redis/samples/sample5.ps1)]

<span data-ttu-id="bf59e-148">A continuación, abra el archivo Global.asax.</span><span class="sxs-lookup"><span data-stu-id="bf59e-148">Next, open the Global.asax file.</span></span> <span data-ttu-id="bf59e-149">Agregue el código siguiente a la **aplicación\_iniciar** método:</span><span class="sxs-lookup"><span data-stu-id="bf59e-149">Add the following code to the **Application\_Start** method:</span></span>

[!code-csharp[Main](scaleout-with-redis/samples/sample6.cs)]

- <span data-ttu-id="bf59e-150">"servidor" es el nombre del servidor que está ejecutando Redis.</span><span class="sxs-lookup"><span data-stu-id="bf59e-150">"server" is the name of the server that is running Redis.</span></span>
- <span data-ttu-id="bf59e-151">*puerto* es el número de puerto</span><span class="sxs-lookup"><span data-stu-id="bf59e-151">*port* is the port number</span></span>
- <span data-ttu-id="bf59e-152">"password" es la contraseña que ha definido en el archivo redis.conf.</span><span class="sxs-lookup"><span data-stu-id="bf59e-152">"password" is the password that you defined in the redis.conf file.</span></span>
- <span data-ttu-id="bf59e-153">"AppName" es cualquier cadena.</span><span class="sxs-lookup"><span data-stu-id="bf59e-153">"AppName" is any string.</span></span> <span data-ttu-id="bf59e-154">SignalR crea un canal de pub/sub de Redis con este nombre.</span><span class="sxs-lookup"><span data-stu-id="bf59e-154">SignalR creates a Redis pub/sub channel with this name.</span></span>

<span data-ttu-id="bf59e-155">Por ejemplo:</span><span class="sxs-lookup"><span data-stu-id="bf59e-155">For example:</span></span>

[!code-csharp[Main](scaleout-with-redis/samples/sample7.cs)]

## <a name="deploy-and-run-the-application"></a><span data-ttu-id="bf59e-156">Implementar y ejecutar la aplicación</span><span class="sxs-lookup"><span data-stu-id="bf59e-156">Deploy and Run the Application</span></span>

<span data-ttu-id="bf59e-157">Preparar las instancias del servidor de Windows para implementar la aplicación de SignalR.</span><span class="sxs-lookup"><span data-stu-id="bf59e-157">Prepare your Windows Server instances to deploy the SignalR application.</span></span>

<span data-ttu-id="bf59e-158">Agregue el rol IIS.</span><span class="sxs-lookup"><span data-stu-id="bf59e-158">Add the IIS role.</span></span> <span data-ttu-id="bf59e-159">Incluye características de "Desarrollo de aplicaciones", incluido el protocolo WebSocket.</span><span class="sxs-lookup"><span data-stu-id="bf59e-159">Include "Application Development" features, including the WebSocket Protocol.</span></span>

![](scaleout-with-redis/_static/image5.png)

<span data-ttu-id="bf59e-160">También incluye el servicio de administración (aparecen en "Herramientas de administración").</span><span class="sxs-lookup"><span data-stu-id="bf59e-160">Also include the Management Service (listed under "Management Tools").</span></span>

![](scaleout-with-redis/_static/image6.png)

<span data-ttu-id="bf59e-161">**Instalar Web Deploy 3.0.**</span><span class="sxs-lookup"><span data-stu-id="bf59e-161">**Install Web Deploy 3.0.**</span></span> <span data-ttu-id="bf59e-162">Cuando se ejecuta el Administrador de IIS, se le pedirá que instale la plataforma Web de Microsoft, o bien puede [descargar el intstaller](https://go.microsoft.com/fwlink/?LinkId=255386).</span><span class="sxs-lookup"><span data-stu-id="bf59e-162">When you run IIS Manager, it will prompt you to install Microsoft Web Platform, or you can [download the intstaller](https://go.microsoft.com/fwlink/?LinkId=255386).</span></span> <span data-ttu-id="bf59e-163">En el instalador de plataforma, buscar Web Deploy e instalar Web Deploy 3.0</span><span class="sxs-lookup"><span data-stu-id="bf59e-163">In the Platform Installer, search for Web Deploy and install Web Deploy 3.0</span></span>

![](scaleout-with-redis/_static/image7.png)

<span data-ttu-id="bf59e-164">Compruebe que se está ejecutando el servicio de administración de Web.</span><span class="sxs-lookup"><span data-stu-id="bf59e-164">Check that the Web Management Service is running.</span></span> <span data-ttu-id="bf59e-165">Si no es así, inicie el servicio.</span><span class="sxs-lookup"><span data-stu-id="bf59e-165">If not, start the service.</span></span> <span data-ttu-id="bf59e-166">(Si no ve el servicio de administración Web en la lista de servicios de Windows, asegúrese de que instala el servicio de administración cuando se agrega el rol IIS.)</span><span class="sxs-lookup"><span data-stu-id="bf59e-166">(If you don't see Web Management Service in the list of Windows services, make sure that you installed the Management Service when you added the IIS role.)</span></span>

<span data-ttu-id="bf59e-167">De forma predeterminada, el servicio de administración Web escucha en el puerto TCP 8172.</span><span class="sxs-lookup"><span data-stu-id="bf59e-167">By default, the Web Management Service listens on TCP port 8172.</span></span> <span data-ttu-id="bf59e-168">En Firewall de Windows, cree una nueva regla de entrada para permitir el tráfico TCP en el puerto 8172.</span><span class="sxs-lookup"><span data-stu-id="bf59e-168">In Windows Firewall, create a new inbound rule to allow TCP traffic on port 8172.</span></span> <span data-ttu-id="bf59e-169">Para obtener más información, consulte [configuración de reglas de Firewall](https://technet.microsoft.com/en-us/library/dd448559(WS.10).aspx).</span><span class="sxs-lookup"><span data-stu-id="bf59e-169">For more information, see [Configuring Firewall Rules](https://technet.microsoft.com/en-us/library/dd448559(WS.10).aspx).</span></span> <span data-ttu-id="bf59e-170">(Si hospeda las máquinas virtuales en Azure, puede hacerlo directamente en el portal de Azure.</span><span class="sxs-lookup"><span data-stu-id="bf59e-170">(If you are hosting the VMs on Azure, you can do this directly in the Azure portal.</span></span> <span data-ttu-id="bf59e-171">Vea [cómo configurar extremos en una máquina Virtual](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-set-up-endpoints/).)</span><span class="sxs-lookup"><span data-stu-id="bf59e-171">See [How to Set Up Endpoints to a Virtual Machine](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-set-up-endpoints/).)</span></span>

<span data-ttu-id="bf59e-172">Ahora está listo para implementar el proyecto de Visual Studio desde el equipo de desarrollo en el servidor.</span><span class="sxs-lookup"><span data-stu-id="bf59e-172">Now you are ready to deploy the Visual Studio project from your development machine to the server.</span></span> <span data-ttu-id="bf59e-173">En el Explorador de soluciones, haga clic en la solución y haga clic en **publicar**.</span><span class="sxs-lookup"><span data-stu-id="bf59e-173">In Solution Explorer, right-click the solution and click **Publish**.</span></span>

<span data-ttu-id="bf59e-174">Para obtener información más detallada acerca de la implementación de web, consulte [mapa de contenido de implementación Web para Visual Studio y ASP.NET](../../../whitepapers/aspnet-web-deployment-content-map.md).</span><span class="sxs-lookup"><span data-stu-id="bf59e-174">For more detailed documentation about web deployment, see [Web Deployment Content Map for Visual Studio and ASP.NET](../../../whitepapers/aspnet-web-deployment-content-map.md).</span></span>

<span data-ttu-id="bf59e-175">Si implementa la aplicación en dos servidores, puede abrir cada instancia en otra ventana del explorador y vea que reciban mensajes de SignalR desde la otra.</span><span class="sxs-lookup"><span data-stu-id="bf59e-175">If you deploy the application to two servers, you can open each instance in a separate browser window and see that they each receive SignalR messages from the other.</span></span> <span data-ttu-id="bf59e-176">(Por supuesto, en un entorno de producción, los dos servidores se colocan detrás de un equilibrador de carga.)</span><span class="sxs-lookup"><span data-stu-id="bf59e-176">(Of course, in a production environment, the two servers would sit behind a load balancer.)</span></span>

![](scaleout-with-redis/_static/image8.png)

<span data-ttu-id="bf59e-177">Si tiene curiosidad ver los mensajes que se envían a Redis, puede usar el **redis cli** cliente, que se instala con Redis.</span><span class="sxs-lookup"><span data-stu-id="bf59e-177">If you're curious to see the messages that are sent to Redis, you can use the **redis-cli** client, which installs with Redis.</span></span>

[!code-xml[Main](scaleout-with-redis/samples/sample8.xml)]

![](scaleout-with-redis/_static/image9.png)