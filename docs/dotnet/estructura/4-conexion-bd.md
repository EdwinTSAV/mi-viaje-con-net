# Configuración de conexión de un microservicio a la base de datos

> **Nota**
> Este flujo asume que ya tienes la estructura y los paquetes de un microservicio, revisa primero [Crear un microservicio](2-microservicio.md) y [Gestión de paquetes NuGet](3-paquetes-nuget.md).

Este documento cubre la **configuración mínima necesaria** para conectar la capa `Infrastructure` de un microservicio a una base de datos real a través de **Entity Framework Core**.

## 1. Crear el DbContext

En `Infrastructure/Persistence/AppDbContext.cs` :

```csharp
using Microsoft.EntityFrameworkCore;

namespace MiProyecto.Infrastructure.Persistence;

public class AppDbContext(
    DbContextOptions<AppDbContext> options
) : DbContext(options)
{
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
    }
}
```

## 2. Crear DependencyInjection

En `Infrastructure/DependencyInjection.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using MiProyecto.Infrastructure.Persistence;

namespace MiProyecto.Infrastructure;

public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddDbContext<AppDbContext>(options =>
        {
            // Cambia el proveedor dependiendo de la BD (aquí, PostgreSQL)
            options.UseNpgsql(
                configuration.GetConnectionString("DefaultConnection"));
        });

        return services;
    }
}
```

## 3. Agregar la cadena de conexión

En `API/appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=MiBD;Username=postgres;Password=123456"
  }
}
```

> **Advertencia**
> El ejemplo anterior es válido únicamente para un entorno **local de desarrollo**. Nunca subas contraseñas reales al repositorio dentro de `appsettings.json`. Para credenciales locales usa `dotnet user-secrets` (ver abajo), y para staging o producción usa **Azure Key Vault** o las **App Settings** del servicio de hosting.

### Guardar la cadena de conexión con user-secrets

Desde la raíz donde está el archivo `.slnx`:

```bash
dotnet user-secrets init --project ./MicroservicioPersona/API/MicroservicioPersona.API.csproj

dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Host=localhost;Port=5432;Database=MiBD;Username=postgres;Password=<tu-contraseña>" --project ./MicroservicioPersona/API/MicroservicioPersona.API.csproj
```

> **Nota**
> Los secretos se guardan fuera del repositorio, en el perfil de tu usuario, y solo se cargan cuando el entorno es `Development`. Si defines la misma clave en `appsettings.json` y en user-secrets, el valor de user-secrets es el que se usa.

## 4. Registrar Infrastructure

En `API/Program.cs`:

```csharp
using MiProyecto.Infrastructure;

builder.Services.AddInfrastructure(builder.Configuration);
```

## Buenas prácticas

- Mantén el nombre de la clave de la cadena de conexión (`DefaultConnection`) consistente entre todos los microservicios, salvo que necesites distinguir varias bases de datos en un mismo servicio.
- No agregues lógica de negocio dentro de `AppDbContext`; su única responsabilidad es representar el modelo de persistencia.

[⬅](3-paquetes-nuget.md) Gestión de paquetes NuGet | Registro de MediatR y AutoMapper [➡](5-registro-mediatr.md)
