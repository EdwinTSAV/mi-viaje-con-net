# Configuración mínima de conexión de un microservicio a la base de datos

## Tabla de contenido
- [¿Para qué sirve?](#para-qué-sirve)
- [1. Crear la estructura de la carpeta Persistence](#1-crear-la-estructura-de-la-carpeta-persistence)
- [2. Crear el DbContext](#2-crear-el-dbcontext)
- [3. Crear DependencyInjection](#3-crear-dependencyinjection)
- [4. Agregar la cadena de conexión](#4-agregar-la-cadena-de-conexión)
- [5. Registrar Infrastructure](#5-registrar-infrastructure)
- [Buenas prácticas](#buenas-prácticas)
- [Recursos relacionados](#recursos-relacionados)

## ¿Para qué sirve?

Este documento cubre la **configuración mínima necesaria** para conectar la capa `Infrastructure` de un microservicio a una base de datos real a través de **Entity Framework Core**: instalar el proveedor correspondiente, crear el `DbContext`, registrar la infraestructura en el contenedor de dependencias y definir la cadena de conexión.

## 1. Crear la estructura de la carpeta Persistence

```text
Infrastructure/
├── Persistence/
│   └── AppDbContext.cs
└── DependencyInjection.cs
```

## 2. Crear el DbContext

Archivo donde se registran las configuraciones y los `DbSet` de las entidades:

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

> **Tip**
> Cuando el proyecto tenga varias entidades, en lugar de agregar una línea `ApplyConfiguration(...)` por cada una, se puede escanear todo el assembly automáticamente:
> ```csharp
> modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
> ```

## 3. Crear DependencyInjection

Archivo donde se registra toda la infraestructura del proyecto:

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

## 4. Agregar la cadena de conexión

En `API/appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=MiBD;Username=postgres;Password=123456"
  }
}
```

> **Advertencia**
> El ejemplo anterior es válido únicamente para un entorno **local de desarrollo**. Nunca subas contraseñas reales al repositorio dentro de `appsettings.json`. Para entornos de staging o producción, usa `appsettings.Development.json` (ignorado en Git) para credenciales locales, y **Azure Key Vault** o las **App Settings** del servicio de hosting para credenciales productivas, tal como se indica en [Despliegue en Azure App Service](../azure/app-service-deployment.md#paso-4--configurar-el-connection-string).

## 5. Registrar Infrastructure

En `API/Program.cs`:

```csharp
using MiProyecto.Infrastructure;

builder.Services.AddInfrastructure(builder.Configuration);
```

## Buenas prácticas

- Mantén el nombre de la clave de la cadena de conexión (`DefaultConnection`) consistente entre todos los microservicios, salvo que necesites distinguir varias bases de datos en un mismo servicio.
- No agregues lógica de negocio dentro de `AppDbContext`; su única responsabilidad es representar el modelo de persistencia.

## Recursos relacionados

- [Crear un microservicio](2-crear-microservicio.md)
- [Gestión de paquetes NuGet](3-paquetes-nuget.md)
- [Clean Architecture](../arquitectura/clean-architecture.md)
- [Entity Framework Core: migraciones](entity-framework-migraciones.md)

⬅ [Volver al índice](README.md)
