# ⚙️ .NET

Esta sección reúne los comandos y flujos de trabajo prácticos de **.NET CLI** usados para crear, configurar y publicar microservicios bajo Clean Architecture.

## Contenido

| Documento | Descripción |
|---|---|
| [Estructura de los microservicios](estructura/README.md) | Configuración inicial y única: crea la solución que contiene todos los microservicios, Comandos para crear un microservicio por capa y referenciarlos entre sí, Instalar, eliminar y listar paquetes; paquetes esenciales recomendados por capa |
| [Conectar a la base de datos](database/README.md) | Configuración mínima del `DbContext`, `DependencyInjection` y la cadena de conexión en `Infrastructure`, crear, aplicar y eliminar migraciones con Clean Architecture |
| [Preparar proyecto](preparar-proyecto.md) | Entidad, configuración, DbContext, migración, repositorio, Commands/Queries y controlador, de punta a punta |
| [Crear una entidad (ejemplo Persona)](crear-entidad.md) | Entidad, configuración, DbContext, migración, repositorio |
| [Publicar un microservicio](publicar-proyecto.md) | Compilación y generación del paquete de publicación (`dotnet publish`) |

[⬅ Volver al índice principal](../../README.md)
