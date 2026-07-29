# Configuración mínima de conexión de un microservicio a la base de datos

## Tabla de contenido
- [¿Para qué sirve?](#para-qué-sirve)
- [1. Instalar los paquetes NuGet](#1-instalar-los-paquetes-nuget)
- [2. Crear la estructura de la carpeta Persistence](#2-crear-la-estructura-de-la-carpeta-persistence)
- [3. Crear el DbContext](#3-crear-el-dbcontext)
- [4. Crear DependencyInjection](#4-crear-dependencyinjection)
- [5. Agregar la cadena de conexión](#5-agregar-la-cadena-de-conexión)
- [6. Registrar Infrastructure](#6-registrar-infrastructure)
- [Buenas prácticas](#buenas-prácticas)
- [Recursos relacionados](#recursos-relacionados)

## ¿Para qué sirve?

Este documento cubre la **configuración mínima necesaria** para conectar la capa `Infrastructure` de un microservicio a una base de datos real a través de **Entity Framework Core**: instalar el proveedor correspondiente, crear el `DbContext`, registrar la infraestructura en el contenedor de dependencias y definir la cadena de conexión.

Es el paso previo indispensable antes de poder trabajar con [migraciones de Entity Framework Core](entity-framework-migraciones.md), ya que sin un `DbContext` correctamente registrado no es posible generar ni aplicar migraciones.

## 1. Instalar los paquetes NuGet

En la capa `Infrastructure`, instala EF Core junto con el proveedor de base de datos correspondiente (en este ejemplo, PostgreSQL):

```bash
dotnet add Infrastructure package Microsoft.EntityFrameworkCore
dotnet add Infrastructure package Npgsql.EntityFrameworkCore.PostgreSQL
```

> **Nota**
> Si usas SQL Server en lugar de PostgreSQL, instala `Microsoft.EntityFrameworkCore.SqlServer` en su lugar (ver [Gestión de paquetes NuGet](paquetes-nuget.md)).

En `API`, asegúrate de tener la referencia al proyecto `Infrastructure` (y las demás referencias entre capas). Si ya seguiste la guía de [creación de la solución](crear-solucion-microservicios.md), estas referencias ya deberían existir:

```bash
dotnet add src/MiProyecto.API reference src/MiProyecto.Infrastructure
dotnet add src/MiProyecto.Infrastructure reference src/MiProyecto.Application
dotnet add src/MiProyecto.Application reference src/MiProyecto.Domain
```

## 2. Crear la estructura de la carpeta Persistence

```text
Infrastructure/
├── Persistence/
│   └── AppDbContext.cs
└── DependencyInjection.cs
```

> **Tip**
> Esta es la estructura mínima para levantar la conexión. A medida que agregues entidades, esta carpeta también alojará `Configurations/` (para las clases `IEntityTypeConfiguration`) y `Repositories/`, tal como se describe en [Estructura de carpetas por capa](../arquitectura/estructura-carpetas.md).

## 3. Crear el DbContext

Archivo donde se registran las configuraciones y los `DbSet` de las entidades:

```csharp
using Microsoft.EntityFrameworkCore;

namespace MiProyecto.Infrastructure.Persistence;

public class AppDbContext(
    DbContextOptions<AppDbContext> options
) : DbContext(options)
{
}
```

> **Nota**
> Este ejemplo usa **constructores primarios** (disponibles desde C# 12 / .NET 8). Si tu proyecto usa una versión anterior de .NET, declara el constructor de forma tradicional:
> ```csharp
> public class AppDbContext : DbContext
> {
>     public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
> }
> ```

## 4. Crear DependencyInjection

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

## 5. Agregar la cadena de conexión

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

## 6. Registrar Infrastructure

En `API/Program.cs`:

```csharp
using MiProyecto.Infrastructure;

builder.Services.AddInfrastructure(builder.Configuration);
```

## Buenas prácticas

- Mantén el nombre de la clave de la cadena de conexión (`DefaultConnection`) consistente entre todos los microservicios, salvo que necesites distinguir varias bases de datos en un mismo servicio.
- No agregues lógica de negocio dentro de `AppDbContext`; su única responsabilidad es representar el modelo de persistencia.
- Una vez configurada la conexión, continúa con la generación de la primera migración (ver [Entity Framework Core: migraciones](entity-framework-migraciones.md)).

## Recursos relacionados

- [Crear una solución de microservicios](crear-solucion-microservicios.md)
- [Gestión de paquetes NuGet](paquetes-nuget.md)
- [Clean Architecture](../arquitectura/clean-architecture.md)
- [Entity Framework Core: migraciones](entity-framework-migraciones.md)

[⬅ Volver al índice de .NET](README.md)
