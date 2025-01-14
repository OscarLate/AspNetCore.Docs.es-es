---
title: Formularios y validación de ASP.NET Core increíbles
author: guardrex
description: Obtenga información sobre cómo usar los escenarios de validación de campos y formularios en extraordinarias.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/23/2019
uid: blazor/forms-validation
ms.openlocfilehash: 6f6ace3deb7ed4262b643d897273bc767334b5e6
ms.sourcegitcommit: 79eeb17604b536e8f34641d1e6b697fb9a2ee21f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/24/2019
ms.locfileid: "71207182"
---
# <a name="aspnet-core-blazor-forms-and-validation"></a>Formularios y validación de ASP.NET Core increíbles

Por [Daniel Roth](https://github.com/danroth27) y [Luke Latham](https://github.com/guardrex)

Los formularios y la validación se admiten en el uso de [anotaciones de datos](xref:mvc/models/validation).

El tipo `ExampleModel` siguiente define la lógica de validación mediante anotaciones de datos:

```csharp
using System.ComponentModel.DataAnnotations;

public class ExampleModel
{
    [Required]
    [StringLength(10, ErrorMessage = "Name is too long.")]
    public string Name { get; set; }
}
```

Un formulario se define mediante el `EditForm` componente. En el siguiente formulario se muestran los elementos, componentes y código Razor típicos:

```csharp
<EditForm Model="@exampleModel" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <InputText id="name" @bind-Value="@exampleModel.Name" />

    <button type="submit">Submit</button>
</EditForm>

@code {
    private ExampleModel exampleModel = new ExampleModel();

    private void HandleValidSubmit()
    {
        Console.WriteLine("OnValidSubmit");
    }
}
```

* El formulario valida los datos proporcionados por `name` el usuario en el campo utilizando la `ExampleModel` validación definida en el tipo. El modelo se crea en el bloque del `@code` componente y se mantiene en un campo privado`exampleModel`(). El campo se asigna al `Model` atributo `<EditForm>` del elemento.
* El `DataAnnotationsValidator` componente adjunta la compatibilidad con la validación mediante anotaciones de datos.
* El `ValidationSummary` componente resume los mensajes de validación.
* `HandleValidSubmit`se desencadena cuando el formulario se envía correctamente (pasa la validación).

Hay disponible un conjunto de componentes de entrada integrados para recibir y validar los datos proporcionados por el usuario. Las entradas se validan cuando se cambian y cuando se envía un formulario. Los componentes de entrada disponibles se muestran en la tabla siguiente.

| Componente de entrada | Se representa como&hellip;       |
| --------------- | ------------------------- |
| `InputText`     | `<input>`                 |
| `InputTextArea` | `<textarea>`              |
| `InputSelect`   | `<select>`                |
| `InputNumber`   | `<input type="number">`   |
| `InputCheckbox` | `<input type="checkbox">` |
| `InputDate`     | `<input type="date">`     |

Todos los componentes de entrada, incluido `EditForm`, admiten atributos arbitrarios. Cualquier atributo que no coincida con un parámetro de componente se agrega al elemento HTML representado.

Los componentes de entrada proporcionan el comportamiento predeterminado para validar en la edición y cambiar su clase CSS para reflejar el estado del campo. Algunos componentes incluyen lógica de análisis útil. Por ejemplo, `InputDate` y `InputNumber` controlan los valores que no se puedan analizar correctamente si se registran como errores de validación. Los tipos que pueden aceptar valores null también admiten la nulabilidad del campo de destino ( `int?`por ejemplo,).

El tipo `Starship` siguiente define la lógica de validación mediante un conjunto mayor de propiedades y anotaciones de datos que `ExampleModel`los anteriores:

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class Starship
{
    [Required]
    [StringLength(16, ErrorMessage = "Identifier too long (16 character limit).")]
    public string Identifier { get; set; }

    public string Description { get; set; }

    [Required]
    public string Classification { get; set; }

    [Range(1, 100000, ErrorMessage = "Accommodation invalid (1-100000).")]
    public int MaximumAccommodation { get; set; }

    [Required]
    [Range(typeof(bool), "true", "true", 
        ErrorMessage = "This form disallows unapproved ships.")]
    public bool IsValidatedDesign { get; set; }

    [Required]
    public DateTime ProductionDate { get; set; }
}
```

En el ejemplo anterior, `Description` es opcional porque no hay ninguna anotación de datos presente.

En el siguiente formulario se validan los datos proporcionados por el `Starship` usuario mediante la validación definida en el modelo:

```cshtml
@page "/FormsValidation"

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <p>
        <label for="identifier">Identifier: </label>
        <InputText id="identifier" @bind-Value="starship.Identifier" />
    </p>
    <p>
        <label for="description">Description (optional): </label>
        <InputTextArea id="description" @bind-Value="starship.Description" />
    </p>
    <p>
        <label for="classification">Primary Classification: </label>
        <InputSelect id="classification" @bind-Value="starship.Classification">
            <option value="">Select classification ...</option>
            <option value="Exploration">Exploration</option>
            <option value="Diplomacy">Diplomacy</option>
            <option value="Defense">Defense</option>
        </InputSelect>
    </p>
    <p>
        <label for="accommodation">Maximum Accommodation: </label>
        <InputNumber id="accommodation" 
            @bind-Value="starship.MaximumAccommodation" />
    </p>
    <p>
        <label for="valid">Engineering Approval: </label>
        <InputCheckbox id="valid" @bind-Value="starship.IsValidatedDesign" />
    </p>
    <p>
        <label for="productionDate">Production Date: </label>
        <InputDate id="productionDate" @bind-Value="starship.ProductionDate" />
    </p>

    <button type="submit">Submit</button>

    <p>
        <a href="http://www.startrek.com/">Star Trek</a>, 
        &copy;1966-2019 CBS Studios, Inc. and 
        <a href="https://www.paramount.com">Paramount Pictures</a>
    </p>
</EditForm>

@code {
    private Starship starship = new Starship();

    private void HandleValidSubmit()
    {
        Console.WriteLine("OnValidSubmit");
    }
}
```

Crea como un [valor en cascada](xref:blazor/components#cascading-values-and-parameters) que realiza un seguimiento de los metadatos sobre el proceso de edición, incluidos los campos modificados y los mensajes de validación actuales. `EditContext` `EditForm` También proporciona eventos útiles para envíos válidos y no válidos `OnInvalidSubmit`(`OnValidSubmit`,). `EditForm` También puede usar `OnSubmit` para desencadenar la validación y comprobar los valores de los campos con código de validación personalizado.

## <a name="inputtext-based-on-the-input-event"></a>InputText basado en el evento de entrada

Utilice el `InputText` componente para crear un componente personalizado que use el `input` `change` evento en lugar del evento.

Cree un componente con el marcado siguiente y use el componente tal y como `InputText` se usa:

```cshtml
@inherits InputText

<input 
    @attributes="AdditionalAttributes" 
    class="@CssClass" 
    value="@CurrentValue" 
    @oninput="EventCallback.Factory.CreateBinder<string>(
        this, __value => CurrentValueAsString = __value, CurrentValueAsString)" />
