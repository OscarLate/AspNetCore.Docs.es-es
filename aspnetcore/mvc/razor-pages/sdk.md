---
title: SDK de Razor de ASP.NET Core
author: Rick-Anderson
description: Obtenga información sobre cómo las páginas de Razor de ASP.NET Core facilitan la programación de escenarios centrados en páginas y hacen que resulte más productiva que con MVC.
manager: wpickett
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 4/12/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: content
uid: mvc/razor-pages/sdk
ms.openlocfilehash: 2cbebb12ccd1098e1950aa7eeb22fab4ffc689e6
ms.sourcegitcommit: c79fd3592f444d58e17518914f8873d0a11219c0
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2018
---
# <a name="aspnet-core-razor-sdk"></a><span data-ttu-id="87126-103">SDK de Razor de ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="87126-103">ASP.NET Core Razor SDK</span></span>

<span data-ttu-id="87126-104">Por [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="87126-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

[!INCLUDE[](~/includes/2.1.md)]

<span data-ttu-id="87126-105">El [!INCLUDE [](~/includes/2.1-SDK.md)] incluye el SDK de MSBuild `Microsoft.NET.Sdk.Razor` (SDK de Razor).</span><span class="sxs-lookup"><span data-stu-id="87126-105">The [!INCLUDE[](~/includes/2.1-SDK.md)] includes the `Microsoft.NET.Sdk.Razor` MSBuild SDK (Razor SDK).</span></span> <span data-ttu-id="87126-106">El SDK de Razor:</span><span class="sxs-lookup"><span data-stu-id="87126-106">The Razor SDK:</span></span>

* <span data-ttu-id="87126-107">Normaliza la experiencia de creación, empaquetado y publicación de proyectos que contienen archivos de [Razor](xref:mvc/views/razor) relativos a proyectos basados en ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="87126-107">Standardizes the experience around building, packaging, and publishing projects containing [Razor](xref:mvc/views/razor) files for ASP.NET Core MVC-based projects.</span></span>
* <span data-ttu-id="87126-108">Incluye un conjunto de destinos, propiedades y elementos predefinidos que permiten personalizar la compilación de archivos de Razor.</span><span class="sxs-lookup"><span data-stu-id="87126-108">Includes a set of predefined targets, properties, and items that allow customizing the compilation of Razor files.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="87126-109">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="87126-109">Prerequisites</span></span>

[!INCLUDE[](~/includes/2.1-SDK.md)]

## <a name="using-the-razor-sdk"></a><span data-ttu-id="87126-110">Uso del SDK de Razor</span><span class="sxs-lookup"><span data-stu-id="87126-110">Using the Razor SDK</span></span>

<span data-ttu-id="87126-111">En la mayoría de las aplicaciones web no es necesario hacer una referencia expresa al SDK de Razor.</span><span class="sxs-lookup"><span data-stu-id="87126-111">Most web apps don't need to expressly reference the Razor SDK.</span></span> 

<span data-ttu-id="87126-112">Para usar el SDK de Razor para crear bibliotecas de clases que contengan vistas de Razor o páginas de Razor:</span><span class="sxs-lookup"><span data-stu-id="87126-112">To use the Razor SDK to build class libraries containing Razor views or Razor Pages:</span></span>

* <span data-ttu-id="87126-113">Use `Microsoft.NET.Sdk.Razor` en lugar de `Microsoft.NET.Sdk`:</span><span class="sxs-lookup"><span data-stu-id="87126-113">Use `Microsoft.NET.Sdk.Razor` instead of `Microsoft.NET.Sdk`:</span></span>
```xml
<Project SDK="Microsoft.NET.Sdk.Razor">
  ...
</Project>
```

* <span data-ttu-id="87126-114">Las referencias de paquete a `Microsoft.AspNetCore.Mvc` suelen ser necesarias para incluir más dependencias que son necesarias para generar y compilar páginas de Razor y vistas de Razor.</span><span class="sxs-lookup"><span data-stu-id="87126-114">Typically a package reference to `Microsoft.AspNetCore.Mvc` is required to bring in additional dependencies required to build and compile Razor Pages and Razor views.</span></span> <span data-ttu-id="87126-115">Como mínimo, el proyecto debe tener agregadas referencias de paquete a:</span><span class="sxs-lookup"><span data-stu-id="87126-115">At minimum, your project needs to add package references to:</span></span>

    * `Microsoft.AspNetCore.Razor.Design` 
    * `Microsoft.AspNetCore.Mvc.Razor.Extensions`
    
 <span data-ttu-id="87126-116">Los paquetes anteriores están incluidos en `Microsoft.AspNetCore.Mvc`.</span><span class="sxs-lookup"><span data-stu-id="87126-116">The preceding packages are included in `Microsoft.AspNetCore.Mvc`.</span></span> <span data-ttu-id="87126-117">En el siguiente marcado se muestra un archivo *.csproj* básico que usa el SDK de Razor para crear archivos de Razor para una aplicación de páginas de Razor de ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="87126-117">The following markup shows a basic *.csproj* file that uses the Razor SDK to build Razor files for an ASP.NET Core Razor Pages app:</span></span>
    
 [!code-xml[Main](sdk/sample/RazorSDK.csproj)]

### <a name="properties"></a><span data-ttu-id="87126-118">Propiedades</span><span class="sxs-lookup"><span data-stu-id="87126-118">Properties</span></span>

<span data-ttu-id="87126-119">Las siguientes propiedades controlan el comportamiento del SDK de Razor como parte de la generación de un proyecto:</span><span class="sxs-lookup"><span data-stu-id="87126-119">The following properties control the Razor's SDK behavior as part of a project build:</span></span>

* <span data-ttu-id="87126-120">`RazorCompileOnBuild`: si es `true`, compila y emite el ensamblado de Razor como parte de la generación del proyecto.</span><span class="sxs-lookup"><span data-stu-id="87126-120">`RazorCompileOnBuild` : When `true`, compiles and emits the Razor assembly as part of building the project.</span></span> <span data-ttu-id="87126-121">Tiene como valor predeterminado `true`.</span><span class="sxs-lookup"><span data-stu-id="87126-121">Defaults to `true`.</span></span>
* <span data-ttu-id="87126-122">`RazorCompileOnPublish`: si es `true`, compila y emite el ensamblado de Razor como parte de la publicación del proyecto.</span><span class="sxs-lookup"><span data-stu-id="87126-122">`RazorCompileOnPublish` : When `true`, compiles and emits the Razor assembly as part of publishing the project.</span></span> <span data-ttu-id="87126-123">Tiene como valor predeterminado `true`.</span><span class="sxs-lookup"><span data-stu-id="87126-123">Defaults to `true`.</span></span>

<span data-ttu-id="87126-124">Para configurar las entradas y la salida del SDK de Razor se usan las siguientes propiedades y elementos:</span><span class="sxs-lookup"><span data-stu-id="87126-124">The following properties and items are used to configure inputs and output to the Razor SDK:</span></span>

| <span data-ttu-id="87126-125">Elementos</span><span class="sxs-lookup"><span data-stu-id="87126-125">Items</span></span>                                         | <span data-ttu-id="87126-126">Description</span><span class="sxs-lookup"><span data-stu-id="87126-126">Description</span></span>                                                                   |
| ------------                                  | -------------                                                                 |
| <span data-ttu-id="87126-127">RazorGenerate</span><span class="sxs-lookup"><span data-stu-id="87126-127">RazorGenerate</span></span>                                 | <span data-ttu-id="87126-128">Elementos (archivos *.cshtml*) que son entradas para los destinos de generación de código.</span><span class="sxs-lookup"><span data-stu-id="87126-128">Item elements (*.cshtml* files) that are inputs to code generation targets.</span></span> |
| <span data-ttu-id="87126-129">RazorCompile</span><span class="sxs-lookup"><span data-stu-id="87126-129">RazorCompile</span></span>                                  | <span data-ttu-id="87126-130">Elementos (archivos .cs) que son entradas para los destinos de compilación de Razor.</span><span class="sxs-lookup"><span data-stu-id="87126-130">Item elements (.cs files) that are inputs to  Razor compilation targets.</span></span> <span data-ttu-id="87126-131">Use este ItemGroup para especificar más archivos para que se compilen en el ensamblado de Razor.</span><span class="sxs-lookup"><span data-stu-id="87126-131">Use this ItemGroup to specify additional files to be compiled in to the Razor assembly.</span></span> |
| <span data-ttu-id="87126-132">RazorAssemblyAttribute</span><span class="sxs-lookup"><span data-stu-id="87126-132">RazorAssemblyAttribute</span></span>                        | <span data-ttu-id="87126-133">Elementos que se usan para generar atributos de código para el ensamblado de Razor.</span><span class="sxs-lookup"><span data-stu-id="87126-133">Item elements used to code generate attributes for the Razor assembly.</span></span> <span data-ttu-id="87126-134">Por ejemplo:</span><span class="sxs-lookup"><span data-stu-id="87126-134">For example:</span></span>  <br />`<RazorAssemblyAttribute ` <br />  `Include="System.Reflection.AssemblyMetadataAttribute"`<br />`  _Parameter1="BuildSource" _Parameter2="https://docs.asp.net/">` |
| <span data-ttu-id="87126-135">RazorEmbeddedResource</span><span class="sxs-lookup"><span data-stu-id="87126-135">RazorEmbeddedResource</span></span>                         | <span data-ttu-id="87126-136">Elementos que se agregan como recursos insertados en el ensamblado de Razor generado.</span><span class="sxs-lookup"><span data-stu-id="87126-136">Item elements added as embedded resources to the generated Razor assembly</span></span> |

| <span data-ttu-id="87126-137">Property</span><span class="sxs-lookup"><span data-stu-id="87126-137">Property</span></span>                                      | <span data-ttu-id="87126-138">Description</span><span class="sxs-lookup"><span data-stu-id="87126-138">Description</span></span>                                                                   |
| ------------                                  | -------------                                                                 |
| <span data-ttu-id="87126-139">RazorTargetName</span><span class="sxs-lookup"><span data-stu-id="87126-139">RazorTargetName</span></span>                               | <span data-ttu-id="87126-140">Nombre de archivo (sin extensión) del ensamblado generado por Razor.</span><span class="sxs-lookup"><span data-stu-id="87126-140">File name (without extension) of the assembly produced by Razor.</span></span> | 
| <span data-ttu-id="87126-141">RazorOutputPath</span><span class="sxs-lookup"><span data-stu-id="87126-141">RazorOutputPath</span></span>                               | <span data-ttu-id="87126-142">Directorio de salida de Razor.</span><span class="sxs-lookup"><span data-stu-id="87126-142">The Razor output directory.</span></span>                                      |
| <span data-ttu-id="87126-143">RazorCompileToolset</span><span class="sxs-lookup"><span data-stu-id="87126-143">RazorCompileToolset</span></span>                           | <span data-ttu-id="87126-144">Sirve para determinar el conjunto de herramientas que se va a usar para compilar el ensamblado de Razor.</span><span class="sxs-lookup"><span data-stu-id="87126-144">Used to determine the toolset used to build the Razor assembly.</span></span> <span data-ttu-id="87126-145">Los valores válidos son `Implicit` y `PrecompilationTool`.</span><span class="sxs-lookup"><span data-stu-id="87126-145">Valid values are `Implicit`, , and `PrecompilationTool`.</span></span> |
| <span data-ttu-id="87126-146">EnableDefaultContentItems</span><span class="sxs-lookup"><span data-stu-id="87126-146">EnableDefaultContentItems</span></span>                     | <span data-ttu-id="87126-147">Si es `true`, incluye ciertos tipos de archivo (como archivos *.cshtml*) como contenido en el proyecto.</span><span class="sxs-lookup"><span data-stu-id="87126-147">When `true`, includes certain file types, such as *.cshtml* files, as content in the project.</span></span> <span data-ttu-id="87126-148">Si la referencia se hace a través de Microsoft.NET.Sdk.Web, también se incluyen todos los archivos de *wwwroot* y los archivos de configuración.</span><span class="sxs-lookup"><span data-stu-id="87126-148">When referenced via Microsoft.NET.Sdk.Web, also includes all files under *wwwroot*, and config files.</span></span>         |
| <span data-ttu-id="87126-149">EnableDefaultRazorGenerateItems</span><span class="sxs-lookup"><span data-stu-id="87126-149">EnableDefaultRazorGenerateItems</span></span>               | <span data-ttu-id="87126-150">Si es `true`, incluye los archivos *.cshtml* de los elementos `Content` en elementos `RazorGenerate`.</span><span class="sxs-lookup"><span data-stu-id="87126-150">When `true`, includes *.cshtml* files from `Content` items in `RazorGenerate` items.</span></span> |
| <span data-ttu-id="87126-151">GenerateRazorTargetAssemblyInfo</span><span class="sxs-lookup"><span data-stu-id="87126-151">GenerateRazorTargetAssemblyInfo</span></span>               | <span data-ttu-id="87126-152">Si es `true`, genera un archivo *.cs* que contiene los atributos especificados por `RazorAssemblyAttribute` y este se incluye en la salida de compilación.</span><span class="sxs-lookup"><span data-stu-id="87126-152">When `true`, generates a *.cs* file containing attributes specified by `RazorAssemblyAttribute` and includes it in the compile output.</span></span> |
| <span data-ttu-id="87126-153">EnableDefaultRazorTargetAssemblyInfoAttributes</span><span class="sxs-lookup"><span data-stu-id="87126-153">EnableDefaultRazorTargetAssemblyInfoAttributes</span></span> | <span data-ttu-id="87126-154">Si es `true`, agrega a `RazorAssemblyAttribute` un conjunto predeterminado de atributos de ensamblado.</span><span class="sxs-lookup"><span data-stu-id="87126-154">When `true`, adds a default set of assembly attributes to `RazorAssemblyAttribute`.</span></span> |
| <span data-ttu-id="87126-155">CopyRazorGenerateFilesToPublishDirectory</span><span class="sxs-lookup"><span data-stu-id="87126-155">CopyRazorGenerateFilesToPublishDirectory</span></span>       | <span data-ttu-id="87126-156">Si es `true`, copia los elementos de RazorGenerate (archivos *.cshtml*) en el directorio de publicación.</span><span class="sxs-lookup"><span data-stu-id="87126-156">When `true`, copies RazorGenerate items (*.cshtml*) files to the publish directory.</span></span> <span data-ttu-id="87126-157">Los archivos de Razor no suelen ser necesarios en una aplicación publicada si forman parte de la compilación en tiempo de compilación o de publicación.</span><span class="sxs-lookup"><span data-stu-id="87126-157">Typically Razor files are not needed for a published application if they participate in compilation at build-time or publish-time.</span></span> <span data-ttu-id="87126-158">Tiene como valor predeterminado `false`.</span><span class="sxs-lookup"><span data-stu-id="87126-158">Defaults to `false`.</span></span> |
| <span data-ttu-id="87126-159">CopyRefAssembliesToPublishDirectory</span><span class="sxs-lookup"><span data-stu-id="87126-159">CopyRefAssembliesToPublishDirectory</span></span>            | <span data-ttu-id="87126-160">Si es `true`, copia los elementos de referencia del ensamblado en el directorio de publicación.</span><span class="sxs-lookup"><span data-stu-id="87126-160">When `true`, copy reference assembly items to the publish directory.</span></span> <span data-ttu-id="87126-161">Los ensamblados de referencia no suelen ser necesarios en una aplicación publicada si la compilación de Razor tiene lugar en tiempo de compilación o de publicación.</span><span class="sxs-lookup"><span data-stu-id="87126-161">Typically reference assemblies are not needed for a published application if Razor compilation occurs at build-time or publish-time.</span></span> <span data-ttu-id="87126-162">Establézcala en `true` si la aplicación publicada requiere compilación en tiempo de ejecución, por ejemplo, si modifica archivos cshtml en tiempo de ejecución o si usa vistas incrustadas.</span><span class="sxs-lookup"><span data-stu-id="87126-162">Set to `true`, if your published application requires runtime compilation, for example, modifies cshtml files at runtime, or uses embedded views.</span></span> <span data-ttu-id="87126-163">Tiene como valor predeterminado `false`.</span><span class="sxs-lookup"><span data-stu-id="87126-163">Defaults to `false`.</span></span> |
| <span data-ttu-id="87126-164">IncludeRazorContentInPack</span><span class="sxs-lookup"><span data-stu-id="87126-164">IncludeRazorContentInPack</span></span>                      | <span data-ttu-id="87126-165">Si es `true`, todos los elementos de contenido de Razor (archivos *.cshtml*) se marcarán para incluirse en el paquete NuGet generado.</span><span class="sxs-lookup"><span data-stu-id="87126-165">When `true`, all Razor content items (*.cshtml* files) will be marked for inclusion in the generated NuGet package.</span></span> <span data-ttu-id="87126-166">Tiene como valor predeterminado `false`.</span><span class="sxs-lookup"><span data-stu-id="87126-166">Defaults to `false`.</span></span> |
| <span data-ttu-id="87126-167">EmbedRazorGenerateSources</span><span class="sxs-lookup"><span data-stu-id="87126-167">EmbedRazorGenerateSources</span></span> | <span data-ttu-id="87126-168">Si es `true`, agrega los elementos de RazorGenerate (*.cshtml*) como archivos incrustados al ensamblado de Razor generado.</span><span class="sxs-lookup"><span data-stu-id="87126-168">When `true`, adds RazorGenerate (*.cshtml*) items as embedded files to the generated Razor assembly.</span></span> <span data-ttu-id="87126-169">Tiene como valor predeterminado `false`.</span><span class="sxs-lookup"><span data-stu-id="87126-169">Defaults to `false`.</span></span> |
| <span data-ttu-id="87126-170">UseRazorBuildServer</span><span class="sxs-lookup"><span data-stu-id="87126-170">UseRazorBuildServer</span></span>                           | <span data-ttu-id="87126-171">Si es `true`, usa un proceso de servidor de compilación persistente para descargar el trabajo de generación de código.</span><span class="sxs-lookup"><span data-stu-id="87126-171">When `true`, uses a persistent build server process to offload code generation work.</span></span> <span data-ttu-id="87126-172">El valor predeterminado es `UseSharedCompilation`.</span><span class="sxs-lookup"><span data-stu-id="87126-172">Defaults to the value of `UseSharedCompilation`.</span></span> |

### <a name="targets"></a><span data-ttu-id="87126-173">Destinos</span><span class="sxs-lookup"><span data-stu-id="87126-173">Targets</span></span>
<span data-ttu-id="87126-174">El SDK de Razor establece dos objetivos principales:</span><span class="sxs-lookup"><span data-stu-id="87126-174">The Razor SDK defines two primary targets:</span></span>

* <span data-ttu-id="87126-175">`RazorGenerate`: el código genera archivos *.cs* a partir de elementos de RazorGenerate.</span><span class="sxs-lookup"><span data-stu-id="87126-175">`RazorGenerate` - Code generates *.cs* files from RazorGenerate item elements.</span></span> <span data-ttu-id="87126-176">Use la propiedad `RazorGenerateDependsOn` para especificar más destinos que puedan ejecutarse antes o después de este destino.</span><span class="sxs-lookup"><span data-stu-id="87126-176">Use `RazorGenerateDependsOn` property to specify additional targets that can run before or after this target.</span></span>
* <span data-ttu-id="87126-177">`RazorCompile`: compila los archivos *.cs* generados en un ensamblado de Razor.</span><span class="sxs-lookup"><span data-stu-id="87126-177">`RazorCompile` - Compiles generated *.cs* files in to a Razor assembly.</span></span> <span data-ttu-id="87126-178">Use `RazorCompileDependsOn` para especificar más destinos que puedan ejecutarse antes o después de este destino.</span><span class="sxs-lookup"><span data-stu-id="87126-178">Use `RazorCompileDependsOn` to specify additional targets that can run before or after this target.</span></span>