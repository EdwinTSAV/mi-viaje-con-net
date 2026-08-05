# Estructura de carpetas por capa

## Tabla de contenido
- [¿Qué es?](#qué-es)
- [¿Para qué sirve?](#para-qué-sirve)
- [Estructura general](#estructura-general)
- [Domain](#domain)
- [Application](#application)
- [Infrastructure](#infrastructure)
- [API](#api)
- [Buenas prácticas](#buenas-prácticas)
- [Recursos relacionados](#recursos-relacionados)

## ¿Qué es?

Es la organización interna de carpetas y archivos dentro de cada uno de los cuatro proyectos que conforman un microservicio bajo Clean Architecture: `Domain`, `Application`, `Infrastructure` y `API`.

## ¿Para qué sirve?

Una estructura de carpetas consistente entre microservicios facilita:
- La navegación del código para cualquier desarrollador del equipo.
- La incorporación de nuevas funcionalidades siguiendo siempre el mismo patrón.
- La escalabilidad del proyecto a medida que crece el número de casos de uso.

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
│   └── CreateUsuario/
│       ├── CreateUsuarioCommand.cs
│       ├── CreateUsuarioCommandHandler.cs
│       └── CreateUsuarioCommandValidator.cs
├── Queries/
│   └── GetUsuarioById/
│       ├── GetUsuarioByIdQuery.cs
│       └── GetUsuarioByIdQueryHandler.cs
├── DTOs/
├── Repositories/           # Contratos de persistencia (IUserRepository, etc.)
└── Services/               # Contratos de servicios externos (IEmailService, etc.)
```

- **Commands**: operaciones que modifican el estado (crear, actualizar, eliminar).
- **Queries**: operaciones de solo lectura.
- **DTOs**: objetos de transferencia de datos entre capas.
- **Repositories**: contratos (interfaces) encargados de abstraer el acceso a la persistencia de datos, por ejemplo IUsuarioRepository.
- **Services**: contratos (interfaces) que representan dependencias externas al dominio, como servicios de correo, almacenamiento de archivos, caché, autenticación o mensajería.

> **Nota**
> Tanto los Repositories como los Services son abstracciones (puertos) definidas en Application. Sus implementaciones concretas se encuentran en Infrastructure
> Cada carpeta de caso de uso (por ejemplo, `CreateUsuario`) agrupa el comando, su handler y su validador en un mismo lugar. Este enfoque se conoce como **Vertical Slice**, ya que organiza el código por funcionalidad en lugar de hacerlo por tipo de archivo.

## Infrastructure

```text
Infrastructure/
├── Persistence/
│   ├── AppDbContext.cs
│   ├── Configurations/     # IEntityTypeConfiguration
│   ├── Repositories/       # Implementaciones de contratos de persistencia
│   └── Migrations/
├── Services/               # Implementaciones de puertos
└── DependencyInjection.cs  # extension method para registrar todo
```

- **Persistence**: todo lo relacionado con el acceso a datos (contexto de EF Core, configuraciones de entidades, repositorios).
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

- Mantén un mismo criterio de nombres entre microservicios (por ejemplo, siempre `Persistence/Repositories`, nunca mezclar con `Data/Repos` en otro servicio).
- Agrupa los archivos por **caso de uso** en `Application` en lugar de por tipo técnico, para facilitar la localización del código relacionado.
- Evita ubicar lógica de acceso a datos fuera de `Infrastructure/Persistence`.

## Recursos relacionados

- [Clean Architecture](clean-architecture.md)
- [Crear un microservicio](../dotnet/2-crear-microservicio.md)

⬅ [Clean Architecture](clean-architecture.md) | [Finalizar](../../README.md)
