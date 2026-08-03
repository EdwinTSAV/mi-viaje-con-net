# Mapeo con AutoMapper (ejemplo Persona)

## Tabla de contenido
- [¿Para qué sirve?](#para-qué-sirve)
- [1. Crear el Mapping Profile](#1-crear-el-mapping-profile)
- [2. Registrar AutoMapper en Application](#2-registrar-automapper-en-application)
- [3. Usar el mapeo en las Queries](#3-usar-el-mapeo-en-las-queries)
- [4. Renombrar una propiedad con ForMember](#4-renombrar-una-propiedad-con-formember)
- [5. Combinar varias propiedades en una sola](#5-combinar-varias-propiedades-en-una-sola)
- [El controlador no cambia](#el-controlador-no-cambia)
- [Buenas prácticas](#buenas-prácticas)
- [Errores comunes](#errores-comunes)
- [Recursos relacionados](#recursos-relacionados)

## ¿Para qué sirve?

**AutoMapper** convierte una entidad de dominio en un DTO (o viceversa) sin que tengas que escribir el mapeo campo por campo a mano. Este documento asume que ya creaste `PersonaResponse` en [DTOs de respuesta](dtos.md); aquí se cubre cómo configurar la conversión de `Persona` → `PersonaResponse`, incluyendo los casos en que los nombres de propiedad **no coinciden**.

## 1. Crear el Mapping Profile

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
> Mientras `PersonaResponse` tenga exactamente los mismos nombres de propiedad que `Persona` (como en [DTOs de respuesta](dtos.md)), AutoMapper resuelve el mapeo automáticamente sin configuración adicional. Los casos donde esto no alcanza se ven en los pasos 4 y 5.

## 2. Registrar AutoMapper en Application

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

## 3. Usar el mapeo en las Queries

Se actualiza la Query y su Handler para que devuelvan `PersonaResponse` en lugar de la entidad `Persona`. Ejemplo con `ListarPersonasQuery` (de [Crear un CRUD](crear-crud.md)):

```csharp
using MediatR;
using MiProyecto.Application.DTOs;

namespace MiProyecto.Application.Queries.Personas.Listar;

public record ListarPersonasQuery : IRequest<List<PersonaResponse>>;
```

```csharp
using AutoMapper;
using MediatR;
using MiProyecto.Application.DTOs;
using MiProyecto.Application.Interfaces;

namespace MiProyecto.Application.Queries.Personas.Listar;

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

## 4. Renombrar una propiedad con ForMember

Supongamos que `PersonaResponse` expone el nombre en inglés (`FirstName`), mientras que en el dominio la propiedad se llama `PrimerNombre`:

```csharp
namespace MiProyecto.Application.DTOs;

public class PersonaResponse
{
    public Guid Id { get; set; }
    public string NroDni { get; set; } = string.Empty;
    public string FirstName { get; set; } = string.Empty; // en Domain: PrimerNombre
    public string ApellidoPaterno { get; set; } = string.Empty;
    public string ApellidoMaterno { get; set; } = string.Empty;
    public DateOnly FechaNacimiento { get; set; }
}
```

Como los nombres ya no coinciden, AutoMapper **no** puede resolverlo solo: hay que indicarle explícitamente de dónde sale el valor con `.ForMember(...)`:

```csharp
public class PersonaProfile : Profile
{
    public PersonaProfile()
    {
        CreateMap<Persona, PersonaResponse>()
            .ForMember(
                destino => destino.FirstName,
                opciones => opciones.MapFrom(origen => origen.PrimerNombre));
    }
}
```

| Parte | Significado |
|---|---|
| `destino => destino.FirstName` | La propiedad del DTO (`PersonaResponse`) que se quiere completar |
| `opciones.MapFrom(origen => origen.PrimerNombre)` | De dónde sale el valor: la propiedad `PrimerNombre` de la entidad (`Persona`) |

> **Tip**
> Todas las demás propiedades (`Id`, `NroDni`, `ApellidoPaterno`, `ApellidoMaterno`, `FechaNacimiento`) siguen mapeándose automáticamente, porque sus nombres coinciden entre `Persona` y `PersonaResponse`. Solo necesitas un `.ForMember(...)` por cada propiedad que **no** coincide; no hay que declarar las que sí coinciden.

## 5. Combinar varias propiedades en una sola

Ahora un caso más complejo: `PersonaResponse` expone un único campo `NombresCompletos`, construido a partir de tres propiedades distintas de la entidad (`PrimerNombre`, `ApellidoPaterno`, `ApellidoMaterno`):

```csharp
namespace MiProyecto.Application.DTOs;

public class PersonaResponse
{
    public Guid Id { get; set; }
    public string NroDni { get; set; } = string.Empty;
    public string NombresCompletos { get; set; } = string.Empty; // combinación de 3 campos del dominio
    public DateOnly FechaNacimiento { get; set; }
}
```

```csharp
public class PersonaProfile : Profile
{
    public PersonaProfile()
    {
        CreateMap<Persona, PersonaResponse>()
            .ForMember(
                destino => destino.NombresCompletos,
                opciones => opciones.MapFrom(origen =>
                    $"{origen.PrimerNombre} {origen.ApellidoPaterno} {origen.ApellidoMaterno}"));
    }
}
```

> **Nota**
> Dentro de `MapFrom(origen => ...)` puedes escribir cualquier expresión de C# válida: interpolación de strings (como en este ejemplo), concatenación, operadores condicionales, o incluso llamar a un método. AutoMapper simplemente ejecuta esa función por cada entidad que mapea.

Si en el mismo DTO necesitas **ambos** casos (un renombre simple y una combinación), se encadenan los `.ForMember(...)` uno tras otro:

```csharp
CreateMap<Persona, PersonaResponse>()
    .ForMember(d => d.FirstName, o => o.MapFrom(s => s.PrimerNombre))
    .ForMember(d => d.NombresCompletos, o => o.MapFrom(s =>
        $"{s.PrimerNombre} {s.ApellidoPaterno} {s.ApellidoMaterno}"));
```

## El controlador no cambia

`PersonaController` (creado en [Crear un CRUD](crear-crud.md)) no requiere ninguna modificación: sigue llamando a `mediator.Send(...)` igual que antes. El cambio de `Persona` a `PersonaResponse` ocurre completamente dentro de `Application`, sin afectar la capa `API`.

## Buenas prácticas

- Crea un `Profile` por entidad (`PersonaProfile`, `PedidoProfile`, etc.), nunca un único `Profile` gigante para todo el proyecto.
- Usa `.ForMember(...)` únicamente para las propiedades que de verdad lo necesitan (nombre distinto o valor calculado); no lo agregues para propiedades que ya coinciden por nombre.
- Cuando la lista de resultados venga de una consulta a la base de datos y el volumen de datos sea grande, evalúa `ProjectTo<PersonaResponse>(configuration)` (de `AutoMapper.QueryableExtensions`) en lugar de `Map<List<T>>`: proyecta directamente en la consulta SQL, sin traer a memoria columnas que el DTO no necesita.

## Errores comunes

- ❌ Olvidar el `.ForMember(...)` cuando el nombre de la propiedad no coincide: AutoMapper no lanza error por defecto, simplemente deja el campo en su valor por defecto (`string.Empty`, `0`, `null`), lo cual puede pasar desapercibido si no hay pruebas.
- ❌ Poner lógica de negocio compleja dentro de `MapFrom(...)`: si la expresión empieza a tener varias condiciones o llamadas a servicios, es mejor moverla a un método de la entidad o a un mapeo manual, y dejar el `Profile` con mapeos simples.
- ❌ Olvidar que `AddAutoMapper` debe apuntar al assembly donde están los `Profile` (`Application`), no al de `API` ni al de `Domain`.

## Recursos relacionados

- [DTOs de respuesta](dtos.md)
- [Crear un CRUD](crear-crud.md)
- [Preparar Application (registro de MediatR)](preparar-application.md)
- [Gestión de paquetes NuGet](paquetes-nuget.md)

[⬅ Volver al índice de .NET](README.md)
