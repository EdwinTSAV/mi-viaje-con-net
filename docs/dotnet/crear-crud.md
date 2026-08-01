# Crear un CRUD (ejemplo: Persona)

## Tabla de contenido
- [¿Para qué sirve?](#para-qué-sirve)
- [Paso 1 — Estructura de Commands y Queries](#paso-1--estructura-de-commands-y-queries)
- [Paso 2 — Crear un Command/Handler y un Query/Handler](#paso-2--crear-un-commandhandler-y-un-queryhandler)
- [Paso 3 — Crear el controlador](#paso-3--crear-el-controlador)
- [¿Qué falta para completar la Fase 0?](#qué-falta-para-completar-la-fase-0)
- [Recursos relacionados](#recursos-relacionados)

## ¿Para qué sirve?

Este documento describe el flujo para construir la **lógica de negocio** del primer CRUD de un microservicio bajo Clean Architecture: la estructura de Commands/Queries, sus Handlers despachados con **MediatR**, y el controlador que los expone como endpoints HTTP.

Se usa como ejemplo la entidad **Persona**, con los campos: `Id`, `NroDni`, `PrimerNombre`, `ApellidoPaterno`, `ApellidoMaterno` y `FechaNacimiento`.

> **Nota**
> Este documento asume que ya completaste los dos pasos previos de la ruta de aprendizaje: la entidad, su configuración, migración y repositorio (ver [Crear una entidad](crear-entidad.md)), y el registro de MediatR en `Application` (ver [Preparar Application](preparar-application.md)). Sin esos dos, el código de este documento no compila.

## Paso 1 — Estructura de Commands y Queries

Siguiendo el patrón **CQRS** ya descrito en [Estructura de carpetas por capa](../arquitectura/estructura-carpetas.md), cada operación del CRUD se organiza en su propia carpeta dentro de `Application`:

```text
Application/
├── Commands/
│   └── Personas/
│       ├── Crear/
│       ├── Actualizar/
│       └── Eliminar/
└── Queries/
    └── Personas/
        ├── Listar/
        └── ObtenerPorId/
```

> **Nota**
> La carpeta (y por lo tanto el namespace) se llama `Personas` **en plural**, a propósito: si se llamara `Persona` en singular, colisionaría con la clase `Persona` del dominio (`Application.Commands.Persona.Crear` vs. `Domain.Entities.Persona`), obligando a calificar el tipo en cada archivo (`Domain.Entities.Persona`) o a usar un alias. Nombrando la carpeta en plural, ambos conviven sin conflicto y el código queda más simple.

## Paso 2 — Crear un Command/Handler y un Query/Handler

### Command: Crear Persona

`Application/Commands/Personas/Crear/CrearPersonaCommand.cs`:

```csharp
using MediatR;

namespace MiProyecto.Application.Commands.Personas.Crear;

public record CrearPersonaCommand(
    string NroDni,
    string PrimerNombre,
    string ApellidoPaterno,
    string ApellidoMaterno,
    DateOnly FechaNacimiento
) : IRequest<Guid>;
```

`Application/Commands/Personas/Crear/CrearPersonaCommandHandler.cs`:

```csharp
using MediatR;
using MiProyecto.Application.Interfaces;
using MiProyecto.Domain.Entities;

namespace MiProyecto.Application.Commands.Personas.Crear;

public class CrearPersonaCommandHandler(IPersonaRepository repositorio)
    : IRequestHandler<CrearPersonaCommand, Guid>
{
    public async Task<Guid> Handle(CrearPersonaCommand request, CancellationToken cancellationToken)
    {
        var persona = new Persona
        {
            Id = Guid.NewGuid(),
            NroDni = request.NroDni,
            PrimerNombre = request.PrimerNombre,
            ApellidoPaterno = request.ApellidoPaterno,
            ApellidoMaterno = request.ApellidoMaterno,
            FechaNacimiento = request.FechaNacimiento
        };

        await repositorio.AgregarAsync(persona, cancellationToken);
        await repositorio.GuardarCambiosAsync(cancellationToken);

        return persona.Id;
    }
}
```

### Query: Listar Personas

`Application/Queries/Personas/Listar/ListarPersonasQuery.cs`:

```csharp
using MediatR;
using MiProyecto.Domain.Entities;

namespace MiProyecto.Application.Queries.Personas.Listar;

public record ListarPersonasQuery : IRequest<List<Persona>>;
```

`Application/Queries/Personas/Listar/ListarPersonasQueryHandler.cs`:

```csharp
using MediatR;
using MiProyecto.Application.Interfaces;
using MiProyecto.Domain.Entities;

namespace MiProyecto.Application.Queries.Personas.Listar;

public class ListarPersonasQueryHandler(IPersonaRepository repositorio)
    : IRequestHandler<ListarPersonasQuery, List<Persona>>
{
    public Task<List<Persona>> Handle(ListarPersonasQuery request, CancellationToken cancellationToken)
        => repositorio.ListarAsync(cancellationToken);
}
```

> **Tip**
> `Actualizar`, `Eliminar` y `ObtenerPorId` siguen exactamente el mismo patrón: un `record` que implementa `IRequest<T>` y un handler que recibe `IPersonaRepository` por constructor. Recréalos siguiendo el mismo molde que `Crear` y `Listar`.

## Paso 3 — Crear el controlador

En `API/Controllers/PersonaController.cs`:

```csharp
using MediatR;
using Microsoft.AspNetCore.Mvc;
using MiProyecto.Application.Commands.Personas.Actualizar;
using MiProyecto.Application.Commands.Personas.Crear;
using MiProyecto.Application.Commands.Personas.Eliminar;
using MiProyecto.Application.Queries.Personas.Listar;
using MiProyecto.Application.Queries.Personas.ObtenerPorId;

namespace MiProyecto.API.Controllers;

[ApiController]
[Route("api/[controller]")]
public class PersonaController(IMediator mediator) : ControllerBase
{
    [HttpGet]
    public async Task<IActionResult> Listar(CancellationToken cancellationToken)
    {
        var personas = await mediator.Send(new ListarPersonasQuery(), cancellationToken);
        return Ok(personas);
    }

    [HttpGet("{id:guid}")]
    public async Task<IActionResult> ObtenerPorId(Guid id, CancellationToken cancellationToken)
    {
        var persona = await mediator.Send(new ObtenerPersonaPorIdQuery(id), cancellationToken);
        return persona is null ? NotFound() : Ok(persona);
    }

    [HttpPost]
    public async Task<IActionResult> Crear(CrearPersonaCommand command, CancellationToken cancellationToken)
    {
        var id = await mediator.Send(command, cancellationToken);
        return CreatedAtAction(nameof(ObtenerPorId), new { id }, id);
    }

    [HttpPut("{id:guid}")]
    public async Task<IActionResult> Actualizar(Guid id, ActualizarPersonaCommand command, CancellationToken cancellationToken)
    {
        if (id != command.Id) return BadRequest();

        await mediator.Send(command, cancellationToken);
        return NoContent();
    }

    [HttpDelete("{id:guid}")]
    public async Task<IActionResult> Eliminar(Guid id, CancellationToken cancellationToken)
    {
        await mediator.Send(new EliminarPersonaCommand(id), cancellationToken);
        return NoContent();
    }
}
```

> **Nota**
> Este controlador asume que `ActualizarPersonaCommand` e `EliminarPersonaCommand` (con `Id` como parámetro) y `ObtenerPersonaPorIdQuery(Guid id)` ya fueron creados siguiendo el mismo patrón del Paso 2.

## ¿Qué falta para completar la Fase 0?

Con estos 3 pasos —sumados a la entidad y a la preparación de Application ya hechas en los documentos previos— ya tienes un CRUD funcional de punta a punta. Antes de dar por cerrada la Fase 0, quedan dos detalles que conviene tener en cuenta (aunque su documentación completa corresponde a fases posteriores):

1. **DTOs de entrada/salida**: en este ejemplo el controlador recibe y devuelve directamente `Command`/`Query`/entidad de dominio. Para un CRUD ya listo para producción, normalmente se agregan DTOs de respuesta (por ejemplo, `PersonaResponse`) y se mapean con **AutoMapper**, evitando exponer la entidad de dominio tal cual en la API.
2. **Validaciones, manejo de errores y respuestas uniformes**: por ahora, los Commands de `Crear`/`Actualizar` no validan nada (por ejemplo, que `NroDni` no esté vacío) ni se maneja el caso de que la entidad no exista al actualizar/eliminar. Esto corresponde a la **Fase 1** de la ruta de aprendizaje (FluentValidation + Pipeline Behaviors, excepciones, middleware global y respuestas estandarizadas).

## Recursos relacionados

- [Crear una entidad](crear-entidad.md)
- [Preparar Application (registro de MediatR)](preparar-application.md)
- [Clean Architecture](../arquitectura/clean-architecture.md)
- [Estructura de carpetas por capa](../arquitectura/estructura-carpetas.md)

[⬅ Volver al índice de .NET](README.md)
