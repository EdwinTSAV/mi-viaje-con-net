# Crear un microservicio

> **Nota**
> Este flujo asume que ya tienes: la solución creada, revisa primero [Crear la solución general](1-solucion-general.md).

Los siguientes comandos deben ejecutarse desde la raíz donde está el archivo `.slnx`:

## 1. Crear la estructura

```bash
dotnet new webapi -n MicroservicioPersona.API -o MicroservicioPersona/API --use-controllers         # Capa API
dotnet new classlib -n MicroservicioPersona.Application -o MicroservicioPersona/Application         # Capa Application
dotnet new classlib -n MicroservicioPersona.Domain -o MicroservicioPersona/Domain                   # Capa Domain
dotnet new classlib -n MicroservicioPersona.Infrastructure -o MicroservicioPersona/Infrastructure   # Capa Infrastructure
```

> **Nota**
> `API` se genera con la plantilla `webapi` porque expone endpoints HTTP. La opción `--use-controllers` es necesaria porque, desde .NET 8, la plantilla genera Minimal APIs por defecto, y los siguientes documentos usan controladores (`AddControllers()` / `MapControllers()`). Las demás capas (`Application`, `Domain`, `Infrastructure`) se generan como `classlib`, ya que son bibliotecas de clases sin punto de entrada propio.
>
> La opción `-o` define la **carpeta** de destino, mientras que `-n` define el **nombre del proyecto** y del archivo `.csproj` generado (con el prefijo explícito). Por eso el archivo resultante para la capa API es `MicroservicioPersona/API/MicroservicioPersona.API.csproj`, y no `API.csproj`.

## 2. Agregar los proyectos a la solución general

```bash
dotnet sln SolucionGeneral.slnx add ./MicroservicioPersona/Domain/MicroservicioPersona.Domain.csproj
dotnet sln SolucionGeneral.slnx add ./MicroservicioPersona/Application/MicroservicioPersona.Application.csproj
dotnet sln SolucionGeneral.slnx add ./MicroservicioPersona/Infrastructure/MicroservicioPersona.Infrastructure.csproj
dotnet sln SolucionGeneral.slnx add ./MicroservicioPersona/API/MicroservicioPersona.API.csproj
```

## 3. Referenciar entre proyectos respetando Clean Architecture

```bash
# Application → Domain
dotnet add ./MicroservicioPersona/Application/MicroservicioPersona.Application.csproj reference ./MicroservicioPersona/Domain/MicroservicioPersona.Domain.csproj

# Infrastructure → Application
dotnet add ./MicroservicioPersona/Infrastructure/MicroservicioPersona.Infrastructure.csproj reference ./MicroservicioPersona/Application/MicroservicioPersona.Application.csproj

# Infrastructure → Domain
dotnet add ./MicroservicioPersona/Infrastructure/MicroservicioPersona.Infrastructure.csproj reference ./MicroservicioPersona/Domain/MicroservicioPersona.Domain.csproj

# API → Application
dotnet add ./MicroservicioPersona/API/MicroservicioPersona.API.csproj reference ./MicroservicioPersona/Application/MicroservicioPersona.Application.csproj

# API → Infrastructure (necesario para registrar implementaciones en DI)
dotnet add ./MicroservicioPersona/API/MicroservicioPersona.API.csproj reference ./MicroservicioPersona/Infrastructure/MicroservicioPersona.Infrastructure.csproj
```

> **Importante**
> Sin la referencia `API → Infrastructure` **no es posible** registrar las implementaciones concretas (repositorios, `DbContext`, servicios externos) en el contenedor de dependencias dentro de `Program.cs`, ya que es en la capa API donde se ensamblan todas las dependencias de la aplicación.

### Eliminar una referencia

Si necesitas revertir una referencia entre proyectos:

```bash
dotnet remove ./MicroservicioPersona/API/MicroservicioPersona.API.csproj reference ./MicroservicioPersona/Infrastructure/MicroservicioPersona.Infrastructure.csproj
```

## 4. Verificar que compila

```bash
dotnet build SolucionGeneral.slnx
```

## Buenas prácticas

- Usa un prefijo consistente por microservicio en el nombre de los **proyectos** (`-n MicroservicioPersona.API`), no necesariamente en el nombre de la carpeta (`-o`), ya que la carpeta ya queda anidada bajo `MicroservicioPersona/` y el prefijo ahí sería redundante.
- Automatiza este flujo con un script (bash o PowerShell) si vas a crear varios microservicios con la misma estructura.
- Verifica las referencias con `dotnet list <proyecto> reference` después de cada cambio importante.

## Errores comunes

- Olvidar agregar los proyectos a la solución (`dotnet sln add`), lo que impide que aparezcan al abrir la solución en el IDE.
- Crear `Domain` con la plantilla `webapi` en lugar de `classlib`.
- Crear `API` sin `--use-controllers`: el proyecto arranca, pero `AddControllers()` / `MapControllers()` no tienen controladores que mapear y la plantilla incluye endpoints de ejemplo con Minimal APIs.

[⬅](1-solucion-general.md) Crear la solución general | Gestión de paquetes NuGet [➡](3-paquetes-nuget.md)
