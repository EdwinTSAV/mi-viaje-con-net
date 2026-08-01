# DTOs y mapeo con AutoMapper (ejemplo Persona)

## Tabla de contenido
- [¿Para qué sirve?](#para-qué-sirve)
- [1. Crear el DTO de respuesta](#1-crear-el-dto-de-respuesta)
- [2. Crear el Mapping Profile](#2-crear-el-mapping-profile)
- [3. Registrar AutoMapper en Application](#3-registrar-automapper-en-application)
- [4. Usar el DTO en las Queries](#4-usar-el-dto-en-las-queries)
- [5. El controlador no cambia](#5-el-controlador-no-cambia)
- [Buenas prácticas](#buenas-prácticas)
- [Errores comunes](#errores-comunes)
- [Recursos relacionados](#recursos-relacionados)

## ¿Para qué sirve?

Hasta ahora, las Queries de [Crear un CRUD](crear-crud.md) devuelven directamente la entidad de dominio (`Persona`) al controlador. Esto funciona, pero acopla el contrato de la API a los detalles internos de persistencia: cualquier cambio en la entidad (agregar una columna interna, renombrar una propiedad por una migración) rompe automáticamente la respuesta que reciben los consumidores de la API.

Un **DTO (Data Transfer Object)** de respuesta resuelve esto: define explícitamente qué campos se exponen hacia afuera, independientemente de cómo esté modelada la entidad por dentro. **AutoMapper** evita escribir el mapeo campo por campo a mano.

> **Nota**
> No se crea un DTO de "entrada" (`Request`) para `Crear`/`Actualizar`, porque en este proyecto los propios `Command` (`CrearPersonaCommand`, `ActualizarPersonaCommand`) ya cumplen esa función: son el objeto que el controlador recibe del cliente. Solo se agrega DTO para lo que **sale** de la API (`Response`), que es donde se filtra qué expone la entidad de dominio.

## 1. Crear el DTO de respuesta

En `Application/DTOs/PersonaResponse.cs`:

```csharp
namespace MiProyecto.Application.DTOs;

public class PersonaResponse
{
    public Guid Id { get; set; }
    public string NroDni { get; set; } = string.Empty;
    public string PrimerNombre { get; set; } = string.Empty;
    public string ApellidoPaterno { get; set; } = string.Empty;
    public string ApellidoMaterno { get; set; } = string.Empty;
    public DateOnly FechaNacimiento { get; set; }
}
```

> **Tip**
> En este ejemplo `PersonaResponse` tiene los mismos campos que la entidad `Persona`, así que el beneficio no se nota a simple vista. La diferencia se hace evidente cuando la entidad crece con campos que **no** deben salir por la API (por ejemplo, un campo interno de auditoría o una clave foránea técnica): el DTO solo expone lo que tú decidas.

## 2. Crear el Mapping Profile

En `Application/Mappings/PersonaProfile.cs`, se declara la correspondencia entre la entidad y el DTO:

```csharp
using AutoMapper;
using MiProyecto.Application.DTOs;
using MiProyecto.Domain.Entities;

namespace MiProyecto.Application.Mappings;

public class PersonaProfile : Profile
{
    public PersonaProfile()
    {
        CreateMap<Persona, PersonaResponse>();
    }
}
```

> **Nota**
> Como `PersonaResponse` tiene exactamente los mismos nombres de propiedad que `Persona`, AutoMapper resuelve el mapeo automáticamente sin configuración adicional. Si en el futuro un campo necesita transformarse o renombrarse, se agrega con `.ForMember(...)` dentro de este mismo `CreateMap`.

## 3. Registrar AutoMapper en Application

En `Application/DependencyInjection.cs` (el mismo archivo de [Preparar Application](preparar-application.md)), se agrega el registro de AutoMapper:

```csharp
services.AddAutoMapper(Assembly.GetExecutingAssembly());
```

Quedando el método completo así:

```csharp
public static class DependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        services.AddMediatR(cfg =>
            cfg.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly()));

        services.AddAutoMapper(Assembly.GetExecutingAssembly());

        return services;
    }
}
```

> **Tip**
> No es necesario registrar `PersonaProfile` uno por uno: `AddAutoMapper(Assembly.GetExecutingAssembly())` escanea el assembly completo de `Application` en busca de clases que hereden de `Profile`, igual que MediatR escanea los Handlers.

## 4. Usar el DTO en las Queries

Se actualiza la Query y su Handler para que devuelvan `PersonaResponse` en lugar de la entidad `Persona`. Ejemplo con `ListarPersonasQuery` (de [Crear un CRUD](crear-crud.md#paso-9--crear-un-commandhandler-y-un-queryhandler)):

```csharp
using MediatR;
using MiProyecto.Application.DTOs;

namespace MiProyecto.Application.Queries.Persona.Listar;

public record ListarPersonasQuery : IRequest<List<PersonaResponse>>;
```

```csharp
using AutoMapper;
using MediatR;
using MiProyecto.Application.DTOs;
using MiProyecto.Application.Interfaces;

namespace MiProyecto.Application.Queries.Persona.Listar;

public class ListarPersonasQueryHandler(IPersonaRepository repositorio, IMapper mapper)
    : IRequestHandler<ListarPersonasQuery, List<PersonaResponse>>
{
    public async Task<List<PersonaResponse>> Handle(ListarPersonasQuery request, CancellationToken cancellationToken)
    {
        var personas = await repositorio.ListarAsync(cancellationToken);
        return mapper.Map<List<PersonaResponse>>(personas);
    }
}
```

`ObtenerPersonaPorIdQuery` sigue el mismo patrón, devolviendo `PersonaResponse?` en lugar de `Persona?`.

> **Nota**
> El Command de `Crear` (`CrearPersonaCommandHandler`) no necesita este cambio: ya devuelve solo el `Guid` del `Id` creado, no la entidad completa, así que no expone nada del dominio.

## 5. El controlador no cambia

`PersonaController` (creado en [Crear un CRUD](crear-crud.md#paso-10--crear-el-controlador)) no requiere ninguna modificación: sigue llamando a `mediator.Send(...)` igual que antes. El cambio de `Persona` a `PersonaResponse` ocurre completamente dentro de `Application`, sin afectar la capa `API`.

## Buenas prácticas

- Crea un `Profile` por entidad (`PersonaProfile`, `PedidoProfile`, etc.), nunca un único `Profile` gigante para todo el proyecto.
- Cuando la lista de resultados venga de una consulta a la base de datos y el volumen de datos sea grande, evalúa `ProjectTo<PersonaResponse>(configuration)` (de `AutoMapper.QueryableExtensions`) en lugar de `Map<List<T>>`: proyecta directamente en la consulta SQL, sin traer a memoria columnas que el DTO no necesita.
- Mantén el `Profile` con mapeos puros (asignación de campos); si necesitas lógica condicional compleja, considera un mapeo manual o un método de extensión en vez de forzarlo dentro de `CreateMap`.

## Errores comunes

- ❌ Olvidar que `AddAutoMapper` debe apuntar al assembly donde están los `Profile` (`Application`), no al de `API` ni al de `Domain`.
- ❌ Definir el DTO de respuesta dentro de `Domain` o `Infrastructure`: los DTOs son un concepto de `Application` (o incluso de `API`), nunca del dominio.
- ❌ Reutilizar el mismo DTO de respuesta como DTO de entrada para `Crear`/`Actualizar`: mezclar ambos casos suele terminar con campos opcionales que no deberían serlo (por ejemplo, `Id` en un DTO de creación).

## Recursos relacionados

- [Crear un CRUD](crear-crud.md)
- [Crear una entidad](crear-entidad.md)
- [Preparar Application (registro de MediatR)](preparar-application.md)
- [Gestión de paquetes NuGet](paquetes-nuget.md)

[⬅ Volver al índice de .NET](README.md)
