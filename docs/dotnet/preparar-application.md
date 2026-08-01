# Preparar Application (registro de MediatR)

## Tabla de contenido
- [¿Para qué sirve?](#para-qué-sirve)
- [1. Crear DependencyInjection](#1-crear-dependencyinjection)
- [2. Registrar Application](#2-registrar-application)
- [¿Qué falta para completar la Fase 0?](#qué-falta-para-completar-la-fase-0)
- [Buenas prácticas](#buenas-prácticas)
- [Recursos relacionados](#recursos-relacionados)

## ¿Para qué sirve?

Este documento describe la configuración **única por microservicio** de la capa `Application`: registrar **MediatR** en el contenedor de dependencias para poder despachar Commands y Queries desde el controlador (`IMediator.Send(...)`).

A diferencia de [Crear una entidad](crear-entidad.md) o [Crear un CRUD](crear-crud.md), que se repiten cada vez que agregas una nueva entidad u operación, este paso se hace **una sola vez** al iniciar el microservicio: una vez registrado, MediatR detecta automáticamente todos los Commands, Queries y Handlers nuevos que agregues más adelante, sin necesidad de volver a tocar este archivo.

> **Nota**
> Este flujo asume que ya tienes: la solución creada, los paquetes instalados y la conexión a la base de datos configurada. Si aún no lo tienes, revisa primero [Crear una solución de microservicios](crear-solucion-microservicios.md) y [Configuración mínima de conexión a la base de datos](crear-conexion-bd.md).

## 1. Crear DependencyInjection

En `Application/DependencyInjection.cs`:

```csharp
using System.Reflection;
using MediatR;
using Microsoft.Extensions.DependencyInjection;

namespace MiProyecto.Application;

public static class DependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        services.AddMediatR(cfg =>
            cfg.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly()));

        return services;
    }
}
```

> **Tip**
> `RegisterServicesFromAssembly(Assembly.GetExecutingAssembly())` escanea automáticamente el proyecto `Application` en busca de clases que implementen `IRequestHandler<,>`. Por eso, al crear un Command/Query/Handler nuevo (ver [Crear un CRUD](crear-crud.md#paso-9--crear-un-commandhandler-y-un-queryhandler)), MediatR lo reconoce sin configuración adicional.

## 2. Registrar Application

En `API/Program.cs`, se registra el método `AddApplication()` creado en el paso anterior:

```csharp
using MiProyecto.Application;
using MiProyecto.Infrastructure;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddApplication();
builder.Services.AddInfrastructure(builder.Configuration);

var app = builder.Build();

app.MapControllers();

app.Run();
```

> **Nota**
> `AddInfrastructure(builder.Configuration)` ya debería existir en tu `Program.cs` desde [Configuración mínima de conexión a la BD](crear-conexion-bd.md). Este paso solo agrega la línea `AddApplication()`, `AddControllers()` y `MapControllers()`; no se reemplaza lo que ya tenías.

## ¿Qué falta para completar la Fase 0?

**Validaciones, manejo de errores y respuestas uniformes**: por ahora, los Commands de `Crear`/`Actualizar` no validan nada (por ejemplo, que `NroDni` no esté vacío) ni se maneja el caso de que la entidad no exista al actualizar/eliminar. Esto corresponde a la **Fase 1** de la ruta de aprendizaje (FluentValidation + Pipeline Behaviors, excepciones, middleware global y respuestas estandarizadas).

> **Nota**
> Cuando llegue esa fase, **este mismo archivo** (`Application/DependencyInjection.cs`) es el que se extenderá — ahí se agregará `services.AddValidatorsFromAssembly(...)` y el registro del `ValidationBehavior` como Pipeline Behavior de MediatR. No será necesario crear un archivo nuevo para eso.

## Buenas prácticas

- No agregues lógica de negocio dentro de `DependencyInjection.cs`; su única responsabilidad es registrar servicios, nunca ejecutarlos.
- Mantén un único método de extensión por capa (`AddApplication`, `AddInfrastructure`) para que `Program.cs` describa el arranque del microservicio en pocas líneas, sin importar cuánto crezca cada capa por dentro.
- El orden entre `AddApplication()` y `AddInfrastructure(...)` en `Program.cs` no afecta el resultado, pero mantén siempre el mismo orden en todos tus microservicios para que el `Program.cs` se lea igual en todo el proyecto.

## Recursos relacionados

- [Configuración mínima de conexión a la BD](crear-conexion-bd.md)
- [Crear una entidad](crear-entidad.md)
- [Crear un CRUD](crear-crud.md)
- [Clean Architecture](../arquitectura/clean-architecture.md)

[⬅ Volver al índice de .NET](README.md)