```

## <a name="validation-support"></a>Compatibilidad con la validación

El `DataAnnotationsValidator` componente adjunta la compatibilidad con la validación mediante anotaciones de datos en `EditContext`cascada. Habilitar la compatibilidad con la validación mediante anotaciones de datos requiere este gesto explícito. Para usar un sistema de validación distinto al de las `DataAnnotationsValidator` anotaciones de datos, reemplace con una implementación personalizada. La implementación de ASP.NET Core está disponible para su inspección en el origen de referencia: [DataAnnotationsValidator](https://github.com/aspnet/AspNetCore/blob/master/src/Components/Forms/src/DataAnnotationsValidator.cs)/[AddDataAnnotationsValidation](https://github.com/aspnet/AspNetCore/blob/master/src/Components/Forms/src/EditContextDataAnnotationsExtensions.cs).

El `ValidationSummary` componente resume todos los mensajes de validación, que es similar a la [aplicación auxiliar de etiquetas de Resumen de validación](xref:mvc/views/working-with-forms#the-validation-summary-tag-helper).

El `ValidationMessage` componente muestra los mensajes de validación de un campo específico, que es similar a la [aplicación auxiliar de etiquetas de mensaje de validación](xref:mvc/views/working-with-forms#the-validation-message-tag-helper). Especifique el campo para la validación con `For` el atributo y una expresión lambda con el nombre de la propiedad del modelo:

```cshtml
<ValidationMessage For="@(() => starship.MaximumAccommodation)" />
```

Los `ValidationMessage` componentes `ValidationSummary` y admiten atributos arbitrarios. Cualquier atributo que no coincida con un parámetro de componente se agrega al `<div>` elemento `<ul>` o generado.

### <a name="validation-of-complex-or-collection-type-properties"></a>Validación de propiedades de tipo complejo o de colección

Los atributos de validación que se aplican a las propiedades de un modelo se validan cuando se envía el formulario. Sin embargo, las propiedades de las colecciones o los tipos de datos complejos de un modelo no se validan en el envío del formulario. Para respetar los atributos de validación anidados en este escenario, utilice un componente de validación personalizado. Para obtener un ejemplo, consulte el [ejemplo de validación de increíble información en el repositorio de github de ASPNET/samples](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/blazor/Validation).
