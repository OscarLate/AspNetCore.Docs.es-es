---
title: "Páginas de Razor con EF Core - Migrations - 4 de 8"
author: rick-anderson
description: "En este tutorial, debe comenzar con la característica de migraciones de EF principal para administrar los cambios de modelo de datos en una aplicación de MVC de ASP.NET Core."
ms.author: riande
manager: wpickett
ms.date: 10/15/2017
ms.topic: get-started-article
ms.technology: aspnet
ms.prod: asp.net-core
uid: data/ef-rp/migrations
ms.openlocfilehash: 9a0fb52a1d1a62bce3f11c7e0394c00b9d544ab3
ms.sourcegitcommit: 3d512ea991ac36dfd4c800b7d1f8a27bfc50635e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/23/2018
---
# <a name="migrations---ef-core-with-razor-pages-tutorial-4-of-8"></a><span data-ttu-id="fa81f-103">Migraciones - Core EF con el tutorial de las páginas de Razor (4 de 8)</span><span class="sxs-lookup"><span data-stu-id="fa81f-103">Migrations - EF Core with Razor Pages tutorial (4 of 8)</span></span>

<span data-ttu-id="fa81f-104">Por [Tom Dykstra](https://github.com/tdykstra), [Jon P Smith](https://twitter.com/thereformedprog), y [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="fa81f-104">By [Tom Dykstra](https://github.com/tdykstra), [Jon P Smith](https://twitter.com/thereformedprog), and [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

[!INCLUDE[about the series](../../includes/RP-EF/intro.md)]

<span data-ttu-id="fa81f-105">En este tutorial, se usa la característica de migraciones de EF principal para administrar los cambios del modelo de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-105">In this tutorial, the EF Core migrations feature for managing data model changes is used.</span></span>

<span data-ttu-id="fa81f-106">Si experimenta problemas no puede resolver, descargue el [aplicación final de esta fase](
https://github.com/aspnet/Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/StageSnapShots/cu-part4-migrations).</span><span class="sxs-lookup"><span data-stu-id="fa81f-106">If you run into problems you can't solve, download the [completed app for this stage](
https://github.com/aspnet/Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/StageSnapShots/cu-part4-migrations).</span></span>

<span data-ttu-id="fa81f-107">Cuando se desarrolla una aplicación nueva, el modelo de datos cambie con frecuencia.</span><span class="sxs-lookup"><span data-stu-id="fa81f-107">When a new app is developed, the data model changes frequently.</span></span> <span data-ttu-id="fa81f-108">Cada vez que cambia el modelo, el modelo obtiene sincronizado con la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-108">Each time the model changes, the model gets out of sync with the database.</span></span> <span data-ttu-id="fa81f-109">Este tutorial iniciado mediante la configuración de Entity Framework para crear la base de datos si no existe.</span><span class="sxs-lookup"><span data-stu-id="fa81f-109">This tutorial started by configuring the Entity Framework to create the database if it doesn't exist.</span></span> <span data-ttu-id="fa81f-110">Cada vez que los datos de cambios en el modelo:</span><span class="sxs-lookup"><span data-stu-id="fa81f-110">Each time the data model changes:</span></span>

* <span data-ttu-id="fa81f-111">Se quita la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-111">The DB is dropped.</span></span>
* <span data-ttu-id="fa81f-112">EF crea uno nuevo que coincide con el modelo.</span><span class="sxs-lookup"><span data-stu-id="fa81f-112">EF creates a new one that matches the model.</span></span>
* <span data-ttu-id="fa81f-113">La aplicación propaga la base de datos con datos de prueba.</span><span class="sxs-lookup"><span data-stu-id="fa81f-113">The app seeds the DB with test data.</span></span>

<span data-ttu-id="fa81f-114">Este enfoque para mantener la base de datos sincronizada con el modelo de datos funciona bien hasta que implemente la aplicación para producción.</span><span class="sxs-lookup"><span data-stu-id="fa81f-114">This approach to keeping the DB in sync with the data model works well until you deploy the app to production.</span></span> <span data-ttu-id="fa81f-115">Cuando se ejecuta la aplicación en producción, normalmente está almacenando datos que hay que mantenerlo.</span><span class="sxs-lookup"><span data-stu-id="fa81f-115">When the app is running in production, it is usually storing data that needs to be maintained.</span></span> <span data-ttu-id="fa81f-116">No se puede iniciar la aplicación con una prueba de base de datos cada vez que se realiza un cambio (por ejemplo, agregar una nueva columna).</span><span class="sxs-lookup"><span data-stu-id="fa81f-116">The app can't start with a test DB each time a change is made (such as adding a new column).</span></span> <span data-ttu-id="fa81f-117">La característica de EF Core migraciones soluciona este problema habilitando el núcleo de EF para actualizar el esquema de base de datos en lugar de crear una nueva base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-117">The EF Core Migrations feature solves this problem by enabling EF Core to update the DB schema instead of creating a new DB.</span></span>

<span data-ttu-id="fa81f-118">En lugar de quitar y volver a crear la base de datos cuando los datos de cambios en el modelo, las migraciones actualiza el esquema y conserva los datos existentes.</span><span class="sxs-lookup"><span data-stu-id="fa81f-118">Rather than dropping and recreating the DB when the data model changes, migrations updates the schema and retains existing data.</span></span>

## <a name="entity-framework-core-nuget-packages-for-migrations"></a><span data-ttu-id="fa81f-119">Paquetes de Entity Framework Core NuGet para migraciones</span><span class="sxs-lookup"><span data-stu-id="fa81f-119">Entity Framework Core NuGet packages for migrations</span></span>

<span data-ttu-id="fa81f-120">Para trabajar con las migraciones, utilice la **Package Manager Console** (PMC) o la interfaz de línea de comandos (CLI).</span><span class="sxs-lookup"><span data-stu-id="fa81f-120">To work with migrations, use the **Package Manager Console** (PMC) or the command-line interface (CLI).</span></span> <span data-ttu-id="fa81f-121">Estos tutoriales muestra cómo utilizar los comandos CLI.</span><span class="sxs-lookup"><span data-stu-id="fa81f-121">These tutorials show how to use CLI commands.</span></span> <span data-ttu-id="fa81f-122">Obtener información sobre el PMC está en [al final de este tutorial](#pmc).</span><span class="sxs-lookup"><span data-stu-id="fa81f-122">Information about the PMC is at [the end of this tutorial](#pmc).</span></span>

<span data-ttu-id="fa81f-123">Las herramientas principales de EF de la interfaz de línea de comandos (CLI) se proporcionan en [Microsoft.EntityFrameworkCore.Tools.DotNet](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools.DotNet).</span><span class="sxs-lookup"><span data-stu-id="fa81f-123">The EF Core tools for the command-line interface (CLI) are provided in [Microsoft.EntityFrameworkCore.Tools.DotNet](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools.DotNet).</span></span> <span data-ttu-id="fa81f-124">Para instalar este paquete, agréguelo a la `DotNetCliToolReference` colección en la *.csproj* de archivos, como se muestra.</span><span class="sxs-lookup"><span data-stu-id="fa81f-124">To install this package, add it to the `DotNetCliToolReference` collection in the *.csproj* file, as shown.</span></span> <span data-ttu-id="fa81f-125">**Nota:** este paquete debe instalarse mediante la edición de la *.csproj* archivo.</span><span class="sxs-lookup"><span data-stu-id="fa81f-125">**Note:** This package must be installed by editing the *.csproj* file.</span></span> <span data-ttu-id="fa81f-126">El`install-package` comando o la interfaz gráfica de usuario del Administrador de paquetes no se puede usar para instalar este paquete.</span><span class="sxs-lookup"><span data-stu-id="fa81f-126">The`install-package` command or the package manager GUI cannot be used to install this package.</span></span> <span data-ttu-id="fa81f-127">Editar la *.csproj* archivo haciendo clic en el nombre del proyecto en **el Explorador de soluciones** y seleccionando **ContosoUniversity.csproj editar**.</span><span class="sxs-lookup"><span data-stu-id="fa81f-127">Edit the *.csproj* file by right-clicking the project name in **Solution Explorer** and selecting **Edit ContosoUniversity.csproj**.</span></span>

<span data-ttu-id="fa81f-128">El marcado siguiente muestra la actualización *.csproj* archivo con las herramientas de EF Core CLI resaltado:</span><span class="sxs-lookup"><span data-stu-id="fa81f-128">The following markup shows the updated *.csproj* file with the EF Core CLI tools highlighted:</span></span>

[!code-xml[](intro/samples/cu/ContosoUniversity.csproj?highlight=12)]
  
<span data-ttu-id="fa81f-129">Los números de versión en el ejemplo anterior eran actuales cuando se escribió el tutorial.</span><span class="sxs-lookup"><span data-stu-id="fa81f-129">The version numbers in the preceding example were current when the tutorial was written.</span></span> <span data-ttu-id="fa81f-130">Utilice la misma versión para las herramientas de EF CLI de núcleo disponible en los otros paquetes.</span><span class="sxs-lookup"><span data-stu-id="fa81f-130">Use the same version for the EF Core CLI tools found in the other packages.</span></span>

## <a name="change-the-connection-string"></a><span data-ttu-id="fa81f-131">Cambiar la cadena de conexión</span><span class="sxs-lookup"><span data-stu-id="fa81f-131">Change the connection string</span></span>

<span data-ttu-id="fa81f-132">En el *appSettings.JSON que se* de archivo, cambie el nombre de la base de datos en la cadena de conexión a ContosoUniversity2.</span><span class="sxs-lookup"><span data-stu-id="fa81f-132">In the *appsettings.json* file, change the name of the DB in the connection string to ContosoUniversity2.</span></span>

[!code-json[Main](intro/samples/cu/appsettings2.json?range=1-4)]

<span data-ttu-id="fa81f-133">Cambiar el nombre de la base de datos en la cadena de conexión hace que la primera migración crear una nueva base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-133">Changing the DB name in the connection string causes the first migration to create a new DB.</span></span> <span data-ttu-id="fa81f-134">Se crea una nueva base de datos porque no existe uno con ese nombre.</span><span class="sxs-lookup"><span data-stu-id="fa81f-134">A new DB is created because one with that name does not exist.</span></span> <span data-ttu-id="fa81f-135">Cambiar la cadena de conexión no es necesario para comenzar a usar migraciones.</span><span class="sxs-lookup"><span data-stu-id="fa81f-135">Changing the connection string isn't required for getting started with migrations.</span></span>

<span data-ttu-id="fa81f-136">Una alternativa a cambiar el nombre de la base de datos está eliminando la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-136">An alternative to changing the DB name is deleting the DB.</span></span> <span data-ttu-id="fa81f-137">Use **Explorador de objetos de SQL Server** (SSOX) o `database drop` comando de CLI:</span><span class="sxs-lookup"><span data-stu-id="fa81f-137">Use **SQL Server Object Explorer** (SSOX) or the `database drop` CLI command:</span></span>

 ```console
 dotnet ef database drop
 ```

<span data-ttu-id="fa81f-138">En la siguiente sección se explica cómo ejecutar comandos de CLI.</span><span class="sxs-lookup"><span data-stu-id="fa81f-138">The following section explains how to run CLI commands.</span></span>

## <a name="create-an-initial-migration"></a><span data-ttu-id="fa81f-139">Crear una migración inicial</span><span class="sxs-lookup"><span data-stu-id="fa81f-139">Create an initial migration</span></span>

<span data-ttu-id="fa81f-140">Compile el proyecto.</span><span class="sxs-lookup"><span data-stu-id="fa81f-140">Build the project.</span></span>

<span data-ttu-id="fa81f-141">Abra una ventana de comandos y navegue hasta la carpeta del proyecto.</span><span class="sxs-lookup"><span data-stu-id="fa81f-141">Open a command window and navigate to the project folder.</span></span> <span data-ttu-id="fa81f-142">La carpeta del proyecto contiene el *Startup.cs* archivo.</span><span class="sxs-lookup"><span data-stu-id="fa81f-142">The project folder contains the *Startup.cs* file.</span></span>

<span data-ttu-id="fa81f-143">En la ventana de comandos, escriba lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="fa81f-143">Enter the following in the command window:</span></span>

```console
dotnet ef migrations add InitialCreate
```

<span data-ttu-id="fa81f-144">La ventana de comandos muestra información similar al siguiente:</span><span class="sxs-lookup"><span data-stu-id="fa81f-144">The command window displays information similar to the following:</span></span>

```console
info: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[0]
      User profile is available. Using 'C:\Users\username\AppData\Local\ASP.NET\DataProtection-Keys' as key repository and Windows DPAPI to encrypt keys at rest.
info: Microsoft.EntityFrameworkCore.Infrastructure[100403]
      Entity Framework Core 2.0.0-rtm-26452 initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
Done. To undo this action, use 'ef migrations remove'
```

<span data-ttu-id="fa81f-145">Si se produce un error en la migración con el mensaje "*no se puede obtener acceso al archivo... ContosoUniversity.dll porque está siendo utilizado por otro proceso.* "</span><span class="sxs-lookup"><span data-stu-id="fa81f-145">If the migration fails with the message "*cannot access the file ... ContosoUniversity.dll because it is being used by another process.*"</span></span> <span data-ttu-id="fa81f-146">se muestra:</span><span class="sxs-lookup"><span data-stu-id="fa81f-146">is displayed:</span></span>

* <span data-ttu-id="fa81f-147">Detener IIS Express.</span><span class="sxs-lookup"><span data-stu-id="fa81f-147">Stop IIS Express.</span></span>

   * <span data-ttu-id="fa81f-148">Salga y reinicie Visual Studio, o</span><span class="sxs-lookup"><span data-stu-id="fa81f-148">Exit and restart Visual Studio, or</span></span>
   * <span data-ttu-id="fa81f-149">El icono de IIS Express se encuentra en la bandeja del sistema de Windows.</span><span class="sxs-lookup"><span data-stu-id="fa81f-149">Find the IIS Express icon in the Windows System Tray.</span></span>
   * <span data-ttu-id="fa81f-150">Haga clic en el icono de IIS Express y, a continuación, haga clic en **ContosoUniversity > Detener sitio**.</span><span class="sxs-lookup"><span data-stu-id="fa81f-150">Right-click the IIS Express icon, and then click **ContosoUniversity > Stop Site**.</span></span>

<span data-ttu-id="fa81f-151">Si el mensaje de error "" Error al crear.</span><span class="sxs-lookup"><span data-stu-id="fa81f-151">If the error message "Build failed."</span></span> <span data-ttu-id="fa81f-152">se muestra, vuelva a ejecutar el comando.</span><span class="sxs-lookup"><span data-stu-id="fa81f-152">is displayed, run the command again.</span></span> <span data-ttu-id="fa81f-153">Si obtiene este error, deje una nota en la parte inferior de este tutorial.</span><span class="sxs-lookup"><span data-stu-id="fa81f-153">If you get this error, leave a note at the bottom of this tutorial.</span></span>

### <a name="examine-the-up-and-down-methods"></a><span data-ttu-id="fa81f-154">Examine el arriba y abajo métodos</span><span class="sxs-lookup"><span data-stu-id="fa81f-154">Examine the Up and Down methods</span></span>

<span data-ttu-id="fa81f-155">El comando EF Core `migrations add` genera código para crear la base de datos de.</span><span class="sxs-lookup"><span data-stu-id="fa81f-155">The EF Core command `migrations add` generated code to create the DB from.</span></span> <span data-ttu-id="fa81f-156">Este código de migraciones se encuentra en la *migraciones\<timestamp > _InitialCreate.cs* archivo.</span><span class="sxs-lookup"><span data-stu-id="fa81f-156">This migrations code is in the *Migrations\<timestamp>_InitialCreate.cs* file.</span></span> <span data-ttu-id="fa81f-157">El `Up` método de la `InitialCreate` clase crea las tablas de base de datos que corresponden a los conjuntos de entidades del modelo de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-157">The `Up` method of the `InitialCreate` class creates the DB tables that correspond to the data model entity sets.</span></span> <span data-ttu-id="fa81f-158">El `Down` método elimina, tal como se muestra en el ejemplo siguiente:</span><span class="sxs-lookup"><span data-stu-id="fa81f-158">The `Down` method deletes them, as shown in the following example:</span></span>

[!code-csharp[Main](intro/samples/cu/Migrations/20171026010210_InitialCreate.cs?range=8-24,77-)]

<span data-ttu-id="fa81f-159">Llamadas de migraciones el `Up` método para implementar los cambios de modelo de datos para una migración.</span><span class="sxs-lookup"><span data-stu-id="fa81f-159">Migrations calls the `Up` method to implement the data model changes for a migration.</span></span> <span data-ttu-id="fa81f-160">Cuando se especifica un comando para revertir la actualización, llamadas de migraciones el `Down` método.</span><span class="sxs-lookup"><span data-stu-id="fa81f-160">When you enter a command to roll back the update, migrations calls the `Down` method.</span></span>

<span data-ttu-id="fa81f-161">El código anterior es para la migración inicial.</span><span class="sxs-lookup"><span data-stu-id="fa81f-161">The preceding code is for the initial migration.</span></span> <span data-ttu-id="fa81f-162">Ese código se creó cuando el `migrations add InitialCreate` se ejecutó el comando.</span><span class="sxs-lookup"><span data-stu-id="fa81f-162">That code was created when the `migrations add InitialCreate` command was run.</span></span> <span data-ttu-id="fa81f-163">El parámetro de nombre de migración ("InitialCreate" en el ejemplo) se utiliza para el nombre de archivo.</span><span class="sxs-lookup"><span data-stu-id="fa81f-163">The migration name parameter ("InitialCreate" in the example) is used for the file name.</span></span> <span data-ttu-id="fa81f-164">El nombre de la migración puede ser cualquier nombre de archivo válido.</span><span class="sxs-lookup"><span data-stu-id="fa81f-164">The migration name can be any valid file name.</span></span> <span data-ttu-id="fa81f-165">Es mejor elegir una palabra o frase que resume lo que se hace en la migración.</span><span class="sxs-lookup"><span data-stu-id="fa81f-165">It's best to choose a word or phrase that summarizes what is being done in the migration.</span></span> <span data-ttu-id="fa81f-166">Por ejemplo, una migración que se ha agregado una tabla de departamento podría denominarse "AddDepartmentTable."</span><span class="sxs-lookup"><span data-stu-id="fa81f-166">For example, a migration that added a department table might be called "AddDepartmentTable."</span></span>

<span data-ttu-id="fa81f-167">Si la migración inicial se crea y se cierra la base de datos:</span><span class="sxs-lookup"><span data-stu-id="fa81f-167">If the initial migration is created and the DB exits:</span></span>

* <span data-ttu-id="fa81f-168">Se genera el código de creación de la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-168">The DB creation code is generated.</span></span>
* <span data-ttu-id="fa81f-169">El código de creación de la base de datos no tiene que ejecutar porque la base de datos ya coincide con el modelo de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-169">The DB creation code doesn't need to run because the DB already matches the data model.</span></span> <span data-ttu-id="fa81f-170">Si se ejecuta el código de creación de la base de datos, no realiza ningún cambio porque la base de datos ya coincide con el modelo de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-170">If the The DB creation code is run, it doesn't make any changes because the DB already matches the data model.</span></span>

<span data-ttu-id="fa81f-171">Cuando la aplicación se implementa en un nuevo entorno, se debe ejecutar el código de creación de la base de datos para crear la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-171">When the app is deployed to a new environment, the DB creation code must be run to create the DB.</span></span>

<span data-ttu-id="fa81f-172">Anteriormente, la cadena de conexión se cambió para usar un nombre nuevo para la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-172">Previously the connection string was changed to use a new name for the DB.</span></span> <span data-ttu-id="fa81f-173">La base de datos especificado no existe, por lo que las migraciones crea la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-173">The specified DB doesn't exist, so migrations creates the DB.</span></span>

### <a name="examine-the-data-model-snapshot"></a><span data-ttu-id="fa81f-174">Examine la instantánea del modelo de datos</span><span class="sxs-lookup"><span data-stu-id="fa81f-174">Examine the data model snapshot</span></span>

<span data-ttu-id="fa81f-175">Migraciones crea un *instantánea* del esquema de base de datos actual en *Migrations/SchoolContextModelSnapshot.cs*:</span><span class="sxs-lookup"><span data-stu-id="fa81f-175">Migrations creates a *snapshot* of the current DB schema in *Migrations/SchoolContextModelSnapshot.cs*:</span></span>

[!code-csharp[Main](intro/samples/cu/Migrations/SchoolContextModelSnapshot1.cs?name=snippet_Truncate)]

<span data-ttu-id="fa81f-176">Como el esquema de base de datos actual se representa en el código, Core EF no tiene que interactuar con la base de datos para crear las migraciones.</span><span class="sxs-lookup"><span data-stu-id="fa81f-176">Because the current DB schema is represented in code, EF Core doesn't have to interact with the DB to create migrations.</span></span> <span data-ttu-id="fa81f-177">Cuando se agrega una migración, Core EF determina qué cambiado al comparar los modelos de datos para el archivo de instantánea.</span><span class="sxs-lookup"><span data-stu-id="fa81f-177">When you add a migration, EF Core determines what changed by comparing the data model to the snapshot file.</span></span> <span data-ttu-id="fa81f-178">Núcleo EF interactúa con la base de datos sólo cuando tiene que actualizar la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-178">EF Core interacts with the DB only when it has to update the DB.</span></span>

<span data-ttu-id="fa81f-179">El archivo de instantánea debe estar sincronizado con las migraciones que lo creó.</span><span class="sxs-lookup"><span data-stu-id="fa81f-179">The snapshot file must be in sync with the migrations that created it.</span></span> <span data-ttu-id="fa81f-180">No se puede quitar una migración eliminando el archivo denominado  *\<timestamp > _\<migrationname > .cs*.</span><span class="sxs-lookup"><span data-stu-id="fa81f-180">A migration can't be removed by deleting the file named *\<timestamp>_\<migrationname>.cs*.</span></span> <span data-ttu-id="fa81f-181">Si se elimina dicho archivo, las restantes migraciones están sincronizadas con el archivo de instantánea de base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-181">If that file is deleted, the remaining migrations are out of sync with the DB snapshot file.</span></span> <span data-ttu-id="fa81f-182">Para eliminar la última migración agregada, use la [migraciones de ef dotnet quitar](https://docs.microsoft.com/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove) comando.</span><span class="sxs-lookup"><span data-stu-id="fa81f-182">To delete the last migration added, use the [dotnet ef migrations remove](https://docs.microsoft.com/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove) command.</span></span>

## <a name="remove-ensurecreated"></a><span data-ttu-id="fa81f-183">Quitar EnsureCreated</span><span class="sxs-lookup"><span data-stu-id="fa81f-183">Remove EnsureCreated</span></span>

<span data-ttu-id="fa81f-184">Para el desarrollo inicial, el `EnsureCreated` se utilizó el comando.</span><span class="sxs-lookup"><span data-stu-id="fa81f-184">For early development, the `EnsureCreated` command was used.</span></span> <span data-ttu-id="fa81f-185">En este tutorial, se usa las migraciones.</span><span class="sxs-lookup"><span data-stu-id="fa81f-185">In this tutorial, migrations is used.</span></span> <span data-ttu-id="fa81f-186">`EnsureCreated`tiene las siguientes limitaciones:</span><span class="sxs-lookup"><span data-stu-id="fa81f-186">`EnsureCreated` has the following limitations:</span></span>

* <span data-ttu-id="fa81f-187">Omite las migraciones y crea la base de datos y el esquema.</span><span class="sxs-lookup"><span data-stu-id="fa81f-187">Bypasses migrations and creates the DB and schema.</span></span>
* <span data-ttu-id="fa81f-188">No se crea una tabla de las migraciones.</span><span class="sxs-lookup"><span data-stu-id="fa81f-188">Does not create a migrations table.</span></span>
* <span data-ttu-id="fa81f-189">Puede *no* utilizarse con las migraciones.</span><span class="sxs-lookup"><span data-stu-id="fa81f-189">Can *not* be used with migrations.</span></span>
* <span data-ttu-id="fa81f-190">Está diseñado para crear prototipos de prueba o rápido en la base de datos se quita y vuelve a crear con frecuencia.</span><span class="sxs-lookup"><span data-stu-id="fa81f-190">Is designed for testing or rapid prototyping where the DB is dropped and re-created frequently.</span></span>

<span data-ttu-id="fa81f-191">Quite la siguiente línea de `DbInitializer`:</span><span class="sxs-lookup"><span data-stu-id="fa81f-191">Remove the following line from `DbInitializer`:</span></span>

```csharp
context.Database.EnsureCreated();
```

## <a name="apply-the-migration-to-the-db-in-development"></a><span data-ttu-id="fa81f-192">La migración se aplican a la base de datos en desarrollo</span><span class="sxs-lookup"><span data-stu-id="fa81f-192">Apply the migration to the DB in development</span></span>

<span data-ttu-id="fa81f-193">En la ventana de comandos, escriba lo siguiente para crear la base de datos y tablas.</span><span class="sxs-lookup"><span data-stu-id="fa81f-193">In the command window, enter the following to create the DB and tables.</span></span>

```console
dotnet ef database update
```

<span data-ttu-id="fa81f-194">Nota: Si el `update` comando devuelve el error "Error al generar.":</span><span class="sxs-lookup"><span data-stu-id="fa81f-194">Note: If the `update` command returns the error "Build failed.":</span></span>

* <span data-ttu-id="fa81f-195">Vuelva a ejecutar el comando.</span><span class="sxs-lookup"><span data-stu-id="fa81f-195">Run the command again.</span></span>
* <span data-ttu-id="fa81f-196">Si se produce un error nuevo, salga de Visual Studio y, a continuación, ejecute el `update` comando.</span><span class="sxs-lookup"><span data-stu-id="fa81f-196">If it fails again, exit Visual Studio and then run the `update` command.</span></span>
* <span data-ttu-id="fa81f-197">Dejar un mensaje en la parte inferior de la página.</span><span class="sxs-lookup"><span data-stu-id="fa81f-197">Leave a message at the bottom of the page.</span></span>

<span data-ttu-id="fa81f-198">El resultado del comando es similar a la `migrations add` salida de comandos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-198">The output from the command is similar to the `migrations add` command output.</span></span> <span data-ttu-id="fa81f-199">En el comando anterior, se muestran los registros para los comandos SQL que configurar la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-199">In the preceding command, logs for the SQL commands that set up the DB are displayed.</span></span> <span data-ttu-id="fa81f-200">La mayoría de los registros se omite en la siguiente salida de ejemplo:</span><span class="sxs-lookup"><span data-stu-id="fa81f-200">Most of the logs are omitted in the following sample output:</span></span>

```text
info: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[0]
      User profile is available. Using 'C:\Users\username\AppData\Local\ASP.NET\DataProtection-Keys' as key repository and Windows DPAPI to encrypt keys at rest.
info: Microsoft.EntityFrameworkCore.Infrastructure[100403]
      Entity Framework Core 2.0.0-rtm-26452 initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
info: Microsoft.EntityFrameworkCore.Database.Command[200101]
      Executed DbCommand (467ms) [Parameters=[], CommandType='Text', CommandTimeout='60']
      CREATE DATABASE [ContosoUniversity2];
info: Microsoft.EntityFrameworkCore.Database.Command[200101]
      Executed DbCommand (20ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      CREATE TABLE [__EFMigrationsHistory] (
          [MigrationId] nvarchar(150) NOT NULL,
          [ProductVersion] nvarchar(32) NOT NULL,
          CONSTRAINT [PK___EFMigrationsHistory] PRIMARY KEY ([MigrationId])
      );

<logs omitted for brevity>

info: Microsoft.EntityFrameworkCore.Database.Command[200101]
      Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      INSERT INTO [__EFMigrationsHistory] ([MigrationId], [ProductVersion])
      VALUES (N'20170816151242_InitialCreate', N'2.0.0-rtm-26452');
Done.
```

<span data-ttu-id="fa81f-201">Para reducir el nivel de detalle en los mensajes de registro, puede cambiar los niveles de registro en el *appsettings. Development.JSON* archivo.</span><span class="sxs-lookup"><span data-stu-id="fa81f-201">To reduce the level of detail in log messages, can change the log levels in the *appsettings.Development.json* file.</span></span> <span data-ttu-id="fa81f-202">Para obtener más información, consulte [Introducción al registro](xref:fundamentals/logging/index).</span><span class="sxs-lookup"><span data-stu-id="fa81f-202">For more information, see [Introduction to logging](xref:fundamentals/logging/index).</span></span>

<span data-ttu-id="fa81f-203">Use **Explorador de objetos de SQL Server** para inspeccionar la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-203">Use **SQL Server Object Explorer** to inspect the DB.</span></span> <span data-ttu-id="fa81f-204">Tenga en cuenta la adición de un `__EFMigrationsHistory` tabla.</span><span class="sxs-lookup"><span data-stu-id="fa81f-204">Notice the addition of an `__EFMigrationsHistory` table.</span></span> <span data-ttu-id="fa81f-205">El `__EFMigrationsHistory` tabla realiza un seguimiento de las migraciones se han aplicado a la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fa81f-205">The `__EFMigrationsHistory` table keeps track of which migrations have been applied to the DB.</span></span> <span data-ttu-id="fa81f-206">Ver los datos en el `__EFMigrationsHistory` tabla, muestra una fila para la primera migración.</span><span class="sxs-lookup"><span data-stu-id="fa81f-206">View the data in the `__EFMigrationsHistory` table, it shows one row for the first migration.</span></span> <span data-ttu-id="fa81f-207">El último registro en el ejemplo de salida CLI anterior muestra la instrucción INSERT que crea esta fila.</span><span class="sxs-lookup"><span data-stu-id="fa81f-207">The last log in the preceding CLI output example shows the INSERT statement that creates this row.</span></span>

<span data-ttu-id="fa81f-208">Ejecute la aplicación y compruebe que todo funciona.</span><span class="sxs-lookup"><span data-stu-id="fa81f-208">Run the app and verify that everything works.</span></span>

## <a name="appling-migrations-in-production"></a><span data-ttu-id="fa81f-209">Migraciones de aplicar en producción</span><span class="sxs-lookup"><span data-stu-id="fa81f-209">Appling migrations in production</span></span>

<span data-ttu-id="fa81f-210">Se recomienda que las aplicaciones de producción deberían **no** llamar a [Database.Migrate](https://docs.microsoft.com/dotnet/api/microsoft.entityframeworkcore.relationaldatabasefacadeextensions.migrate?view=efcore-2.0#Microsoft_EntityFrameworkCore_RelationalDatabaseFacadeExtensions_Migrate_Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_) al iniciar la aplicación.</span><span class="sxs-lookup"><span data-stu-id="fa81f-210">We recommend production apps should **not** call [Database.Migrate](https://docs.microsoft.com/dotnet/api/microsoft.entityframeworkcore.relationaldatabasefacadeextensions.migrate?view=efcore-2.0#Microsoft_EntityFrameworkCore_RelationalDatabaseFacadeExtensions_Migrate_Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_) at application startup.</span></span> <span data-ttu-id="fa81f-211">`Migrate`no debe llamarse desde una aplicación en la granja de servidores.</span><span class="sxs-lookup"><span data-stu-id="fa81f-211">`Migrate` should not be called from an app in server farm.</span></span> <span data-ttu-id="fa81f-212">Por ejemplo, si la aplicación se ha implementado con escalado horizontal (se ejecutan varias instancias de la aplicación) en la nube.</span><span class="sxs-lookup"><span data-stu-id="fa81f-212">For example, if the app has been cloud deployed with scale-out (multiple instances of the app are running).</span></span>

<span data-ttu-id="fa81f-213">Migración de base de datos debe realizarse como parte de la implementación y de un modo controlado.</span><span class="sxs-lookup"><span data-stu-id="fa81f-213">Database migration should be done as part of deployment, and in a controlled way.</span></span> <span data-ttu-id="fa81f-214">Entre los métodos de migración de base de datos de producción se incluyen:</span><span class="sxs-lookup"><span data-stu-id="fa81f-214">Production database migration approaches include:</span></span>

* <span data-ttu-id="fa81f-215">Mediante las migraciones para crear secuencias de comandos SQL y las secuencias de comandos SQL en la implementación.</span><span class="sxs-lookup"><span data-stu-id="fa81f-215">Using migrations to create SQL scripts and using the SQL scripts in deployment.</span></span>
* <span data-ttu-id="fa81f-216">Ejecutando `dotnet ef database update` desde un entorno controlado.</span><span class="sxs-lookup"><span data-stu-id="fa81f-216">Running `dotnet ef database update` from a controlled environment.</span></span>

<span data-ttu-id="fa81f-217">EF Core utiliza el `__MigrationsHistory` tabla para ver si necesitan ejecutar las migraciones.</span><span class="sxs-lookup"><span data-stu-id="fa81f-217">EF Core uses the `__MigrationsHistory` table to see if any migrations need to run.</span></span> <span data-ttu-id="fa81f-218">Si la base de datos está actualizado, no se ejecuta migración.</span><span class="sxs-lookup"><span data-stu-id="fa81f-218">If the DB is up to date, no migration is run.</span></span>

<a id="pmc"></a>
## <a name="command-line-interface-cli-vs-package-manager-console-pmc"></a><span data-ttu-id="fa81f-219">Diferencias entre la interfaz de línea de comandos (CLI) y Consola de administrador de paquetes (PMC)</span><span class="sxs-lookup"><span data-stu-id="fa81f-219">Command-line interface (CLI) vs. Package Manager Console (PMC)</span></span>

<span data-ttu-id="fa81f-220">El núcleo de EF conjunto de herramientas para administrar las migraciones están disponible desde:</span><span class="sxs-lookup"><span data-stu-id="fa81f-220">The EF Core tooling for managing migrations is available from:</span></span>

* <span data-ttu-id="fa81f-221">Comandos de CLI de .NET core.</span><span class="sxs-lookup"><span data-stu-id="fa81f-221">.NET Core CLI commands.</span></span>
* <span data-ttu-id="fa81f-222">Los cmdlets de PowerShell en Visual Studio **Package Manager Console** ventana (PMC).</span><span class="sxs-lookup"><span data-stu-id="fa81f-222">The PowerShell cmdlets in the Visual Studio **Package Manager Console** (PMC) window.</span></span>

<span data-ttu-id="fa81f-223">Este tutorial muestra cómo utilizar la CLI, algunos desarrolladores prefieren usar la PMC.</span><span class="sxs-lookup"><span data-stu-id="fa81f-223">This tutorial shows how to use the CLI, some developers prefer using the PMC.</span></span>

<span data-ttu-id="fa81f-224">Los comandos de EF principales para el PMC están en el [Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools) paquete.</span><span class="sxs-lookup"><span data-stu-id="fa81f-224">The EF Core commands for the PMC are in the [Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools) package.</span></span> <span data-ttu-id="fa81f-225">Este paquete se incluye en el [Microsoft.AspNetCore.All](xref:fundamentals/metapackage) metapackage, por lo que no tiene que instalarlo.</span><span class="sxs-lookup"><span data-stu-id="fa81f-225">This package is included in the [Microsoft.AspNetCore.All](xref:fundamentals/metapackage) metapackage, so you don't have to install it.</span></span>

<span data-ttu-id="fa81f-226">**Importante:** no es el mismo paquete que la instalación de la CLI mediante la edición de la *.csproj* archivo.</span><span class="sxs-lookup"><span data-stu-id="fa81f-226">**Important:** This is not the same package as the one you install for the CLI by editing the *.csproj* file.</span></span> <span data-ttu-id="fa81f-227">El nombre de éste termina en `Tools`, a diferencia del nombre de paquete CLI que finaliza en `Tools.DotNet`.</span><span class="sxs-lookup"><span data-stu-id="fa81f-227">The name of this one ends in `Tools`, unlike the CLI package name which ends in `Tools.DotNet`.</span></span>

<span data-ttu-id="fa81f-228">Para obtener más información acerca de los comandos CLI, consulte [.NET Core CLI](https://docs.microsoft.com/ef/core/miscellaneous/cli/dotnet).</span><span class="sxs-lookup"><span data-stu-id="fa81f-228">For more information about the CLI commands, see [.NET Core CLI](https://docs.microsoft.com/ef/core/miscellaneous/cli/dotnet).</span></span>

<span data-ttu-id="fa81f-229">Para obtener más información acerca de los comandos PMC, consulte [Package Manager Console (Visual Studio)](https://docs.microsoft.com/ef/core/miscellaneous/cli/powershell).</span><span class="sxs-lookup"><span data-stu-id="fa81f-229">For more information about the PMC commands, see [Package Manager Console (Visual Studio)](https://docs.microsoft.com/ef/core/miscellaneous/cli/powershell).</span></span>

## <a name="troubleshooting"></a><span data-ttu-id="fa81f-230">Solución de problemas</span><span class="sxs-lookup"><span data-stu-id="fa81f-230">Troubleshooting</span></span>

<span data-ttu-id="fa81f-231">Descargue el [aplicación final de esta fase](
https://github.com/aspnet/Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/StageSnapShots/cu-part4-migrations).</span><span class="sxs-lookup"><span data-stu-id="fa81f-231">Download the [completed app for this stage](
https://github.com/aspnet/Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/StageSnapShots/cu-part4-migrations).</span></span>

<span data-ttu-id="fa81f-232">La aplicación genera la siguiente excepción:</span><span class="sxs-lookup"><span data-stu-id="fa81f-232">The app generates the following exception:</span></span>

```text
`SqlException: Cannot open database "ContosoUniversity" requested by the login.
The login failed.
Login failed for user 'user name'.
```

<span data-ttu-id="fa81f-233">Solución: ejecute`dotnet ef database update`</span><span class="sxs-lookup"><span data-stu-id="fa81f-233">Solution: Run `dotnet ef database update`</span></span>

<span data-ttu-id="fa81f-234">Si el `update` comando devuelve el error "Error al generar.":</span><span class="sxs-lookup"><span data-stu-id="fa81f-234">If the `update` command returns the error "Build failed.":</span></span>

* <span data-ttu-id="fa81f-235">Vuelva a ejecutar el comando.</span><span class="sxs-lookup"><span data-stu-id="fa81f-235">Run the command again.</span></span>
* <span data-ttu-id="fa81f-236">Dejar un mensaje en la parte inferior de la página.</span><span class="sxs-lookup"><span data-stu-id="fa81f-236">Leave a message at the bottom of the page.</span></span>

>[!div class="step-by-step"]
<span data-ttu-id="fa81f-237">[Anterior](xref:data/ef-rp/sort-filter-page)
[Siguiente](xref:data/ef-rp/complex-data-model)</span><span class="sxs-lookup"><span data-stu-id="fa81f-237">[Previous](xref:data/ef-rp/sort-filter-page)
[Next](xref:data/ef-rp/complex-data-model)</span></span>