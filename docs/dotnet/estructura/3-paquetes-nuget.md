# Gestión de paquetes NuGet

> **Nota**
> Este flujo asume que ya tienes: la estructura del microservicio creado, revisa primero [Crear un microservicio](2-microservicio.md).

**NuGet** es el gestor de paquetes oficial de .NET. Permite instalar, actualizar y eliminar librerías de terceros (o de Microsoft) en un proyecto específico dentro de la solución.

## ¿Para qué sirve?

En una arquitectura por capas, **no todas las capas necesitan los mismos paquetes**. Gestionar correctamente qué paquete va en qué proyecto es clave para no romper la regla de dependencias de Clean Architecture (por ejemplo, `Domain` no debería depender de EF Core).

## Comandos básicos

Siempre indica el proyecto destino (`.csproj`); los comandos se ejecutan desde la raíz donde está el archivo `.slnx`:

```bash
# Instalar un paquete en un proyecto (la versión es opcional)
dotnet add <ruta-al-proyecto>.csproj package <NombrePaquete> --version <x.y.z>

# Eliminar un paquete de un proyecto
dotnet remove <ruta-al-proyecto>.csproj package <NombrePaquete>

# Ver los paquetes instalados en un proyecto
dotnet list <ruta-al-proyecto>.csproj package
dotnet list <ruta-al-proyecto>.csproj package --include-transitive
```

## Paquetes esenciales recomendados por capa

| Capa | Paquete | Propósito |
|---|---|---|
| **Domain** | *(ninguno)* | El dominio no debe depender de librerías externas, por principio de Clean Architecture |
| **Application** | `MediatR` | Implementar el patrón mediador para Commands/Queries (CQRS) |
| **Application** | `FluentValidation` | Validar los Commands/Queries antes de ejecutarlos |
| **Application** | `AutoMapper` | Mapear entre entidades del dominio y DTOs |
| **Infrastructure** | `Microsoft.EntityFrameworkCore` | ORM para acceso a datos |
| **Infrastructure** | `Npgsql.EntityFrameworkCore.PostgreSQL` (o `Microsoft.EntityFrameworkCore.SqlServer` para SQL Server) | Proveedor de base de datos específico |
| **Infrastructure** | `Microsoft.EntityFrameworkCore.Tools` | Comandos de migraciones para la consola del administrador de paquetes (`Add-Migration`, `Update-Database`) |

Instalación completa, desde la raíz donde está el archivo `.slnx`:

```bash
# Application
dotnet add ./MicroservicioPersona/Application/MicroservicioPersona.Application.csproj package MediatR
dotnet add ./MicroservicioPersona/Application/MicroservicioPersona.Application.csproj package FluentValidation
dotnet add ./MicroservicioPersona/Application/MicroservicioPersona.Application.csproj package AutoMapper --version 14.0.0

# Infrastructure
dotnet add ./MicroservicioPersona/Infrastructure/MicroservicioPersona.Infrastructure.csproj package Microsoft.EntityFrameworkCore
dotnet add ./MicroservicioPersona/Infrastructure/MicroservicioPersona.Infrastructure.csproj package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add ./MicroservicioPersona/Infrastructure/MicroservicioPersona.Infrastructure.csproj package Microsoft.EntityFrameworkCore.Tools
```

## Buenas prácticas

- Fija versiones explícitas (`--version`) en proyectos productivos para evitar romper compatibilidad con actualizaciones automáticas.
- Ejecuta `dotnet list package --outdated` periódicamente para detectar paquetes desactualizados.
- Evita instalar paquetes de infraestructura (EF Core, proveedores de base de datos) en `Domain` o `Application`.

## Errores comunes

- Instalar `Microsoft.EntityFrameworkCore` en la capa `Domain`, rompiendo el aislamiento del núcleo del negocio.
- No especificar el proyecto destino (`dotnet add package` sin ruta) cuando existen varios `.csproj` en el directorio, lo que puede instalar el paquete en el proyecto incorrecto.
- Usar el nombre corto del archivo (`Application.csproj`) en lugar del nombre real del proyecto (`MicroservicioPersona.Application.csproj`); revisa cómo se generan los nombres en [Crear un microservicio](2-microservicio.md).
- Mezclar versiones distintas del mismo paquete entre las capas de un mismo microservicio.
- Instalar paquetes de EF Core (`Microsoft.EntityFrameworkCore`, el proveedor y `Tools`) con versiones mayores distintas entre sí; deben compartir la misma versión mayor.

[⬅](2-microservicio.md) Crear un microservicio | Conexión a la base de datos [➡](4-conexion-bd.md)
