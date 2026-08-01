# Crear una entidad

## Tabla de contenido
- [¿Para qué sirve?](#para-qué-sirve)
- [Paso 1 — Crear la entidad](#paso-1--crear-la-entidad)
- [Paso 2 — Crear el archivo de configuración](#paso-2--crear-el-archivo-de-configuración)
- [Paso 3 — Instanciar en el DbContext](#paso-3--instanciar-en-el-dbcontext)
- [Paso 4 — Generar la migración](#paso-4--generar-la-migración)
- [Paso 5 — Crear el contrato del repositorio](#paso-5--crear-el-contrato-del-repositorio)
- [Paso 6 — Crear el repositorio](#paso-6--crear-el-repositorio)
- [Paso 7 — Registrar el repositorio en Infrastructure](#paso-7--registrar-el-repositorio-en-infrastructure)
- [Modificar una entidad existente](#modificar-una-entidad-existente)
- [Buenas prácticas](#buenas-prácticas)
- [Recursos relacionados](#recursos-relacionados)

## ¿Para qué sirve?

Este documento describe, paso a paso, el flujo completo para dejar una **entidad** lista para persistencia y consulta dentro de un microservicio bajo Clean Architecture: desde la clase de dominio hasta su repositorio registrado en `Infrastructure`. Se usa como ejemplo una entidad **Persona**, con los campos: `Id`, `NroDni`, `PrimerNombre`, `ApellidoPaterno`, `ApellidoMaterno` y `FechaNacimiento`.

Este documento cubre hasta el punto en que la entidad queda **disponible para ser usada** (creada, consultada, actualizada o eliminada) desde la capa `Application`. La lógica de negocio propiamente dicha — Commands, Queries, Handlers y el controlador — se construye sobre esta base y se documenta en [Crear un CRUD](crear-crud.md).

> **Nota**
> Este flujo asume que ya tienes: la solución creada, los paquetes instalados y la conexión a la base de datos configurada. Si aún no lo tienes, revisa primero [Crear una solución de microservicios](crear-solucion-microservicios.md) y [Configuración mínima de conexión a la base de datos](crear-conexion-bd.md).

## Paso 1 — Crear la entidad

En `Domain/Entities/Persona.cs`:

```csharp
namespace MiProyecto.Domain.Entities;

public class Persona
{
    public Guid Id { get; set; }
    public required string NroDni { get; set; }
    public required string PrimerNombre { get; set; }
    public required string ApellidoPaterno { get; set; }
    public required string ApellidoMaterno { get; set; }
    public DateOnly FechaNacimiento { get; set; }
}
```

> **Tip**
> El modificador `required` (C# 11+) obliga a inicializar la propiedad al crear el objeto, evitando strings vacíos por descuido y con mejor soporte del compilador que `= string.Empty`. Más adelante, a nivel intermedio/avanzado, es común encapsular aún más la entidad con **setters privados** y un constructor o métodos (`Actualizar(...)`) que apliquen las reglas de negocio, evitando que cualquier capa modifique el objeto libremente.

## Paso 2 — Crear el archivo de configuración

En `Infrastructure/Persistence/Configurations/PersonaConfiguration.cs`, se definen las validaciones y restricciones que EF Core aplicará al generar la tabla:

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using MiProyecto.Domain.Entities;

namespace MiProyecto.Infrastructure.Persistence.Configurations;

public class PersonaConfiguration : IEntityTypeConfiguration<Persona>
{
    public void Configure(EntityTypeBuilder<Persona> builder)
    {
        builder.ToTable("personas");

        builder.HasKey(p => p.Id);

        builder.Property(p => p.NroDni)
            .IsRequired()
            .HasMaxLength(15);

        // Evita que dos personas se guarden con el mismo número de documento
        builder.HasIndex(p => p.NroDni)
            .IsUnique();

        builder.Property(p => p.PrimerNombre)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(p => p.ApellidoPaterno)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(p => p.ApellidoMaterno)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(p => p.FechaNacimiento)
            .IsRequired()
            .HasColumnType("date");
    }
}
```

> **Nota**
> `IsRequired()` + `HasMaxLength()` traducen las reglas de negocio en restricciones reales de la base de datos (`NOT NULL`, `VARCHAR(n)`). El índice único (`HasIndex().IsUnique()`) evita duplicados de DNI a nivel de base de datos, no solo en el código.

## Paso 3 — Instanciar en el DbContext

En `Infrastructure/Persistence/AppDbContext.cs`, se agrega el `DbSet<Persona>` y se aplica la configuración creada en el paso anterior:

```csharp
using Microsoft.EntityFrameworkCore;
using MiProyecto.Domain.Entities;
using MiProyecto.Infrastructure.Persistence.Configurations;

namespace MiProyecto.Infrastructure.Persistence;

public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public DbSet<Persona> Personas => Set<Persona>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.ApplyConfiguration(new PersonaConfiguration());
    }
}
```

> **Tip**
> Cuando el proyecto tenga varias entidades, en lugar de agregar una línea `ApplyConfiguration(...)` por cada una, se puede escanear todo el assembly automáticamente:
> ```csharp
> modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
> ```

## Paso 4 — Generar la migración

Con la entidad y su configuración ya registradas en el `DbContext`, se genera y aplica la migración siguiendo el proceso documentado en [Entity Framework Core: migraciones](entity-framework-migraciones.md):

```bash
dotnet ef migrations add AddPersona --project MiProyecto.Infrastructure --startup-project MiProyecto.API
dotnet ef database update --project MiProyecto.Infrastructure --startup-project MiProyecto.API
```

## Paso 5 — Crear el contrato del repositorio

En `Application/Interfaces/IPersonaRepository.cs`:

```csharp
using MiProyecto.Domain.Entities;

namespace MiProyecto.Application.Interfaces;

public interface IPersonaRepository
{
    Task<Persona?> ObtenerPorIdAsync(Guid id, CancellationToken cancellationToken);
    Task<List<Persona>> ListarAsync(CancellationToken cancellationToken);
    Task AgregarAsync(Persona persona, CancellationToken cancellationToken);
    void Actualizar(Persona persona);
    void Eliminar(Persona persona);
    Task<bool> GuardarCambiosAsync(CancellationToken cancellationToken);
}
```

> **Nota**
> `GuardarCambiosAsync` representa el `SaveChanges` del `DbContext`. Separarlo de `Agregar`/`Actualizar`/`Eliminar` permite, si más adelante lo necesitas, agrupar varias operaciones en una sola transacción antes de confirmar los cambios.

## Paso 6 — Crear el repositorio

En `Infrastructure/Repositories/PersonaRepository.cs`, se implementa el contrato anterior:

```csharp
using Microsoft.EntityFrameworkCore;
using MiProyecto.Application.Interfaces;
using MiProyecto.Domain.Entities;
using MiProyecto.Infrastructure.Persistence;

namespace MiProyecto.Infrastructure.Repositories;

public class PersonaRepository(AppDbContext context) : IPersonaRepository
{
    public async Task<Persona?> ObtenerPorIdAsync(Guid id, CancellationToken cancellationToken)
        => await context.Personas.FirstOrDefaultAsync(p => p.Id == id, cancellationToken);

    public async Task<List<Persona>> ListarAsync(CancellationToken cancellationToken)
        => await context.Personas.AsNoTracking().ToListAsync(cancellationToken);

    public async Task AgregarAsync(Persona persona, CancellationToken cancellationToken)
        => await context.Personas.AddAsync(persona, cancellationToken);

    public void Actualizar(Persona persona)
        => context.Personas.Update(persona);

    public void Eliminar(Persona persona)
        => context.Personas.Remove(persona);

    public async Task<bool> GuardarCambiosAsync(CancellationToken cancellationToken)
        => await context.SaveChangesAsync(cancellationToken) > 0;
}
```

> **Tip**
> `AsNoTracking()` en `ListarAsync` mejora el rendimiento en consultas de solo lectura, ya que EF Core no necesita rastrear cambios sobre esas entidades.

## Paso 7 — Registrar el repositorio en Infrastructure

En `Infrastructure/DependencyInjection.cs` (el mismo archivo creado en [Configuración mínima de conexión a la BD](crear-conexion-bd.md)), se agrega el registro del repositorio:

```csharp
services.AddScoped<IPersonaRepository, PersonaRepository>();
```

Quedando el método completo así:

```csharp
public static IServiceCollection AddInfrastructure(
    this IServiceCollection services,
    IConfiguration configuration)
{
    services.AddDbContext<AppDbContext>(options =>
    {
        options.UseNpgsql(configuration.GetConnectionString("DefaultConnection"));
    });

    services.AddScoped<IPersonaRepository, PersonaRepository>();

    return services;
}
```

## Modificar una entidad existente

Los 7 pasos anteriores solo se ejecutan completos al crear una entidad **completamente nueva**. Para modificar una entidad ya existente (por ejemplo, agregar un campo nuevo), el flujo se reduce a:

1. **Modificar la entidad** bajo el nuevo criterio ([Paso 1](#paso-1--crear-la-entidad)).
2. **Modificar el archivo de configuración** de la entidad ([Paso 2](#paso-2--crear-el-archivo-de-configuración)).
3. **Generar la migración** ([Paso 4 — Generar la migración](#paso-4--generar-la-migración)), usando un nombre descriptivo del cambio en lugar de `AddPersona` (por ejemplo, `AddTelefonoAPersona`).

> **Nota**
> No es necesario repetir los pasos 3, 5, 6 y 7: el `DbSet`, el contrato del repositorio, su implementación y el registro en `Infrastructure` ya existen desde la creación inicial de la entidad. Solo se tocan si el cambio agrega un método nuevo al repositorio (por ejemplo, `ObtenerPorNroDniAsync`), en cuyo caso sí se actualizan el contrato ([Paso 5](#paso-5--crear-el-contrato-del-repositorio)) y su implementación ([Paso 6](#paso-6--crear-el-repositorio)).

## Buenas prácticas

- Antes de generar la migración, revisa el modelo con `dotnet ef migrations add <Nombre> --dry-run` (o inspeccionando el archivo generado) para confirmar que el cambio esperado es el único que EF Core detectó.
- Usa nombres de migración descriptivos del cambio real (`AddTelefonoAPersona`), nunca genéricos (`Update2`, `Fix`).

## Recursos relacionados

- [Clean Architecture](../arquitectura/clean-architecture.md)
- [Estructura de carpetas por capa](../arquitectura/estructura-carpetas.md)
- [Configuración mínima de conexión a la BD](crear-conexion-bd.md)
- [Entity Framework Core: migraciones](entity-framework-migraciones.md)
- [Crear un CRUD](crear-crud.md)

[⬅ Volver al índice de .NET](README.md)
