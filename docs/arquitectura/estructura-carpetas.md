# Estructura de carpetas por capa

Organización interna de carpetas y archivos dentro de cada uno de los cuatro proyectos que conforman un microservicio bajo Clean Architecture.

## Estructura general

```text
MicroservicioUsuarios/
├── Domain/
├── Application/
├── Infrastructure/
└── API/
```

## Domain

```text
Domain/
├── Entities/
├── ValueObjects/
└── Exceptions/
```

- **Entities**: entidades del negocio (por ejemplo, `Usuario`, `Pedido`).
- **ValueObjects**: objetos de valor inmutables (por ejemplo, `Email`, `Direccion`).
- **Exceptions**: excepciones propias del dominio (por ejemplo, `UsuarioNoEncontradoException`).

## Application

Se organiza siguiendo el patrón **CQRS** (separación entre comandos y consultas), habitualmente junto con **MediatR**:

```text
Application/
├── Commands/
│   └── Usuarios/
│       └── Create/
│           ├── CreateUsuarioDTO.cs
│           ├── CreateUsuarioCommand.cs
│           ├── CreateUsuarioCommandHandler.cs
│           └── CreateUsuarioCommandValidator.cs
├── Queries/
│   └── Usuarios/
│       └── GetById/
│           ├── GetUsuarioByIdDTO .cs
│           ├── GetUsuarioByIdQuery.cs
│           └── GetUsuarioByIdQueryHandler.cs
├── Repositories/           # Contratos de persistencia (IUserRepository, etc.)
├── Services/               # Contratos de servicios externos (IEmailService, etc.)
└── DependencyInjection.cs
```

- **Commands**: operaciones que modifican el estado (crear, actualizar, eliminar).
- **Queries**: operaciones de solo lectura.
- **DTOs**: objetos de transferencia de datos entre capas.
- **Repositories**: contratos encargados de abstraer el acceso a la persistencia de datos, por ejemplo IUsuarioRepository.
- **Services**: contratos que representan dependencias externas al dominio, como servicios de correo, almacenamiento de archivos, caché, autenticación o mensajería.
- **DependencyInjection.cs**: método de extensión `AddApplication()` que registra todos los servicios de esta capa.

> **Nota**>
> Cada carpeta de caso de uso (por ejemplo, `CreateUsuario`) agrupa el comando, su handler y su validador en un mismo lugar, de ser el caso también almacena su DTO. Este enfoque se conoce como **Vertical Slice**, ya que organiza el código por funcionalidad en lugar de hacerlo por tipo de archivo.

## Infrastructure

```text
Infrastructure/
├── Persistence/
│   ├── AppDbContext.cs
│   ├── Configurations/     # IEntityTypeConfiguration
│   └── Migrations/
├── Repositories/           # Implementaciones de contratos de persistencia
├── Services/               # Implementaciones de puertos
└── DependencyInjection.cs
```

- **Persistence**: todo lo relacionado con el acceso a datos (contexto de EF Core, configuraciones de entidades).
- **Repositories**: Implementaciones concretas de las interfaces definidas en `Application` (por ejemplo, un `IUsuarioRepository`).
- **Services**: implementaciones concretas de las interfaces definidas en `Application` (por ejemplo, un `EmailService` que use SendGrid).
- **DependencyInjection.cs**: método de extensión `AddInfrastructure()` que registra todos los servicios de esta capa.

## API

```text
API/
├── Controllers/
├── Middlewares/
└── Program.cs
```

- **Controllers**: puntos de entrada HTTP que reciben las peticiones y las traducen en `Commands`/`Queries` hacia `Application`.
- **Middlewares**: componentes transversales (manejo global de errores, logging de peticiones, autenticación, etc.).
- **Program.cs**: configuración de arranque de la aplicación, incluyendo el registro de `AddApplication()` y `AddInfrastructure()`.

## Buenas prácticas

- Mantén un mismo criterio de nombres entre microservicios.
- Agrupa los archivos por **caso de uso** en `Application` en lugar de por tipo técnico, para facilitar la localización del código relacionado.
- Evita ubicar lógica de acceso a datos fuera de `Infrastructure/Persistence`.

[⬅](clean-architecture.md) Clean Architecture | Finalizar [➡](../../README.md)
