# Entity Framework Core: Migraciones

## Tabla de contenido
- [¿Qué es?](#qué-es)
- [¿Para qué sirve?](#para-qué-sirve)
- [Parámetros clave](#parámetros-clave)
- [Migraciones en proyectos con Clean Architecture](#migraciones-en-proyectos-con-clean-architecture)
- [Migraciones en proyectos sin Clean Architecture](#migraciones-en-proyectos-sin-clean-architecture)
- [Generar un script SQL de migración](#generar-un-script-sql-de-migración)
- [Eliminar migraciones](#eliminar-migraciones)
- [Buenas prácticas](#buenas-prácticas)
- [Errores comunes](#errores-comunes)
- [Recursos relacionados](#recursos-relacionados)

## ¿Qué es?

Las **migraciones de Entity Framework Core** son una forma de versionar los cambios del modelo de datos (entidades del `DbContext`) y aplicarlos de manera controlada e incremental a la base de datos.

## ¿Para qué sirve?

Permite mantener sincronizado el esquema de la base de datos con el modelo de dominio definido en código, sin necesidad de escribir manualmente sentencias `CREATE TABLE` o `ALTER TABLE`.

> **Nota**
> En Clean Architecture, el `DbContext` debe ubicarse en la capa `Infrastructure`, mientras que el **proyecto de arranque** (`--startup-project`) siempre apunta a `API`, ya que es allí donde se configura la cadena de conexión y se resuelve la inyección de dependencias.

## Parámetros clave

| Parámetro | Significado |
|---|---|
| `--project` | 📦 Proyecto que contiene el `DbContext` (normalmente `Infrastructure`) |
| `--startup-project` | 🚀 Proyecto que arranca la aplicación, donde está `Program.cs` (normalmente `API`) |
| `InitialCreate` / `Initial` | 🧱 Nombre descriptivo de la migración |

## Migraciones en proyectos con Clean Architecture

### Desde fuera de la carpeta `Infrastructure`

### Ver el `DbContext` disponible (opcional, antes de migrar)

```bash
dotnet ef dbcontext list --project SIGESUP.Infrastructure --startup-project SIGESUP.API
```

### Crear una migración

```bash
dotnet ef migrations add InitialCreate --project SIGESUP.Infrastructure --startup-project SIGESUP.API
```

### Generar un script SQL de migración

Útil para entregar el script a un DBA o aplicarlo manualmente en un entorno sin acceso directo desde el CLI:

```bash
dotnet ef --startup-project ../SIGESUP.API/SIGESUP.API.csproj migrations script --idempotent -o MigrationScripts/migrationSIGESUP.sql
```

> **Tip**
> La bandera `--idempotent` genera un script que puede ejecutarse varias veces sin generar errores, ya que verifica qué migraciones ya fueron aplicadas antes de ejecutar cada bloque.

### Aplicar la migración directamente a la base de datos

```bash
dotnet ef database update --project src/SIGESUP.Infrastructure --startup-project src/SIGESUP.API
```

### Eliminar migraciones

```bash
dotnet ef migrations remove --project SIGESUP.Infrastructure --startup-project SIGESUP.API
```

> **Advertencia**
> `dotnet ef migrations remove` solo elimina la **última** migración generada y **no aplicada aún** a la base de datos. Si la migración ya fue aplicada, primero debes revertirla con `dotnet ef database update <MigraciónAnterior>` antes de eliminarla del código.

### Alternativa: ejecutar los comandos desde dentro de la carpeta `Infrastructure`

```bash
cd ./MicroservicioUsuarios/Infrastructure

# Generar la migración
dotnet ef migrations add Initial --startup-project ../API

# Generar un script SQL de migración
dotnet ef migrations script --idempotent -o MigrationScript/migration.sql --startup-project ../API/

# Aplicar la migración directamente a la base de datos
dotnet ef --startup-project ../API database update

# Eliminar migraciones
dotnet ef migrations remove --startup-project ../SIGESUP.API
```

### Alternativa: especificando un directorio de salida para las migraciones

```bash
dotnet ef migrations add Initial1.0.0 \
  --project SIGESUP.Infrastructure \
  --startup-project SIGESUP.API \
  --output-dir Persistence/Migrations
```

## Buenas prácticas

- Nombra las migraciones de forma descriptiva (`AddEmailToUsuario`, no `Migration2`).
- Genera siempre un script SQL (`--idempotent`) antes de aplicar cambios en producción, para poder revisarlo previamente.
- Versiona las migraciones junto con el código fuente en el control de versiones (Git).
- Nunca elimines manualmente archivos de migración; usa siempre `dotnet ef migrations remove`.

## Errores comunes

- ❌ Ejecutar `dotnet ef migrations add` sin especificar `--startup-project` en un proyecto con Clean Architecture, lo que provoca errores de configuración porque el `DbContext` no encuentra la cadena de conexión.
- ❌ Aplicar `database update` directamente en producción sin revisar antes el script SQL generado.
- ❌ Eliminar una migración ya aplicada a la base de datos sin revertirla primero.

## Recursos relacionados

- [Gestión de paquetes NuGet](paquetes-nuget.md)
- [Publicar un microservicio](publicar-proyecto.md)
- [Despliegue en Azure App Service](../azure/app-service-deployment.md)

[⬅ Volver al índice de .NET](README.md)
