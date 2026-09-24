# Registro de MediatR y AutoMapper

> **Nota**
> Este flujo asume que ya instalaste los paquetes de la capa `Application` (`MediatR`, `AutoMapper`), revisa primero [Gestión de paquetes NuGet](3-paquetes-nuget.md).

Este documento describe la configuración **única por microservicio** de la capa `Application`: registrar **MediatR** y **AddAutoMapper** en el contenedor de dependencias para poder despachar Commands y Queries desde el controlador (`IMediator.Send(...)`).

Este paso se hace **una sola vez** al iniciar el microservicio: una vez registrado, MediatR detecta automáticamente todos los Commands, Queries y Handlers nuevos que agregues más adelante, sin necesidad de volver a tocar este archivo.

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

        services.AddAutoMapper(Assembly.GetExecutingAssembly());

        return services;
    }
}
```

> **Tip**
> `RegisterServicesFromAssembly(Assembly.GetExecutingAssembly())` escanea automáticamente el proyecto `Application` en busca de clases que implementen `IRequestHandler<,>`.
> `services.AddAutoMapper(Assembly.GetExecutingAssembly());` escanea el assembly completo de `Application` en busca de clases que hereden de `Profile`.

## 2. Registrar Application

En `API/Program.cs`, se registra el método `AddApplication()` creado en el paso anterior. Junto con el registro de `Infrastructure` ([Conexión a la base de datos](4-conexion-bd.md)), el archivo queda así:

```csharp
using MiProyecto.Application;
using MiProyecto.Infrastructure;

builder.Services.AddControllers();
builder.Services.AddApplication();
builder.Services.AddInfrastructure(builder.Configuration);

var app = builder.Build();

app.MapControllers();
```

## Buenas prácticas

- No agregues lógica de negocio dentro de `DependencyInjection.cs`; su única responsabilidad es registrar servicios, nunca ejecutarlos.
- Mantén un único método de extensión por capa (`AddApplication`, `AddInfrastructure`) para que `Program.cs` describa el arranque del microservicio en pocas líneas, sin importar cuánto crezca cada capa por dentro.
- El orden entre `AddApplication()` y `AddInfrastructure(...)` en `Program.cs` no afecta el resultado, pero mantén siempre el mismo orden en todos tus microservicios para que el `Program.cs` se lea igual en todo el proyecto.

[⬅](4-conexion-bd.md) Conexión a la base de datos
