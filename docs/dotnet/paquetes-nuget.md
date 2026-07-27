# Gestión de paquetes NuGet

## Tabla de contenido
- [¿Qué es?](#qué-es)
- [¿Para qué sirve?](#para-qué-sirve)
- [Comandos básicos](#comandos-básicos)
- [Paquetes esenciales recomendados por capa](#paquetes-esenciales-recomendados-por-capa)
- [Buenas prácticas](#buenas-prácticas)
- [Errores comunes](#errores-comunes)
- [Recursos relacionados](#recursos-relacionados)

## ¿Qué es?

**NuGet** es el gestor de paquetes oficial de .NET. Permite instalar, actualizar y eliminar librerías de terceros (o de Microsoft) en un proyecto específico dentro de la solución.

## ¿Para qué sirve?

En una arquitectura por capas, **no todas las capas necesitan los mismos paquetes**. Gestionar correctamente qué paquete va en qué proyecto es clave para no romper la regla de dependencias de Clean Architecture (por ejemplo, `Domain` no debería depender de EF Core).

## Comandos básicos

### Instalar un paquete

```bash
dotnet add package AutoMapper
dotnet add package AutoMapper --version 12.0.1
```

### Eliminar un paquete

```bash
dotnet remove package AutoMapper
```

### Ver paquetes instalados

```bash
dotnet list package
dotnet list package --include-transitive
```

### Instalar un paquete en un proyecto específico desde la solución

```bash
dotnet add ./MicroservicioUsuarios/API package Refit
```

Otro ejemplo, instalando el proveedor de PostgreSQL en la capa de Infraestructura de un proyecto llamado `SIGESUP`:

```bash
dotnet add SIGESUP.Infrastructure package Npgsql.EntityFrameworkCore.PostgreSQL
```

## Paquetes esenciales recomendados por capa

| Capa | Paquete | Propósito |
|---|---|---|
| **Domain** | *(ninguno)* | El dominio no debe depender de librerías externas, por principio de Clean Architecture |
| **Application** | `MediatR` | Implementar el patrón mediador para Commands/Queries (CQRS) |
| **Application** | `FluentValidation` | Validar los Commands/Queries antes de ejecutarlos |
| **Application** | `AutoMapper` | Mapear entre entidades del dominio y DTOs |
| **Infrastructure** | `Microsoft.EntityFrameworkCore` | ORM para acceso a datos |
| **Infrastructure** | `Microsoft.EntityFrameworkCore.SqlServer` (o `Npgsql.EntityFrameworkCore.PostgreSQL` para PostgreSQL) | Proveedor de base de datos específico |
| **Infrastructure** | `Microsoft.EntityFrameworkCore.Tools` | Herramientas de CLI para migraciones |
| **API** | `MediatR.Extensions.Microsoft.DependencyInjection` | Registrar MediatR en el contenedor de DI de la API |

Instalación completa de ejemplo:

```bash
# Application
dotnet add ./MicroservicioUsuarios/Application/Application.csproj package MediatR
dotnet add ./MicroservicioUsuarios/Application/Application.csproj package FluentValidation
dotnet add ./MicroservicioUsuarios/Application/Application.csproj package AutoMapper

# Infrastructure
dotnet add ./MicroservicioUsuarios/Infrastructure/Infrastructure.csproj package Microsoft.EntityFrameworkCore
dotnet add ./MicroservicioUsuarios/Infrastructure/Infrastructure.csproj package Microsoft.EntityFrameworkCore.SqlServer
dotnet add ./MicroservicioUsuarios/Infrastructure/Infrastructure.csproj package Microsoft.EntityFrameworkCore.Tools

# API
dotnet add ./MicroservicioUsuarios/API/API.csproj package MediatR.Extensions.Microsoft.DependencyInjection
```

## Buenas prácticas

- Fija versiones explícitas (`--version`) en proyectos productivos para evitar romper compatibilidad con actualizaciones automáticas.
- Ejecuta `dotnet list package --outdated` periódicamente para detectar paquetes desactualizados.
- Evita instalar paquetes de infraestructura (EF Core, proveedores de base de datos) en `Domain` o `Application`.

## Errores comunes

- ❌ Instalar `Microsoft.EntityFrameworkCore` en la capa `Domain`, rompiendo el aislamiento del núcleo del negocio.
- ❌ No especificar el proyecto destino (`dotnet add package` sin ruta) cuando existen varios `.csproj` en el directorio, lo que puede instalar el paquete en el proyecto incorrecto.
- ❌ Mezclar versiones distintas del mismo paquete entre las capas de un mismo microservicio.

## Recursos relacionados

- [Crear una solución de microservicios](crear-solucion-microservicios.md)
- [Clean Architecture](../arquitectura/clean-architecture.md)
- [Entity Framework Core: migraciones](entity-framework-migraciones.md)

[⬅ Volver al índice de .NET](README.md)
