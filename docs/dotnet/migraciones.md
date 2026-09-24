# Migraciones

Las **migraciones de Entity Framework Core** son una forma de versionar los cambios del modelo de datos (entidades del `DbContext`) y aplicarlos de manera controlada e incremental a la base de datos, manteniendo sincronizado el esquema con el modelo de dominio definido en código, sin necesidad de escribir manualmente sentencias `CREATE TABLE` o `ALTER TABLE`.

> [!NOTE]
> En Clean Architecture, el `DbContext` debe ubicarse en la capa `Infrastructure`, mientras que el **proyecto de arranque** (`--startup-project`) siempre apunta a `API`, ya que es allí donde se configura la cadena de conexión y se resuelve la inyección de dependencias.

## Prerrequisitos

- Tener instalada la herramienta de EF Core:

  ```bash
  dotnet tool install --global dotnet-ef
  ```

- Tener el paquete `Microsoft.EntityFrameworkCore.Design` en el proyecto de arranque (`API`):

  ```bash
  dotnet add ../API package Microsoft.EntityFrameworkCore.Design
  ```

## Parámetros clave

| Parámetro | Significado |
|---|---|
| `--project` | Proyecto que contiene el `DbContext` (normalmente `Infrastructure`). Si se ejecuta desde su carpeta, puede omitirse |
| `--startup-project` | Proyecto que arranca la aplicación, donde está `Program.cs` (normalmente `API`) |
| `--context` | `DbContext` a usar cuando el proyecto tiene más de uno |
| `InitialCreate` | Nombre descriptivo de la migración. Se usa como nombre de clase C#, por lo que debe ser un identificador válido (sin puntos ni espacios) |
| `--output-dir` | Carpeta donde se generan los archivos de la migración (comando `migrations add`) |
| `--idempotent` | Comando `migrations script`. Genera un script que puede ejecutarse varias veces sin errores, ya que consulta la tabla `__EFMigrationsHistory` para verificar qué migraciones ya fueron aplicadas antes de ejecutar cada bloque |
| `--force` | Comando `migrations remove`. Revierte la migración en la base de datos si ya fue aplicada y luego la elimina del código |

## Comandos

Todos los comandos se ejecutan desde la carpeta del proyecto `Infrastructure`:

```bash
cd ./MicroservicioPersona/Infrastructure

# Ver los DbContext disponibles (opcional, antes de migrar)
dotnet ef dbcontext list --startup-project ../API

# Generar la migración
dotnet ef migrations add InitialCreate --startup-project ../API

# Listar las migraciones y su estado (aplicada / pendiente)
dotnet ef migrations list --startup-project ../API

# Generar un script SQL de migración
dotnet ef migrations script --idempotent -o MigrationScript/migration.sql --startup-project ../API

# Aplicar las migraciones pendientes directamente a la base de datos
dotnet ef database update --startup-project ../API

# Eliminar la última migración (solo si no fue aplicada)
dotnet ef migrations remove --startup-project ../API
```

### Revertir migraciones

Para volver a una migración anterior en la base de datos:

```bash
# Volver a una migración específica
dotnet ef database update InitialCreate --startup-project ../API

# Revertir todas las migraciones (deja la base de datos sin migraciones aplicadas)
dotnet ef database update 0 --startup-project ../API
```

Para generar un script SQL de reversión, el comando recibe `<DESDE> <HASTA>`: el primer argumento es la migración **más reciente** y el segundo, la migración a la que se quiere volver:

```bash
dotnet ef migrations script AddEmailToUsuario InitialCreate --idempotent -o MigrationScript/rollback.sql --startup-project ../API
```

> [!WARNING]
> `dotnet ef migrations remove` solo elimina la **última** migración generada. Si esa migración ya fue aplicada a la base de datos, el comando falla. En ese caso puedes:
>
> - Revertirla primero con `dotnet ef database update <MigraciónAnterior>` y luego ejecutar `migrations remove`, o
> - Usar `dotnet ef migrations remove --force`, que revierte y elimina en un solo paso.

### Alternativa: especificando un directorio de salida para las migraciones

Ejecutado desde la raíz de la solución:

```bash
dotnet ef migrations add InitialCreate \
  --project SIGESUP.Infrastructure \
  --startup-project SIGESUP.API \
  --output-dir Persistence/Migrations
```

## Buenas prácticas

- Nombra las migraciones de forma descriptiva (`AddEmailToUsuario`, no `Migration2`) y sin puntos ni caracteres especiales.
- Genera siempre un script SQL (`--idempotent`) antes de aplicar cambios en producción, para poder revisarlo previamente. Como alternativa, `dotnet ef migrations bundle` genera un ejecutable autónomo para aplicar las migraciones.
- Versiona las migraciones junto con el código fuente en el control de versiones (Git), incluyendo el archivo `*ModelSnapshot.cs`.
- Nunca elimines manualmente archivos de migración; usa siempre `dotnet ef migrations remove`, de lo contrario el `ModelSnapshot` queda desincronizado con el modelo.
- Si el proyecto tiene varios `DbContext`, indica siempre `--context` para evitar generar la migración en el contexto equivocado.

## Errores comunes

- Aplicar `database update` directamente en producción sin revisar antes el script SQL generado.
- Eliminar una migración ya aplicada a la base de datos sin revertirla primero.
- Olvidar `--startup-project` o no tener el paquete `Microsoft.EntityFrameworkCore.Design` en el proyecto de arranque.
- Mezclar ramas con migraciones distintas sobre el mismo modelo, lo que genera conflictos en el `ModelSnapshot`.
- Invertir el orden de los argumentos al generar el script de reversión (`<DESDE> <HASTA>`), obteniendo así un script que avanza en lugar de revertir.

[⬅](README.md) Volver al índice de .NET
