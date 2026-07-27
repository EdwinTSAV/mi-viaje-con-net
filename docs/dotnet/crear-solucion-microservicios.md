# Crear una solución de microservicios con .NET CLI

## Tabla de contenido
- [¿Qué es?](#qué-es)
- [¿Para qué sirve?](#para-qué-sirve)
- [1. Crear la solución general](#1-crear-la-solución-general)
- [2. Crear la estructura de un microservicio](#2-crear-la-estructura-de-un-microservicio)
- [3. Referenciar proyectos entre capas](#3-referenciar-proyectos-entre-capas)
- [4. Eliminar una referencia](#4-eliminar-una-referencia)
- [Ejemplo completo (microservicio "Usuarios")](#ejemplo-completo-microservicio-usuarios)
- [Buenas prácticas](#buenas-prácticas)
- [Errores comunes](#errores-comunes)
- [Recursos relacionados](#recursos-relacionados)

## ¿Qué es?

Es el flujo de comandos de `dotnet CLI` para inicializar una solución (`.sln`) que contendrá **múltiples microservicios**, cada uno estructurado en las cuatro capas de Clean Architecture (`API`, `Application`, `Domain`, `Infrastructure`).

## ¿Para qué sirve?

Permite levantar rápidamente el esqueleto de un nuevo microservicio de forma consistente, asegurando que las referencias entre proyectos respeten la [regla de dependencias](../arquitectura/clean-architecture.md#regla-principal-de-dependencias) de Clean Architecture desde el primer commit.

## 1. Crear la solución general

La solución (`.sln`) actúa como contenedor de todos los microservicios del sistema:

```bash
dotnet new sln -n ProyectoMicroservicios
cd ProyectoMicroservicios
```

## 2. Crear la estructura de un microservicio

Por cada microservicio se crea una carpeta contenedora y, dentro de ella, los cuatro proyectos de capa.

Ejemplo con un microservicio de **Usuarios**:

```bash
# Crear la carpeta del microservicio
mkdir MicroservicioUsuarios
cd MicroservicioUsuarios

# Crear los proyectos por capa
dotnet new webapi -n API                # Capa API
dotnet new classlib -n Application      # Capa Application
dotnet new classlib -n Domain           # Capa Domain
dotnet new classlib -n Infrastructure   # Capa Infrastructure
```

> **Nota**
> `API` se genera con la plantilla `webapi` porque expone endpoints HTTP. Las demás capas (`Application`, `Domain`, `Infrastructure`) se generan como `classlib`, ya que son bibliotecas de clases sin punto de entrada propio.

## 3. Referenciar proyectos entre capas

### 3.1 — Agregar los proyectos a la solución general

Desde la raíz donde está el archivo `.sln`:

```bash
dotnet sln Backend.sln add ./MicroservicioUsuarios/Domain/Domain.csproj
dotnet sln Backend.sln add ./MicroservicioUsuarios/Application/Application.csproj
dotnet sln Backend.sln add ./MicroservicioUsuarios/Infrastructure/Infrastructure.csproj
dotnet sln Backend.sln add ./MicroservicioUsuarios/Api/Api.csproj
```

### 3.2 — Referenciar entre proyectos respetando Clean Architecture

Ejecutado desde la raíz de la solución (`solucion.sln`):

```bash
# Application → Domain
dotnet add ./MicroservicioUsuarios/Application/Application.csproj reference ./MicroservicioUsuarios/Domain/Domain.csproj

# Infrastructure → Application
dotnet add ./MicroservicioUsuarios/Infrastructure/Infrastructure.csproj reference ./MicroservicioUsuarios/Application/Application.csproj

# Infrastructure → Domain
dotnet add ./MicroservicioUsuarios/Infrastructure/Infrastructure.csproj reference ./MicroservicioUsuarios/Domain/Domain.csproj

# API → Application
dotnet add ./MicroservicioUsuarios/API/API.csproj reference ./MicroservicioUsuarios/Application/Application.csproj

# API → Infrastructure (necesario para registrar implementaciones en DI)
dotnet add ./MicroservicioUsuarios/API/API.csproj reference ./MicroservicioUsuarios/Infrastructure/Infrastructure.csproj
```

> **Importante**
> Sin la referencia `API → Infrastructure` **no es posible** registrar las implementaciones concretas (repositorios, `DbContext`, servicios externos) en el contenedor de dependencias dentro de `Program.cs`, ya que es en la capa API donde se ensamblan todas las dependencias de la aplicación.

Puedes revisar de forma alternativa la referencia genérica de un `.csproj` a otro con:

```bash
dotnet reference add lib/lib.csproj --project app/app.csproj
```

## 4. Eliminar una referencia

Si necesitas revertir una referencia entre proyectos:

```bash
dotnet remove ./MicroservicioUsuarios/API/API.csproj reference ./MicroservicioUsuarios/Infrastructure/Infrastructure.csproj
```

## Ejemplo completo (microservicio "Usuarios")

```bash
# 1. Crear la solución
dotnet new sln -n ProyectoMicroservicios
cd ProyectoMicroservicios

# 2. Crear carpeta del microservicio
mkdir MicroservicioUsuarios && cd MicroservicioUsuarios

# 3. Crear proyectos
dotnet new webapi -n MicroservicioUsuarios.API
dotnet new classlib -n MicroservicioUsuarios.Application
dotnet new classlib -n MicroservicioUsuarios.Domain
dotnet new classlib -n MicroservicioUsuarios.Infrastructure

# 4. Volver a la raíz y agregar a la solución
cd ..
dotnet sln ProyectoMicroservicios.sln add ./MicroservicioUsuarios/MicroservicioUsuarios.Domain/MicroservicioUsuarios.Domain.csproj
dotnet sln ProyectoMicroservicios.sln add ./MicroservicioUsuarios/MicroservicioUsuarios.Application/MicroservicioUsuarios.Application.csproj
dotnet sln ProyectoMicroservicios.sln add ./MicroservicioUsuarios/MicroservicioUsuarios.Infrastructure/MicroservicioUsuarios.Infrastructure.csproj
dotnet sln ProyectoMicroservicios.sln add ./MicroservicioUsuarios/MicroservicioUsuarios.API/MicroservicioUsuarios.API.csproj

# 5. Referenciar entre capas (ver sección 3.2)
```

## Buenas prácticas

- Usa un prefijo consistente por microservicio en el nombre de los proyectos (por ejemplo, `MicroservicioUsuarios.API`) para evitar confusiones cuando la solución crezca.
- Automatiza este flujo con un script (bash o PowerShell) si vas a crear varios microservicios con la misma estructura.
- Verifica las referencias con `dotnet list <proyecto> reference` después de cada cambio importante.

## Errores comunes

- ❌ Referenciar `Domain` desde `API` directamente, saltándose `Application`.
- ❌ Olvidar agregar los proyectos a la solución (`dotnet sln add`), lo que impide que aparezcan al abrir la solución en el IDE.
- ❌ Crear `Domain` con la plantilla `webapi` en lugar de `classlib`.

## Recursos relacionados

- [Clean Architecture](../arquitectura/clean-architecture.md)
- [Gestión de paquetes NuGet](paquetes-nuget.md)

[⬅ Volver al índice de .NET](README.md)
