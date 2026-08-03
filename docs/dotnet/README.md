# ⚙️ .NET

Esta sección reúne los comandos y flujos de trabajo prácticos de **.NET CLI** usados para crear, configurar y publicar microservicios bajo Clean Architecture.

## Contenido

| Documento | Descripción |
|---|---|
| [Crear la solución general](1-crear-solucion-general.md) | Configuración inicial y única: crea la solución que contiene todos los microservicios |
| [Crear un microservicio](2-crear-microservicio.md) | Comandos para crear un microservicio por capa y referenciarlos entre sí |
| [Gestión de paquetes NuGet](3-paquetes-nuget.md) | Instalar, eliminar y listar paquetes; paquetes esenciales recomendados por capa |
| [Crear conexión a la base de datos](4-crear-conexion-bd.md) | Configuración mínima del `DbContext`, `DependencyInjection` y la cadena de conexión en `Infrastructure` |
| [Entity Framework Core: migraciones](entity-framework-migraciones.md) | Crear, aplicar y eliminar migraciones, con y sin Clean Architecture |
| [Preparar proyecto](preparar-proyecto.md) | Entidad, configuración, DbContext, migración, repositorio, Commands/Queries y controlador, de punta a punta |
| [Crear una entidad (ejemplo Persona)](crear-entidad.md) | Entidad, configuración, DbContext, migración, repositorio |
| [Publicar un microservicio](publicar-proyecto.md) | Compilación y generación del paquete de publicación (`dotnet publish`) |

[⬅ Volver al índice principal](../../README.md)
